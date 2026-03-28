<!DOCTYPE html>
<html lang="zh">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>抽签左右排版</title>
<style>
:root{
    --bg:#ffffff;
    --text:#111;
    --panel:#f6f6f6;
    --border:#dddddd;
}
body.dark{
    --bg:#0f1115;
    --text:#f2f2f2;
    --panel:#1b1f26;
    --border:#333;
}
body{
    margin:0;
    font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto;
    background:var(--bg);
    color:var(--text);
    height:100vh;
    display:flex;
    justify-content:center;
    align-items:center;
    transition:.3s;
}
.blur{
    filter:blur(12px);
    pointer-events:none;
}
.container{
    display:flex;
    flex-direction:row;
    align-items:flex-start;
    gap:40px;
    max-width:800px;
    width:90%;
}
.left-panel, .right-panel{
    flex:1;
}
button{
    background:var(--text);
    color:var(--bg);
    border:none;
    padding:10px 18px;
    border-radius:10px;
    cursor:pointer;
}
#boards{
    display:flex;
    flex-wrap:wrap;
    gap:6px;
    justify-content:flex-start;
    align-items:flex-start;
    min-height:60px;
}

/* 机械翻牌样式 */
.flip{
    display:inline-block;
    margin:3px;
    width:28px;
    height:42px;
    perspective:200px;
}
.flip-inner{
    position:relative;
    width:100%;height:100%;
    transform-style:preserve-3d;
    transition:transform .18s linear;
}
.flip.flipping .flip-inner{ transform:rotateX(-180deg); }
.front,.back{
    position:absolute;
    width:100%;height:100%;
    backface-visibility:hidden;
    border:1px solid var(--border);
    border-radius:4px;
    display:flex;align-items:center;justify-content:center;
    background:var(--panel);
    font-size:20px;
}
.back{ transform:rotateX(180deg); }

/* Dock */
#dock{
    position:fixed;
    right:18px;
    bottom:18px;
    display:flex;
    gap:14px;
}
.icon{
    width:42px;height:42px;
    border-radius:50%;
    border:1px solid var(--border);
    background:var(--panel);
    display:flex;
    align-items:center;
    justify-content:center;
    cursor:pointer;
}
.icon svg{ width:20px;height:20px; stroke:var(--text); }

/* 面板通用 */
.panel{
    position:fixed;
    bottom:90px;
    right:18px;
    width:240px;
    background:var(--panel);
    border:1px solid var(--border);
    border-radius:18px;
    padding:18px;
    backdrop-filter:blur(20px);
}
.panel input{ width:90%; margin-top:8px; }
.panel ul{
    list-style:none;padding:0;margin:10px 0;
    max-height:120px;overflow:auto;text-align:left;
}
.panel li{
    display:flex;justify-content:space-between;
    padding:6px 0;border-bottom:1px solid var(--border);
}
</style>
</head>
<body>

<div class="container" id="main">
    <!-- 左侧按钮 -->
    <div class="left-panel">
        <button onclick="startDraw()">开始抽签</button>
        <div id="boards"></div>
    </div>
    <!-- 右侧显示 -->
    <div class="right-panel" id="rightPanel">
        <!-- 可显示多结果或者其他信息 -->
    </div>
</div>

<!-- Dock -->
<div id="dock">
    <!-- 数量 -->
    <div class="icon" onclick="toggleCountPanel()">
        <svg viewBox="0 0 24 24" fill="none">
            <rect x="4" y="5" width="6" height="6" stroke-width="2"/>
            <rect x="14" y="5" width="6" height="6" stroke-width="2"/>
            <rect x="4" y="15" width="6" height="6" stroke-width="2"/>
            <rect x="14" y="15" width="6" height="6" stroke-width="2"/>
        </svg>
    </div>

    <!-- 夜间 -->
    <div class="icon" onclick="toggleDark()">
        <svg viewBox="0 0 24 24" fill="none">
            <path d="M21 12.8A9 9 0 1111.2 3 7 7 0 0021 12.8z" stroke-width="2"/>
        </svg>
    </div>

    <!-- 设置 -->
    <div class="icon" onclick="toggleEditPanel()">
        <svg viewBox="0 0 24 24" fill="none">
            <circle cx="12" cy="12" r="3" stroke-width="2"/>
            <path d="M19.4 15a1 1 0 00.2 1.1l.1.1a2 2 0 01-2.8 2.8l-.1-.1a1 1 0 00-1.1-.2 1 1 0 00-.6.9V20a2 2 0 01-4 0v-.2a1 1 0 00-.6-.9 1 1 0 00-1.1.2l-.1.1a2 2 0 01-2.8-2.8l.1-.1a1 1 0 00.2-1.1 1 1 0 00-.9-.6H4a2 2 0 010-4h.2a1 1 0 00.9-.6 1 1 0 00-.2-1.1l-.1-.1a2 2 0 012.8-2.8l.1.1a1 1 0 001.1.2h.1a1 1 0 00.6-.9V4a2 2 0 014 0v.2a1 1 0 00.6.9h.1a1 1 0 001.1-.2l.1-.1a2 2 0 012.8 2.8l-.1.1a1 1 0 00-.2 1.1v.1a1 1 0 00.9.6H20a2 2 0 010 4h-.2a1 1 0 00-.4.1z" stroke-width="1.4"/>
        </svg>
    </div>
</div>

<!-- 数量面板 -->
<div id="countPanel" class="panel hidden">
    <div>结果数量 (1-10)</div>
    <input type="range" min="1" max="10" id="range">
    <div id="rangeVal"></div>
</div>

<!-- 编辑面板 -->
<div id="editPanel" class="panel hidden">
    <div>编辑抽取内容</div>
    <ul id="itemList"></ul>
    <input id="newItem" placeholder="输入内容">
    <button onclick="addItem()">添加</button>
</div>

<script>
let items=JSON.parse(localStorage.getItem("items"))||["张三","李四","王五"];
let multiCount=Number(localStorage.getItem("count"))||3;

const main=document.getElementById("main");
const countPanel=document.getElementById("countPanel");
const editPanel=document.getElementById("editPanel");
const range=document.getElementById("range");
const rangeVal=document.getElementById("rangeVal");

range.value=multiCount;
rangeVal.innerText=multiCount;

range.oninput=()=>{
    multiCount=Number(range.value);
    rangeVal.innerText=multiCount;
    localStorage.setItem("count",multiCount);
};

function blur(on){
    main.classList.toggle("blur",on);
}
function toggleCountPanel(){
    countPanel.classList.toggle("hidden");
    editPanel.classList.add("hidden");
    blur(!countPanel.classList.contains("hidden"));
}
function toggleEditPanel(){
    editPanel.classList.toggle("hidden");
    countPanel.classList.add("hidden");
    blur(!editPanel.classList.contains("hidden"));
    renderList();
}

function toggleDark(){
    document.body.classList.toggle("dark");
}

function renderList(){
    const ul=document.getElementById("itemList");
    ul.innerHTML="";
    items.forEach((it,i)=>{
        const li=document.createElement("li");
        li.innerHTML=`${it} <button onclick="removeItem(${i})">删</button>`;
        ul.appendChild(li);
    });
}
function addItem(){
    const v=document.getElementById("newItem").value.trim();
    if(!v)return;
    items.push(v);
    document.getElementById("newItem").value="";
    localStorage.setItem("items",JSON.stringify(items));
    renderList();
}
function removeItem(i){
    items.splice(i,1);
    localStorage.setItem("items",JSON.stringify(items));
    renderList();
}

function createBoard(text){
    const board=document.createElement("div");
    [...text].forEach(c=>{
        const f=document.createElement("div");
        f.className="flip";
        f.innerHTML=`<div class="flip-inner">
            <div class="front">${c}</div>
            <div class="back">${c}</div>
        </div>`;
        board.appendChild(f);
    });
    return board;
}

function flipTo(board,text){
    const flips=board.querySelectorAll(".flip");
    flips.forEach((f,i)=>{
        const char=text[i]||" ";
        const front=f.querySelector(".front");
        const back=f.querySelector(".back");
        back.innerText=char;
        setTimeout(()=>{
            f.classList.add("flipping");
            setTimeout(()=>{
                front.innerText=char;
                f.classList.remove("flipping");
            },160);
        },i*25);
    });
}

function startDraw(){
    const boards=document.getElementById("boards");
    boards.innerHTML="";
    let pool=[...items];
    for(let i=0;i<Math.min(multiCount,items.length);i++){
        const txt=pool.splice(Math.floor(Math.random()*pool.length),1)[0];
        const board=createBoard(txt);
        boards.appendChild(board);
        setTimeout(()=>flipTo(board,txt),300*i);
    }
}
</script>

</body>
</html>
