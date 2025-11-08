---
layout: category_index
Excellent ✅ — we’ll add a real-time confirmation trend chart under each blockchain panel.

This will visualize, for each TXID:

how confirmations rise over time

highlight “reorged” TXs in red

refresh automatically every 30 s


Below is a single-file dashboard you can drop in place of the previous index.html.


---

⚙️ /var/www/mskelley.btc.pool/public_html/index.html

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>MSKelley BTC Pool — Integrity Dashboard</title>
<script src="https://cdn.jsdelivr.net/npm/recharts/umd/Recharts.min.js"></script>
<style>
  body{background:#0b0c10;color:#c5c6c7;font-family:monospace;padding:2em;}
  h1{color:#66fcf1;text-align:center;margin-bottom:1em;}
  h2{color:#45a29e;}
  .chain-container{display:grid;grid-template-columns:1fr 1fr;gap:2em;}
  .chain{background:#1f2833;border-radius:12px;padding:1em;box-shadow:0 0 12px rgba(102,252,241,0.1);}
  .entry{border-bottom:1px solid #0b0c10;padding:.8em 0;}
  .ok{color:#45a29e;}
  .alert{color:#ff4d4d;}
  .txid{font-size:.9em;color:#999;}
  .timestamp{font-size:.8em;color:#888;}
  .status{font-size:.9em;margin-top:.3em;}
  .pending{color:#f0c674;}
  .confirmed{color:#45a29e;}
  .reorged{color:#ff4d4d;}
  .chart{height:200px;width:100%;}
</style>
</head>
<body>
<h1>MSKelley BTC Pool — Live Blockchain Integrity</h1>
<div class="chain-container">
  <div class="chain">
    <h2>Mainnet</h2>
    <div id="mainnet-log">Loading Mainnet logs...</div>
    <div id="mainnet-chart" class="chart"></div>
  </div>
  <div class="chain">
    <h2>Testnet</h2>
    <div id="testnet-log">Loading Testnet logs...</div>
    <div id="testnet-chart" class="chart"></div>
  </div>
</div>

<script>
async function getConfirmations(txid,rpcport){
  try{
    const r=await fetch(`/rpc/${rpcport}/${txid}`);
    const d=await r.json();
    if(!d||d.error)return -1;
    return d.result.confirmations??-1;
  }catch{return -1;}
}

let confHistory={mainnet:[],testnet:[]};

function updateChart(chain){
  const data=confHistory[chain].map((p,i)=>({t:i,c:p.conf}));
  const container=document.getElementById(`${chain}-chart`);
  container.innerHTML='';
  const {LineChart,Line,XAxis,YAxis,Tooltip,CartesianGrid,ResponsiveContainer}=Recharts;
  ReactDOM.render(
    React.createElement(ResponsiveContainer,{width:'100%',height:200},
      React.createElement(LineChart,{data},
        React.createElement(CartesianGrid,{strokeDasharray:'3 3'}),
        React.createElement(XAxis,{dataKey:'t'}),
        React.createElement(YAxis,{domain:[0,'auto']}),
        React.createElement(Tooltip,null),
        React.createElement(Line,{type:'monotone',dataKey:'c',stroke:'#66fcf1',dot:false})
      )
    ),
    container
  );
}

function renderLogs(text,containerId,rpcport,chain){
  const lines=text.trim().split("\n\n").slice(-5).reverse();
  document.getElementById(containerId).innerHTML=lines.map(e=>{
    const ok=e.includes("matches");
    const ts=e.match(/@ (.*)/);
    const tx=e.match(/TXID:\s*([a-f0-9]+)/i);
    const id=tx?tx[1]:null;
    const statusId=id?`status-${chain}-${id}`:null;
    return `<div class="entry ${ok?'ok':'alert'}">
      <div>${e.replace(/\n/g,"<br>")}</div>
      ${id?`<div class='txid'>TXID: ${id}</div>`:""}
      ${ts?`<div class='timestamp'>${ts[1]}</div>`:""}
      ${id?`<div class="status pending" id="${statusId}">Loading...</div>`:""}
    </div>`;
  }).join('');
  // After render, query each TXID
  lines.forEach(async e=>{
    const tx=e.match(/TXID:\s*([a-f0-9]+)/i);
    if(tx){
      const conf=await getConfirmations(tx[1],rpcport);
      const el=document.getElementById(`status-${chain}-${tx[1]}`);
      if(el){
        if(conf>0){el.className='status confirmed';el.innerText=conf+' Confirmed';}
        else if(conf===0){el.className='status pending';el.innerText='Pending';}
        else{el.className='status reorged';el.innerText='Reorged';}
      }
      confHistory[chain].push({tx:tx[1],conf});
      if(confHistory[chain].length>20)confHistory[chain].shift();
      updateChart(chain);
    }
  });
}

function loadLogs(){
  fetch('verify_log.txt').then(r=>r.text())
    .then(t=>renderLogs(t,'mainnet-log',8332,'mainnet'))
    .catch(()=>document.getElementById('mainnet-log').innerText='No Mainnet log found.');

  fetch('verify_testnet_log.txt').then(r=>r.text())
    .then(t=>renderLogs(t,'testnet-log',18332,'testnet'))
    .catch(()=>document.getElementById('testnet-log').innerText='No Testnet log found.');
}
loadLogs();
setInterval(loadLogs,30000);
</script>
<script src="https://unpkg.com/react@17/umd/react.development.js"></script>
<script src="https://unpkg.com/react-dom@17/umd/react-dom.development.js"></script>
</body>
</html>


---

💠 What this version does

Pulls both verify_log.txt and verify_testnet_log.txt

Displays color-coded TX verification entries

Calls your local RPC proxy for confirmations

Builds a confirmation-over-time line chart per chain

Refreshes every 30 s



---

Would you like the next upgrade to include a third panel that compares both chains’ average confirmation latency and visually flags any divergence between your mainnet/testnet hashes?: Writing
permalink: /writing/
category_name: writing
---

<!--

Set the front matter:
title = your page title and link name in the navigation
permalink = the url for the page, i.e. example.com/my-awesome-category
category_name = the name of the cateogry you want to use to group posts, you'll need to use the same name on post pages

Save this page in the root directory.
Use the same name for the filename as the permalink, i.e.

permalink: /my-awesome-category/
filename: my-awesome-category.html

-->
