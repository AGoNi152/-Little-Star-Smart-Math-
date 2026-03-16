<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>小星星数学冒险 - 竞技版</title>
    <style>
        /* --- 原界面样式变量与基础 --- */
        :root {
            --primary: #FF6B6B; --secondary: #4ECDC4; --accent: #FFE66D;
            --bg: #F0F7FF; --card-bg: #FFFFFF; --text: #2D3436;
            --correct: #2ECC71; --error: #FF4757;
        }

        body { font-family: 'PingFang SC', sans-serif; background: var(--bg); margin: 0; padding: 20px; overflow-x: hidden; }
        
        /* --- 主界面 Header --- */
        .header {
            background: white; padding: 20px; border-radius: 25px; box-shadow: 0 8px 0 #DDE8F0;
            display: flex; flex-wrap: wrap; gap: 15px; align-items: center; justify-content: space-between;
            max-width: 1200px; margin: 0 auto 30px;
        }
        .selectors select { padding: 10px; border-radius: 12px; border: 2px solid #F0F2F5; font-weight: bold; }
        .grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(320px, 1fr)); gap: 25px; max-width: 1200px; margin: 0 auto; }
        
        /* --- 题目卡片样式 --- */
        .card {
            background: var(--card-bg); border-radius: 25px; padding: 25px; box-shadow: 0 8px 0 #E2EAF0;
            display: flex; flex-direction: column; align-items: center; border: 4px solid transparent; transition: 0.2s;
        }
        .correct-card { border-color: var(--correct); background: #F2FFF7; }
        .wrong-card { border-color: var(--error); background: #FFF5F5; }
        .expression { font-size: 26px; font-weight: 800; margin-bottom: 15px; text-align: center; }

        /* 渲染辅助样式 */
        .v-box { display: inline-grid; grid-template-columns: 40px 1fr; font-size: 35px; text-align: right; width: 140px; font-family: 'Courier New'; }
        .v-line { grid-column: 1/span 2; border-bottom: 4px solid #333; margin: 5px 0 10px 0; }
        .v-div-container { display: flex; align-items: flex-end; font-size: 35px; font-family: 'Courier New'; margin-bottom: 15px; }
        .v-div-divisor { padding-right: 8px; border-right: 3px solid #333; }
        .v-div-dividend { padding-left: 10px; border-top: 3px solid #333; border-top-left-radius: 12px; }
        .input-group { display: flex; align-items: center; gap: 8px; margin-bottom: 10px; }
        input[type="text"] { width: 90px; padding: 10px; font-size: 22px; border: 3px solid #EEE; border-radius: 12px; text-align: center; outline: none; }
        .rem-input { width: 70px !important; border-color: var(--accent) !important; }
        .btn-check { background: var(--primary); color: white; border: none; padding: 10px 25px; border-radius: 15px; font-weight: bold; cursor: pointer; box-shadow: 0 4px 0 #E55B5B; }
        .frac { display: inline-flex; flex-direction: column; align-items: center; vertical-align: middle; margin: 0 5px; font-size: 20px; }
        .frac-line { border-top: 3px solid #333; width: 100%; margin: 2px 0; }
        .feedback { font-size: 14px; font-weight: bold; margin-top: 10px; min-height: 20px; }

        /* --- PK 模式专用样式 --- */
        #pk-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.85); z-index: 2000; display: none; align-items: center; justify-content: center; }
        #pk-intro { background: white; padding: 40px; border-radius: 30px; max-width: 500px; text-align: center; }
        #pk-stage { display: none; width: 100vw; height: 100vh; flex-direction: row; background: var(--bg); }
        .pk-player { flex: 1; display: flex; flex-direction: column; align-items: center; justify-content: space-between; padding: 20px; position: relative; }
        #p1-side { background: #FFF5F5; border-right: 2px dashed #CCC; }
        #p2-side { background: #F0FFFD; }

        .energy-wrap { width: 80%; height: 12px; background: #EEE; border-radius: 10px; margin: 10px 0; overflow: hidden; border: 2px solid #DDD; }
        .energy-fill { height: 100%; width: 0%; transition: width 0.3s; background: linear-gradient(90deg, #FF9F43, #FF6B6B); }
        .pk-input-view { font-size: 40px; color: var(--primary); min-height: 60px; border-bottom: 5px solid #DDD; min-width: 140px; text-align: center; margin-bottom: 10px; }
        .pk-keys { display: grid; grid-template-columns: repeat(3, 1fr); gap: 8px; width: 260px; }
        .k { background: white; padding: 15px; border-radius: 12px; font-size: 24px; font-weight: bold; text-align: center; cursor: pointer; box-shadow: 0 4px 0 #DDD; }
        .k-ok { background: var(--secondary); color: white; grid-column: span 2; box-shadow: 0 4px 0 #3DBDB4; }
        .ink-blur { filter: blur(25px); }
        .frozen { pointer-events: none; opacity: 0.4; }
        .status-msg { position: absolute; top: 45%; font-size: 30px; font-weight: bold; color: var(--error); z-index: 500; display: none; background: white; padding: 10px; border: 2px solid; }

        @media (max-width: 768px) {
            #pk-stage { flex-direction: column; }
            #p1-side { transform: rotate(180deg); border-right: none; }
        }
        #pk-result { position: fixed; inset: 0; background: white; z-index: 3000; display: none; flex-direction: column; align-items: center; justify-content: center; text-align: center; }
    </style>
</head>
<body>

<!-- 主界面内容 (保留原生脚本结构) -->
<div class="header no-print">
    <div class="selectors">
        <select id="semester-sel" onchange="loadTopics()">
            <option value="G1_UP">一年级上</option><option value="G1_DOWN">一年级下</option>
            <option value="G2_UP">二年级上</option><option value="G2_DOWN">二年级下</option>
            <option value="G3_UP">三年级上</option><option value="G3_DOWN">三年级下</option>
            <option value="G4_UP">四年级上</option><option value="G4_DOWN">四年级下</option>
            <option value="G5_UP">五年级上</option><option value="G5_DOWN">五年级下</option>
            <option value="G6_UP">六年级上</option><option value="G6_DOWN">六年级下</option>
        </select>
        <select id="topic-sel" onchange="initExercise()"></select>
    </div>
    <div id="progress-text" style="font-weight: bold; color: var(--primary);">进度: 0/12</div>
    <div style="display: flex; gap: 10px;">
        <button class="btn-check" style="background:var(--secondary); box-shadow:0 4px 0 #3DBDB4" onclick="initExercise()">换一批题</button>
        <button class="btn-check" style="background:#6C5CE7; box-shadow:0 4px 0 #5849BE" onclick="openPK()">双人PK</button>
    </div>
</div>

<div id="exercise-area" class="grid"></div>

<!-- PK 模式 Overlay -->
<div id="pk-overlay">
    <!-- 说明弹窗 -->
    <div id="pk-intro">
        <h2>⚔️ 双人同屏对战</h2>
        <p style="text-align: left; background: #f9f9f9; padding: 15px; border-radius: 10px;">
            1. 难度：锁定主界面当前选择考点。<br>
            2. 规则：准确率优先。分数需最简形式。<br>
            3. 技能：答对积攒能量，攒满干扰对方！
        </p>
        <button class="btn-check" style="padding: 15px 50px;" onclick="startBattle()">开始战斗</button>
        <button class="btn-check" style="background:#BBB; box-shadow:0 4px 0 #999; margin-top:10px;" onclick="closePK()">取消</button>
    </div>

    <!-- 战斗区域 -->
    <div id="pk-stage">
        <div id="p1-side" class="pk-player">
            <div id="p1-status" class="status-msg"></div>
            <div style="width:100%; display:flex; justify-content:space-between; font-weight:bold;">
                <span>玩家 1 (红)</span> <span id="p1-stats">正确: 0</span>
            </div>
            <div class="energy-wrap"><div id="p1-energy" class="energy-fill"></div></div>
            <div id="p1-q-box" style="height:150px; display:flex; align-items:center;"></div>
            <div id="p1-in" class="pk-input-view"></div>
            <div class="pk-keys" onclick="handlePKInput('p1', event)">
                <div class="k">1</div><div class="k">2</div><div class="k">3</div>
                <div class="k">4</div><div class="k">5</div><div class="k">6</div>
                <div class="k">7</div><div class="k">8</div><div class="k">9</div>
                <div class="k">/</div><div class="k">0</div><div class="k">.</div>
                <div class="k" style="background:#FAB1A0">C</div><div class="k k-ok">确认</div>
            </div>
        </div>

        <div id="pk-timer-circle" style="position:absolute; left:50%; top:20px; transform:translateX(-50%); background:white; width:70px; height:70px; border-radius:50%; display:flex; align-items:center; justify-content:center; font-weight:bold; font-size:24px; box-shadow:0 4px 10px rgba(0,0,0,0.2); z-index:1500;">60</div>
        <button onclick="quitBattle()" style="position:absolute; left:50%; top:100px; transform:translateX(-50%); z-index:1500; padding:5px 10px; border-radius:5px; border:none; background:#f39c12; color:white; cursor:pointer;">退出</button>

        <div id="p2-side" class="pk-player">
            <div id="p2-status" class="status-msg"></div>
            <div style="width:100%; display:flex; justify-content:space-between; font-weight:bold;">
                <span>玩家 2 (绿)</span> <span id="p2-stats">正确: 0</span>
            </div>
            <div class="energy-wrap"><div id="p2-energy" class="energy-fill"></div></div>
            <div id="p2-q-box" style="height:150px; display:flex; align-items:center;"></div>
            <div id="p2-in" class="pk-input-view"></div>
            <div class="pk-keys" onclick="handlePKInput('p2', event)">
                <div class="k">1</div><div class="k">2</div><div class="k">3</div>
                <div class="k">4</div><div class="k">5</div><div class="k">6</div>
                <div class="k">7</div><div class="k">8</div><div class="k">9</div>
                <div class="k">/</div><div class="k">0</div><div class="k">.</div>
                <div class="k" style="background:#FAB1A0">C</div><div class="k k-ok">确认</div>
            </div>
        </div>
    </div>
</div>

<!-- 结算 -->
<div id="pk-result">
    <h1 id="win-title" style="font-size:50px;"></h1>
    <div id="win-detail" style="font-size:24px; line-height:2;"></div>
    <button class="btn-check" style="padding:15px 50px; margin-top:20px;" onclick="location.reload()">回到主页</button>
</div>

<script>
// --- 通用数据库与引擎 (继承并补全) ---
const DB = {
    G1_UP: [{n:"20以内加减", t:"basic", r:[5,20]}],
    G1_DOWN: [{n:"100以内不进位", t:"basic", r:[10,99]}],
    G2_UP: [{n:"100以内竖式", t:"v_addsub", r:[20,99]}, {n:"乘法口诀", t:"mul_99"}],
    G2_DOWN: [{n:"表内除法", t:"div_99"}, {n:"有余数除法", t:"div_rem"}],
    G3_UP: [{n:"万以内竖式", t:"v_addsub", r:[1000,9999]}, {n:"多位数乘一位数", t:"v_mul", r1:[100,500], r2:[2,9]}],
    G3_DOWN: [{n:"两位数乘两位数", t:"v_mul", r1:[11,99], r2:[11,50]}, {n:"简单小数加减", t:"dec_add"}],
    G4_UP: [{n:"三位数乘两位数", t:"v_mul", r1:[100,400], r2:[10,30]}, {n:"大数除法竖式", t:"v_div", r1:[200,900], r2:[11,30]}],
    G4_DOWN: [{n:"小数进位加减", t:"dec_add"}, {n:"运算定律简算", t:"law"}],
    G5_UP: [{n:"小数乘法", t:"dec_mul"}, {n:"简易方程", t:"eqn"}],
    G5_DOWN: [{n:"异分母分数加减", t:"frac_add"}],
    G6_UP: [{n:"分数乘法", t:"frac_mul"}, {n:"分数除法", t:"frac_div"}],
    G6_DOWN: [{n:"比例计算", t:"ratio"}]
};

// 工具函数：约分校验
const gcd = (a, b) => b === 0 ? a : gcd(b, a % b);
const simplify = (n, d) => {
    const common = Math.abs(gcd(n, d));
    return `${n/common}/${d/common}`;
};

const Gen = {
    rand(min, max) { return Math.floor(Math.random()*(max-min+1))+min; },
    get(topic) {
        let n1, n2, sym, ans, style='h', html=null, isRem=false, ansRem=0;
        switch(topic.t) {
            case 'basic': n1 = this.rand(topic.r[0], topic.r[1]); n2 = this.rand(2, n1); sym = Math.random()>0.5?'+':'-'; ans = eval(`${n1}${sym}${n2}`); break;
            case 'v_addsub': n1 = this.rand(topic.r[0], topic.r[1]); n2 = this.rand(topic.r[0], n1); sym = Math.random()>0.5?'+':'-'; ans = eval(`${n1}${sym}${n2}`); style='v'; break;
            case 'mul_99': n1 = this.rand(2,9); n2 = this.rand(2,9); sym='×'; ans=n1*n2; break;
            case 'div_99': n2 = this.rand(2,9); ans = this.rand(2,9); n1=n2*ans; sym='÷'; break;
            case 'div_rem': n2 = this.rand(3,9); ans = this.rand(2,9); ansRem = this.rand(1, n2-1); n1 = n2*ans+ansRem; sym='÷'; isRem=true; ans=`${ans}.${ansRem}`; break;
            case 'v_mul': n1 = this.rand(topic.r1[0], topic.r1[1]); n2 = this.rand(topic.r2[0], topic.r2[1]); sym='×'; ans=n1*n2; style='v'; break;
            case 'v_div': n2 = this.rand(topic.r2[0], topic.r2[1]); ans = this.rand(10, 30); ansRem = this.rand(0, n2-1); n1 = n2*ans + ansRem; style='v_div'; isRem = true; ans=`${ans}.${ansRem}`; break;
            case 'dec_add': n1 = (this.rand(10, 99)/10).toFixed(1); n2 = (this.rand(10, 50)/10).toFixed(1); sym='+'; ans = (parseFloat(n1)+parseFloat(n2)).toFixed(1); break;
            case 'dec_mul': n1 = (this.rand(11, 40)/10).toFixed(1); n2 = (this.rand(11, 30)/10).toFixed(1); sym='×'; ans = (n1*n2).toFixed(2); break;
            case 'law': n1 = 25; n2 = this.rand(3,9); html = `${n1} × ${n2} × 4`; ans = n1*n2*4; break;
            case 'eqn': ans = this.rand(2,15); n2 = this.rand(1,10); html = `x + ${n2} = ${ans+n2}`; break;
            case 'frac_add': n1=this.rand(1,3); d1=this.rand(2,5); n2=this.rand(1,3); d2=this.rand(2,5); html=`<div class="frac"><span>${n1}</span><div class="frac-line"></div><span>${d1}</span></div> + <div class="frac"><span>${n2}</span><div class="frac-line"></div><span>${d2}</span></div>`; ans=simplify(n1*d2+n2*d1, d1*d2); break;
            case 'frac_mul': n1=this.rand(1,4); d1=this.rand(2,5); n2=this.rand(1,4); d2=this.rand(2,5); html=`<div class="frac"><span>${n1}</span><div class="frac-line"></div><span>${d1}</span></div> × <div class="frac"><span>${n2}</span><div class="frac-line"></div><span>${d2}</span></div>`; ans=simplify(n1*n2, d1*d2); break;
            case 'frac_div': n1=this.rand(1,4); d1=this.rand(2,5); n2=this.rand(1,4); d2=this.rand(2,5); html=`<div class="frac"><span>${n1}</span><div class="frac-line"></div><span>${d1}</span></div> ÷ <div class="frac"><span>${n2}</span><div class="frac-line"></div><span>${d2}</span></div>`; ans=simplify(n1*d2, d1*n2); break;
            case 'ratio': n1=this.rand(2,5); html = `x : ${n1*2} = 3 : 2`; ans = n1*3; break;
        }
        return { n1, n2, sym, ans, style, html, isRem, ansRem };
    },
    render(q) {
        if(q.html) return q.html;
        if(q.style === 'v') return `<div class="v-box"><div style="grid-column:2; font-size:12px; color:#CCC">. .</div><div></div><div>${q.n1}</div><div style="text-align:left">${q.sym}</div><div>${q.n2}</div><div class="v-line"></div></div>`;
        if(q.style === 'v_div') return `<div class="v-div-container"><div class="v-div-divisor">${q.n2}</div><div class="v-div-dividend">${q.n1}</div></div>`;
        return `${q.n1} ${q.sym} ${q.n2} = ?`;
    }
};

// --- 原练习逻辑 ---
let solved = 0;
function loadTopics() {
    const sem = document.getElementById('semester-sel').value;
    const topicSel = document.getElementById('topic-sel');
    topicSel.innerHTML = DB[sem].map((t, i) => `<option value='${i}'>${t.n}</option>`).join('');
    initExercise();
}
function initExercise() {
    solved = 0; updateProgress();
    const sem = document.getElementById('semester-sel').value;
    const idx = document.getElementById('topic-sel').value;
    const area = document.getElementById('exercise-area');
    area.innerHTML = '';
    for(let i=0; i<12; i++) { area.appendChild(createCard(Gen.get(DB[sem][idx]))); }
}
function createCard(q) {
    const card = document.createElement('div');
    card.className = 'card';
    let ui = Gen.render(q);
    let inputs = `<div class="input-group">`;
    if(q.isRem) {
        inputs += `<input type="text" class="q-in" placeholder="商"> <span>...</span> <input type="text" class="r-in rem-input" placeholder="余">`;
    } else {
        inputs += `<input type="text" class="main-in" placeholder="?">`;
    }
    inputs += `</div><button class="btn-check" onclick="verify(this, '${q.ans}')">检查</button><div class="feedback"></div>`;
    card.innerHTML = `<div class="expression">${ui}</div>` + inputs;
    return card;
}
function verify(btn, correct) {
    const card = btn.closest('.card');
    const feedback = card.querySelector('.feedback');
    let val;
    if(card.querySelector('.q-in')) {
        val = `${card.querySelector('.q-in').value.trim()}.${card.querySelector('.r-in').value.trim() || '0'}`;
    } else {
        val = card.querySelector('.main-in').value.trim();
    }
    if(val == correct) {
        card.className = 'card correct-card'; feedback.innerHTML = "🌟 正确！"; feedback.style.color = "var(--correct)";
        btn.disabled = true; solved++; updateProgress();
    } else {
        card.className = 'card wrong-card'; feedback.innerHTML = `❌ 错误`; feedback.style.color = "var(--error)";
    }
}
function updateProgress() { document.getElementById('progress-text').innerText = `进度: ${solved}/12`; }

// --- PK 竞技逻辑 ---
let pkState = { p1:{s:0, t:0, cur:"", target:"", eng:0}, p2:{s:0, t:0, cur:"", target:"", eng:0}, time:60 };
let pkInt;

function openPK() { document.getElementById('pk-overlay').style.display = 'flex'; }
function closePK() { document.getElementById('pk-overlay').style.display = 'none'; }
function quitBattle() { if(confirm("退出对战？")) location.reload(); }

function startBattle() {
    document.getElementById('pk-intro').style.display = 'none';
    document.getElementById('pk-stage').style.display = 'flex';
    pkState = { p1:{s:0, t:0, cur:"", target:"", eng:0}, p2:{s:0, t:0, cur:"", target:"", eng:0}, time:60 };
    nextPKQ('p1'); nextPKQ('p2');
    pkInt = setInterval(() => {
        pkState.time--;
        document.getElementById('pk-timer-circle').innerText = pkState.time;
        if(pkState.time <= 0) endBattle();
    }, 1000);
}

function nextPKQ(p) {
    const sem = document.getElementById('semester-sel').value;
    const idx = document.getElementById('topic-sel').value;
    const q = Gen.get(DB[sem][idx]);
    pkState[p].target = String(q.ans);
    pkState[p].cur = "";
    document.getElementById(`${p}-q-box`).innerHTML = Gen.render(q);
    document.getElementById(`${p}-in`).innerText = "";
}

function handlePKInput(p, e) {
    if(!e.target.classList.contains('k')) return;
    const val = e.target.innerText;
    const data = pkState[p];
    if(val === '确认') {
        data.t++;
        if(data.cur === data.target) {
            data.s++; data.eng += 34; if(data.eng >= 100) triggerHarrass(p);
            updateEng(p, data.eng); nextPKQ(p);
        } else {
            data.eng = 0; updateEng(p, 0); showPKMsg(p, "❌ 错误"); nextPKQ(p);
        }
    } else if(val === 'C') data.cur = "";
    else data.cur += val;
    document.getElementById(`${p}-in`).innerText = data.cur;
    document.getElementById(`${p}-stats`).innerText = `正确: ${data.s}`;
}

function updateEng(p, v) { document.getElementById(`${p}-energy`).style.width = Math.min(v, 100) + "%"; }

function triggerHarrass(attacker) {
    const defender = attacker === 'p1' ? 'p2' : 'p1';
    pkState[attacker].eng = 0; updateEng(attacker, 0);
    const side = document.getElementById(`${defender}-side`);
    const isInk = Math.random() > 0.5;
    if(isInk) {
        side.classList.add('ink-blur'); showPKMsg(defender, "☁️ 墨汁!");
        setTimeout(() => side.classList.remove('ink-blur'), 3000);
    } else {
        side.classList.add('frozen'); showPKMsg(defender, "❄️ 冰冻!");
        setTimeout(() => side.classList.remove('frozen'), 2000);
    }
}

function showPKMsg(p, txt) {
    const m = document.getElementById(`${p}-status`);
    m.innerText = txt; m.style.display = 'block';
    setTimeout(() => m.style.display = 'none', 1500);
}

function endBattle() {
    clearInterval(pkInt);
    const a1 = pkState.p1.t ? pkState.p1.s/pkState.p1.t : 0;
    const a2 = pkState.p2.t ? pkState.p2.s/pkState.p2.t : 0;
    let title = "";
    if(a1 === a2 && pkState.p1.s === pkState.p2.s) title = "握手言和，平局！🤝";
    else if(a1 > a2) title = "玩家 1 获胜！🏆";
    else if(a2 > a1) title = "玩家 2 获胜！🏆";
    else title = pkState.p1.s > pkState.p2.s ? "玩家 1 获胜！" : "玩家 2 获胜！";

    document.getElementById('pk-result').style.display = 'flex';
    document.getElementById('win-title').innerText = title;
    document.getElementById('win-detail').innerHTML = `
        玩家 1: 准确率 ${Math.floor(a1*100)}% | 答对 ${pkState.p1.s}<br>
        玩家 2: 准确率 ${Math.floor(a2*100)}% | 答对 ${pkState.p2.s}
    `;
}

window.onload = loadTopics;
</script>
</body>
</html>
