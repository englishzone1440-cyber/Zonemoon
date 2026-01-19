<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>سجل مدرسة جعفر بن أبي طالب</title>
    <style>
        :root { --primary: #000; --success-bg: #d4edda; --danger-bg: #f8d7da; --border: #ccc; }
        body { font-family: 'Segoe UI', Tahoma, sans-serif; background-color: #f4f7f6; margin: 0; padding: 10px; direction: rtl; color: #000; }
        .header-box { text-align: center; margin-bottom: 20px; }
        .record-title { font-size: 20px; font-weight: bold; color: #000; display: block; margin-bottom: 5px; }
        .school-name { font-size: 24px; font-weight: bold; color: #000; display: block; border-bottom: 2px solid #000; padding-bottom: 10px; }
        .container { background: white; max-width: 650px; margin: 0 auto; padding: 20px; border-radius: 15px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); }
        .input-group { margin-bottom: 15px; }
        .input-group label { display: block; margin-bottom: 5px; font-weight: bold; }
        .input-field { width: 100%; padding: 12px; border: 1px solid var(--border); border-radius: 8px; font-size: 16px; box-sizing: border-box; }
        .btn { padding: 12px; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; transition: 0.2s; }
        .full-btn { width: 100%; margin-top: 10px; }
        .class-nav { display: flex; gap: 5px; margin-bottom: 15px; justify-content: center; }
        .nav-item { flex: 1; padding: 10px; background: #eee; border: 1px solid #ccc; border-radius: 8px; text-align: center; cursor: pointer; font-weight: bold; }
        .nav-item.active { background: #2c3e50; color: white; }
        .hidden { display: none; }
        .selected { border: 3px solid #2980b9 !important; background-color: #e1f5fe !important; }
        table { width: 100%; border-collapse: collapse; margin-top: 15px; font-size: 13px; }
        th, td { padding: 8px; border: 1px solid #ddd; text-align: center; color: #000; }
        .row-pos { background-color: var(--success-bg); font-weight: 500; }
        .row-neg { background-color: var(--danger-bg); font-weight: 500; }
        .delete-btn { color: red; cursor: pointer; font-size: 18px; border: none; background: none; }
        .refresh-btn { background: #3498db; color: white; margin-bottom: 10px; display: flex; align-items: center; justify-content: center; gap: 8px; border: 1px solid #2980b9; }
    </style>
</head>
<body onload="initApp()">

<div class="header-box">
    <span class="record-title">سجل متابعة</span>
    <span class="school-name">مدرسة جعفر بن أبي طالب الإبتدائية</span>
</div>

<div class="container" id="loginPage">
    <div class="input-group">
        <label>اختر الصف:</label>
        <select id="selectClass" class="input-field" onchange="updateStudentOptions()">
            <option value="">-- اختر الصف --</option>
            <option value="A">رابع أ</option>
            <option value="B">رابع ب</option>
            <option value="C">رابع ج</option>
            <option value="D">رابع د</option>
            <option value="admin">دخول المعلم 💼</option>
        </select>
    </div>
    <div id="stLoginDiv" class="input-group">
        <label>اسم الطالب:</label>
        <select id="studentSelect" class="input-field"></select>
    </div>
    <div class="input-group">
        <label>كلمة المرور:</label>
        <input type="password" id="passInput" class="input-field">
    </div>
    <button class="btn full-btn" style="background:#000; color:white;" onclick="handleLogin()">دخول</button>
</div>

<div class="container hidden" id="mainApp">
    <div id="teacherNav" class="class-nav hidden">
        <div class="nav-item" id="navA" onclick="switchClass('A')">أ</div>
        <div class="nav-item" id="navB" onclick="switchClass('B')">ب</div>
        <div class="nav-item" id="navC" onclick="switchClass('C')">ج</div>
        <div class="nav-item" id="navD" onclick="switchClass('D')">د</div>
    </div>
    <h3 id="panelTitle" style="text-align:center; margin: 10px 0;"></h3>

    <div id="parentTools" class="hidden">
        <button class="btn full-btn refresh-btn" onclick="manualRefresh()">🔄 تحديث السجل الآن</button>
    </div>
    
    <div id="adminPanel" class="hidden">
        <div id="studentChecklist" style="max-height:150px; overflow-y:auto; border:1px solid #ccc; padding:10px; border-radius:8px; margin-bottom:10px;"></div>
        <div style="display:flex; gap:10px;">
            <button class="btn" style="background:#28a745; color:white; flex:1;" onclick="setMode('positive')">تميز ⭐</button>
            <button class="btn" style="background:#dc3545; color:white; flex:1;" onclick="setMode('negative')">تنبيه ⚠️</button>
        </div>
        <div id="behaviorGrid" class="hidden" style="display:grid; grid-template-columns:1fr 1fr; gap:5px; margin-top:10px;"></div>
        
        <div id="manualInputDiv" class="hidden" style="margin-top:10px;">
            <input type="text" id="manualText" class="input-field" placeholder="اكتب ملاحظة يدوية هنا...">
        </div>

        <button class="btn full-btn" style="background:#000; color:white;" onclick="saveEntry()">حفظ الرصد ومزامنة 💾</button>
    </div>

    <table>
        <thead><tr style="background:#eee;"><th>الطالب</th><th>السلوك</th><th>التاريخ والوقت</th><th id="delHeader">حذف</th></tr></thead>
        <tbody id="logsList"></tbody>
    </table>

    <button class="btn full-btn" style="background:#eee; margin-top:20px;" onclick="location.reload()">خروج 🚪</button>
</div>

<script>
const scriptURL = "https://script.google.com/macros/s/AKfycbx7i1hIeq1q5rqOTJhZi_e-aR3hE3mi8su7EgYuE4ugim7pSLILBPpFyzTPRQAaGEwM/exec"; 

let db = {
    "A": {"أحمد جاسم المهدي":"9802","أحمد علاء ال ثاني":"1624","الياس زكي العقاقة":"3459","الياس هاني برى":"0564","جعفر فاضل القصاب":"1140","حبيب محمد مكي":"5092","حسن عبد المجيد عيد":"9353","حسن علي البناي":"1519","حسن علي مغيص":"4648","حسين سعود المشهد":"2628","حسين محمد العلقم":"2087","حيدر محمد المعبر":"8612","رضا علي ال فضل":"8947","عباس علي الغمغام":"5846","عبد المحسن وليد غريب":"5102","علي ابراهيم المحيشي":"4825","علي سعيد الخاطر":"5812","علي صادق ال ناس":"1882","علي ممدوح التاروتي":"9291","علي وديع رمضان":"5439","فارس علي غمييض":"8230","قاسم جمال عمران":"9958","كرار سراج طالب":"4098","كنان محمد القصاب":"8276","محسن عبد الكريم المرزوق":"6518","محمد عقيل سليس":"2742","محمد علي درویش":"6239","مصطفى أحمد الميداني":"4472","منتظر عدنان العرب":"2393","مهدي صالح الزين":"9163","نبراس حسين شلفاح":"0557"},
    "B": {"أمان عبدالله امان":"3599","أمير عبد الله القلاف":"4730","ابراهيم محمد الموسى":"7935","حبيب عباس الحميدي":"6096","حسن رضا ال ثنيان":"1070","حسن موسى البشراوي":"2144","حسين ابراهيم الخضراوي":"8828","حسين عبد الله عاشور":"5051","حسين محمد الصفار":"5460","حسين نذير حسين":"3819","رضا زكي الصفار":"5395","ضياء جمال الخاطر":"3592","عباس أشرف ثاني":"5948","عباس عبد الرحمن الشيخ":"3301","عباس علي المناسف":"3621","عباس فاضل ال عمير":"8607","عباس مجتبى المحامل":"1173","عبد الله عابد اصباخ":"9883","عبدالله علي السنان":"7763","عبد الله محمود المحاسنه":"9990","علي نذير حمدان":"4799","فراس محمد عيد":"6977","كرار حسين المحروس":"2393","محمد احمد الضبيكي":"1075","محمد عدنان القصاب":"8273","محمد مصطفى ثويني":"6096","محمد حسن غزوي":"1077","منصور مهدي الشيوخ":"0225","مهدي صالح الشباط":"3525","مهدى عباس الجصاص":"1069","مهدي عبد الرحمن الدار":"4868","نبراس علي بري":"5399","نبراس فيصل ال قنبر":"0022","هاشم نور الشرفا":"3994","يوسف فاضل خزعل":"3844"},
    "C": {"أحمد خليل الحيراني":"5138","أحمد مصطفى ال درويش":"4435","بدر منير المدن":"4416","براء حسين امغيزل":"5599","جاسم محمد المشهد":"3322","جواد عارف البشراوي":"7478","جواد محمد أبوشومي":"8406","جواد زكي الزاير":"3131","حبيب علي المحاسنة":"2591","حسن بليغ البحراني":"4317","حسن حسين ال فضل":"8316","حسن علي ابوسعيد":"2704","حسن ياسر الحداد":"8435","حسين قاسم الصايغ":"4668","حميد ياسر حمدون":"0684","سيف علي الصفار":"6860","سيف فرات شعبان":"5542","عباس محمد الخانكي":"5983","عبد العزيز عبدالله خلف":"5296","عبد العزيز عبدالله عيد":"5159","علي حسين سليس":"6637","علي عبدالهادي الشميمي":"0000","علي زكي افويز":"9816","علي فيصل العجيان":"7767","علي مؤيد ال سماعيل":"1027","فارس علي الشيخ":"6574","فيصل محمد سلاط":"8513","قاسم زهير ثنيان":"1585","محمد جاسم الصدير":"5707","محمد حسن الوهاب":"8857","محمد زكي الجراش":"0965","محمد صالح ال دعبل":"7510","محمد علي النهاش":"9718","مهدي حاتم أبو الرحي":"5125","وليد عقيل الزاير":"8685","قاسم عبدرب النبي":"2216"},
    "D": {"آدم علي العسيف":"9808","أحمد عبد العزيز آل سعيد":"3742","أمجد محمد سويف":"4443","ابراهيم حسين الفشخي":"1235","جواد مجتبى آل الشيخ":"4071","حسن ابراهيم الغمغام":"2595","حسن طالب القصاب":"0813","حسن علي آل خليتيت":"8479","حسن يونس آل سلاط":"9554","حسين يونس آل سلاط":"9570","حسين سلمان الحايك":"1716","حسين علي البندري":"6116","سلمان عبد الله آل سلطان":"1711","طه عبد الله آل عيد":"3864","عباس علوي التكيه":"0971","عباس عيسى آل خليف":"8464","علي حسن الدار":"8970","علي حسين آل مهna":"6348","علي شهاب شهاب":"9013","علي محمد آل ياسين":"9039","علي محمد الخباز":"8560","قاسم احمد العسيف":"2152","قاسم حسن القفاص":"0537","قاسم مصطفى البشراوي":"9689","محسن جعفر بزرون":"0459","محمد زهير هجلس":"6615","محمد سعيد المشهد":"1694","محمد نايف النخلى":"2097","محمد نبيل آل عيد":"8512","محمد ياسر السنان":"8965","محمد رائد أبوتاكي":"4227","محمد عبدالله ابو السعود":"5054","نصر الله نذير الزاير":"9390","هادي علي آل بدر":"6113"}
};

let activeCls = "";
let curMode = "";
let isAdmin = false;
let currentLoggedStudent = "";

async function initApp() {
    await refreshData();
}

async function refreshData() {
    try {
        const r = await fetch(scriptURL);
        const cloud = await r.json();
        if (cloud && cloud.A) { db = cloud; }
        if (!isAdmin && currentLoggedStudent) renderLogs(currentLoggedStudent);
        else if (isAdmin) renderLogs();
    } catch(e) { console.log("Offline"); }
}

async function manualRefresh() {
    const btn = document.querySelector('.refresh-btn');
    btn.innerText = "⏳ جاري التحديث...";
    await refreshData();
    btn.innerText = "🔄 تحديث السجل الآن";
}

function updateStudentOptions() {
    const cls = document.getElementById('selectClass').value;
    const sel = document.getElementById('studentSelect');
    const div = document.getElementById('stLoginDiv');
    if(cls === 'admin' || cls === "") { div.classList.add('hidden'); }
    else {
        div.classList.remove('hidden');
        sel.innerHTML = '<option value="">-- اختر الاسم --</option>';
        if(db[cls]) {
            Object.keys(db[cls]).filter(n=>!n.includes("_logs")).sort().forEach(n => {
                sel.innerHTML += `<option value="${n}">${n}</option>`;
            });
        }
    }
}

function handleLogin() {
    const cls = document.getElementById('selectClass').value;
    const pw = document.getElementById('passInput').value;
    if(cls === 'admin' && pw === "1240Alih") {
        isAdmin = true; switchClass('A'); 
    } else if(db[cls]) {
        const name = document.getElementById('studentSelect').value;
        if(db[cls][name] === pw) {
            isAdmin = false; activeCls = cls; currentLoggedStudent = name; showMain(name);
        } else if (name !== "") { alert("بيانات الدخول غير صحيحة"); }
    }
}

function switchClass(cls) {
    activeCls = cls;
    document.querySelectorAll('.nav-item').forEach(el => el.classList.remove('active'));
    if(document.getElementById('nav' + cls)) document.getElementById('nav' + cls).classList.add('active');
    showMain();
}

function showMain(stName = "") {
    document.getElementById('loginPage').classList.add('hidden');
    document.getElementById('mainApp').classList.remove('hidden');
    document.getElementById('teacherNav').classList.toggle('hidden', !isAdmin);
    document.getElementById('delHeader').classList.toggle('hidden', !isAdmin);
    document.getElementById('parentTools').classList.toggle('hidden', isAdmin);
    if(isAdmin) {
        document.getElementById('adminPanel').classList.remove('hidden');
        document.getElementById('panelTitle').innerText = "لوحة رصد (رابع " + activeCls + ")";
        renderChecklist();
    } else {
        document.getElementById('adminPanel').classList.add('hidden');
        document.getElementById('panelTitle').innerText = "سجل الطالب: " + stName;
    }
    renderLogs(isAdmin ? "" : stName);
}

function renderChecklist() {
    const list = document.getElementById('studentChecklist');
    list.innerHTML = "";
    if(db[activeCls]) {
        Object.keys(db[activeCls]).filter(n => !n.includes("_logs")).sort().forEach(n => {
            list.innerHTML += `<div style="padding:6px; border-bottom:1px solid #eee;"><input type="checkbox" class="st-check" value="${n}"> ${n}</div>`;
        });
    }
}

function setMode(mode) {
    curMode = mode;
    document.getElementById('behaviorGrid').classList.remove('hidden');
    document.getElementById('manualInputDiv').classList.remove('hidden');
    const grid = document.getElementById('behaviorGrid');
    grid.innerHTML = "";
    const behaviors = ["تميز ⭐","هروب الطالب", "إزعاج المعلم", "تكرار التنبيه", "القيام بدون إذن", "عدم إحضار الكتاب", "عدم إخراج الكتاب", "عدم إحضار الأدوات", "التحدث", "اللعب", "الأكل", "التنمر", "الشجار", "النوم", "تخريب ممتلكات"];
    if(mode === 'negative') {
        behaviors.filter(b=>b!=="تميز ⭐").forEach(b => grid.innerHTML += `<div class="btn" style="font-size:11px; background:#fff; border:1px solid #ccc; padding:8px;" onclick="this.classList.toggle('selected')">${b}</div>`);
    } else {
        grid.innerHTML = `<div class="btn selected" style="background:#fff; border:1px solid #ccc;" onclick="this.classList.toggle('selected')">تميز ⭐</div>`;
    }
}

async function saveEntry() {
    const sts = Array.from(document.querySelectorAll('.st-check:checked')).map(c => c.value);
    const acts = Array.from(document.querySelectorAll('#behaviorGrid .selected')).map(b => b.innerText);
    const manual = document.getElementById('manualText').value.trim();
    if(manual) acts.push(manual);
    
    if(!sts.length || !acts.length) return alert("اختر الطالب والسلوك");
    const now = new Date();
    const dt = now.toLocaleDateString('ar-SA');
    const tm = now.toLocaleTimeString('ar-SA', {hour:'2-digit', minute:'2-digit'});
    
    sts.forEach(st => {
        if(!db[activeCls][st+"_logs"]) db[activeCls][st+"_logs"] = [];
        acts.forEach(a => {
            let exist = db[activeCls][st+"_logs"].find(l => l.text === a && l.date === dt);
            if(exist) exist.count++;
            else db[activeCls][st+"_logs"].push({ id: Date.now()+Math.random(), text: a, date: dt, time: tm, type: curMode, count: 1 });
        });
    });
    
    document.getElementById('manualText').value = "";
    renderLogs();
    sync();
}

function renderLogs(filter = "") {
    const body = document.getElementById('logsList');
    body.innerHTML = "";
    let logs = [];
    if(db[activeCls]) {
        for(let k in db[activeCls]) {
            if(k.includes("_logs")) {
                const name = k.replace("_logs","");
                if(filter && name !== filter) continue;
                db[activeCls][k].forEach(l => logs.push({stName: name, ...l}));
            }
        }
    }
    logs.sort((a,b) => b.id - a.id).forEach(l => {
        body.innerHTML += `<tr class="${l.type==='positive'?'row-pos':'row-neg'}">
            <td>${l.stName}</td><td>${l.text} <span style="font-weight:bold">${l.count}x</span></td><td>${l.date} - ${l.time}</td>
            <td class="${!isAdmin?'hidden':''}"><button class="delete-btn" onclick="deleteLog('${l.stName}', ${l.id})">🗑️</button></td>
        </tr>`;
    });
}

function deleteLog(st, id) {
    if(!isAdmin) return;
    if(!confirm("حذف؟")) return;
    db[activeCls][st+"_logs"] = db[activeCls][st+"_logs"].filter(l => l.id !== id);
    renderLogs();
    sync();
}

function sync() { 
    fetch(scriptURL, {method: 'POST', mode: 'no-cors', body: JSON.stringify(db)}); 
}

</script>
</body>
</html>
