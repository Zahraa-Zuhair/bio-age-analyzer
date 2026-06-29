<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>اختبار نمط الحياة والعمر الحيوي | Advanced AI</title>
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;700;900&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
        :root { --accent: #FFD700; --fire: #FF4500; --bg: #000; --card-bg: rgba(15, 15, 15, 0.9); }
        body { font-family: 'Tajawal', sans-serif; background: var(--bg); color: #fff; margin: 0; overflow-x: hidden; transition: 0.5s; }
        #canvas-bg { position: fixed; top: 0; left: 0; z-index: -1; width: 100%; height: 100%; }
        
        .scanner-line { position: fixed; top: -10px; left: 0; width: 100%; height: 4px; background: var(--accent); box-shadow: 0 0 25px var(--accent); z-index: 9999; display: none; }
        .main-container { max-width: 700px; margin: auto; padding: 20px; z-index: 1; position: relative; }
        .card { background: var(--card-bg); backdrop-filter: blur(20px); border: 1px solid rgba(255,215,0,0.1); border-radius: 24px; padding: 30px; margin-bottom: 25px; box-shadow: 0 10px 40px rgba(0,0,0,0.8); }
        
        .btn { width: 100%; padding: 20px; border: none; border-radius: 18px; font-size: 1.2rem; font-weight: 800; cursor: pointer; transition: 0.3s; }
        .btn-main { background: linear-gradient(45deg, var(--accent), var(--fire)); color: #000; }
        
        .ultra-score { font-size: 4rem; font-weight: 900; color: var(--accent); text-shadow: 0 0 30px var(--accent); }
        .bio-age { font-size: 1.8rem; color: #fff; margin: 15px 0; font-weight: bold; background: rgba(255,255,255,0.05); display: inline-block; padding: 10px 25px; border-radius: 15px; border-left: 5px solid var(--accent); }
        
        /* صندوق الأسرار المقفل */
        .secret-box { margin-top: 20px; padding: 20px; border: 2px dashed #444; border-radius: 15px; background: rgba(0,0,0,0.4); }
        .countdown { font-family: monospace; font-size: 1.5rem; color: #ff0000; letter-spacing: 2px; }
        
        .option-label { display: flex; align-items: center; padding: 15px; margin: 10px 0; background: rgba(255,255,255,0.03); border-radius: 12px; cursor: pointer; transition: 0.2s; border: 1px solid transparent; }
        .option-label.selected { border-color: var(--accent); background: rgba(255,215,0,0.05); }
        
        @keyframes pulse { 0% { opacity: 1; } 50% { opacity: 0.5; } 100% { opacity: 1; } }
        .pulse-text { animation: pulse 1s infinite; color: var(--fire); font-weight: 900; }
    </style>
</head>
<body>

<div id="scanner" class="scanner-line"></div>
<canvas id="canvas-bg"></canvas>

<div class="main-container">
    <div id="hero" class="card" style="text-align: center; margin-top: 80px;">
        <div style="font-size: 3rem; margin-bottom: 20px;">🔬</div>
        <h1>اختبار نمط الحياة والعمر الحيوي<br><span style="color:var(--accent)">BIO-AGE ANALYZER</span></h1>
        <p>نظام  يقارن عمرك الحقيقي بكفاءة أعضائك وأسلوب حياتك.</p>
        <button class="btn btn-main" onclick="initApp()">بدء بروتوكول القياس</button>
    </div>

    <div id="quiz-area" style="display: none;">
        <form id="ultraForm"></form>
        <button class="btn btn-main" onclick="startScan()">تحليل البيانات النهائية</button>
    </div>

    <div id="result-area" class="card" style="display: none; text-align: center;">
        <div id="badge" style="background:var(--fire); padding:5px 15px; border-radius:50px; display:inline-block; font-weight:bold; margin-bottom:10px;">تقرير الحالة</div>
        <div class="ultra-score" id="uScore">0%</div>
        
        <div class="bio-age" id="bioAgeDisplay">عمرك الحيوي: -- سنة</div>
        
        <h2 id="uTitle" style="color:var(--accent)"></h2>
        <p id="uDesc" style="color:#ccc; line-height:1.6;"></p>

        <div id="secretSection" class="secret-box">
            <div id="lockIcon">🔒 الأرشيف السري</div>
            <p id="lockMsg" style="font-size:0.9rem; color:#888;">الوصول محجوب بسبب ضعف المؤشرات الحيوية.</p>
            <div id="timer" class="countdown">23:59:59</div>
        </div>

        <button class="btn btn-main" onclick="shareResult()" style="margin-top:20px;">مشاركة النتيجة</button>
        <button onclick="location.reload()" style="background:none; border:none; color:#444; cursor:pointer; margin-top:15px;">إعادة الفحص</button>
    </div>
</div>

<script>
    // الجزيئات الخلفية
    const canvas = document.getElementById('canvas-bg');
    const ctx = canvas.getContext('2d');
    canvas.width = window.innerWidth; canvas.height = window.innerHeight;
    let particles = [];
    class P { constructor(){this.x=Math.random()*canvas.width; this.y=Math.random()*canvas.height; this.s=Math.random()*1; this.v=0.1;} draw(){ctx.fillStyle="rgba(255,215,0,0.2)"; ctx.beginPath(); ctx.arc(this.x,this.y,this.s,0,Math.PI*2); ctx.fill();} update(){this.y-=this.v; if(this.y<0)this.y=canvas.height;} }
    for(let i=0; i<100; i++) particles.push(new P());
    function anim(){ ctx.clearRect(0,0,canvas.width,canvas.height); particles.forEach(p=>{p.update();p.draw();}); requestAnimationFrame(anim); }
    anim();

    const questions = [
        { q: "جودة النوم الليلي", opts: ["عميق ومريح", "متقطع", "أرق دائم"], w: [-2, 2, 5] },
        { q: "استهلاك السكر يومياً", opts: ["شبه معدوم", "متوسط", "مرتفع جداً"], w: [-3, 1, 6] },
        { q: "النشاط البدني", opts: ["رياضة يومية", "مشي خفيف", "خمول تام"], w: [-4, 0, 7] },
        { q: "التوتر والضغط النفسي", opts: ["هادئ", "مضغوط", "احتراق نفسي"], w: [-2, 3, 8] },
        { q: "ساعات الشاشة", opts: ["أقل من 3", "5-8 ساعات", "إدمان كامل"], w: [-1, 2, 4] },
        { q: "شرب الماء", opts: ["مثالي", "مقبول", "جفاف"], w: [-2, 0, 3] },
        { q: "التغذية", opts: ["صحية", "مختلطة", "وجبات سريعة"], w: [-3, 2, 5] },
        { q: "التواصل الاجتماعي", opts: ["غني وداعم", "رسمي", "عزلة"], w: [-2, 1, 4] },
        { q: "الهوايات", opts: ["موجودة", "نادراً", "منسية"], w: [-2, 1, 3] },
        { q: "الكافيين", opts: ["معتدل", "كثير", "مفرط"], w: [0, 2, 4] },
        { q: "التعرض للشمس", opts: ["كافٍ", "قليل", "منعدم"], w: [-2, 1, 3] },
        { q: "القراءة والتعلم", opts: ["دائم", "أحياناً", "منقطع"], w: [-3, 0, 3] },
        { q: "التدخين / الأرجيلة", opts: ["لا أدخن", "أحياناً", "بكثرة"], w: [0, 5, 10] },
        { q: "تنظيم الوقت", opts: ["منظم", "عادي", "فوضى"], w: [-2, 1, 3] },
        { q: "الرضا العام", opts: ["راضٍ جداً", "متوسط", "يائس"], w: [-5, 0, 5] }
    ];

    function initApp() {
        document.getElementById('hero').style.display = 'none';
        document.getElementById('quiz-area').style.display = 'block';
        const f = document.getElementById('ultraForm');
        questions.forEach((q, i) => {
            const d = document.createElement('div'); d.className = 'card';
            d.innerHTML = `<h3>${i+1}. ${q.q}</h3>${q.opts.map((o, idx) => `<label class="option-label" onclick="playEff()"><input type="radio" name="q${i}" value="${q.w[idx]}" required> ${o}</label>`).join('')}`;
            f.appendChild(d);
        });
    }

    function playEff() {
        document.querySelectorAll('.option-label').forEach(l => {
            l.classList.remove('selected'); if(l.querySelector('input').checked) l.classList.add('selected');
        });
    }

    function startScan() {
        const sel = document.querySelectorAll('input[type="radio"]:checked');
        if(sel.length < 15) { alert("أكمل البيانات الحيوية!"); return; }
        
        window.scrollTo(0,0);
        const sc = document.getElementById('scanner');
        sc.style.display = 'block';
        let p = 0;
        const itv = setInterval(() => {
            p += 15; sc.style.top = p + 'px';
            if(p > window.innerHeight) { clearInterval(itv); sc.style.display = 'none'; finalize(sel); }
        }, 30);
    }

    function finalize(sel) {
        document.getElementById('quiz-area').style.display = 'none';
        document.getElementById('result-area').style.display = 'block';
        
        let impact = 0; sel.forEach(i => impact += parseInt(i.value));
        
        // حساب العمر الحيوي (نفرض العمر الحقيقي 25 كقاعدة)
        let baseAge = 25;
        let bioAge = baseAge + impact;
        if(bioAge < 18) bioAge = 18; // الحد الأدنى

        // حساب نسبة الجودة
        let score = 100 - (impact * 1.5);
        if(score > 100) score = 100; if(score < 0) score = 0;

        document.getElementById('uScore').innerText = Math.round(score) + "%";
        document.getElementById('bioAgeDisplay').innerText = "عمرك الحيوي: " + Math.round(bioAge) + " سنة";

        if(score >= 80) {
            document.getElementById('uTitle').innerText = "كفاءة شبابية مثالية";
            document.getElementById('uDesc').innerText = "أعضاؤك تعمل بكفاءة تصغر عمرك الحقيقي. أنت في المسار الذهبي.";
            document.getElementById('secretSection').innerHTML = "🔓 <span style='color:var(--accent)'>تم فتح الأرشيف:</span> سر الشباب الدائم هو الاستمرارية في قراراتك اليومية العظيمة!";
        } else if(score >= 50) {
            document.getElementById('uTitle').innerText = "أداء مستقر";
            document.getElementById('uDesc').innerText = "عمرك الحيوي قريب من عمرك الحقيقي. لديك بعض الثغرات التي تسرع الشيخوخة الرقمية.";
            startTimer();
        } else {
            document.getElementById('uTitle').innerText = "شيخوخة حيوية مبكرة";
            document.getElementById('uDesc').innerText = "تحذير: نمط حياتك يستهلك طاقتك بسرعة فائقة. عمرك الحيوي أكبر بكثير مما تظن.";
            document.getElementById('lockMsg').classList.add('pulse-text');
            startTimer();
        }
    }

    function startTimer() {
        let h=23, m=59, s=59;
        setInterval(() => {
            s--; if(s<0){s=59; m--;} if(m<0){m=59; h--;}
            document.getElementById('timer').innerText = `${h}:${m}:${s}`;
        }, 1000);
    }

    function shareResult() {
        const age = document.getElementById('bioAgeDisplay').innerText;
        const txt = `اكتشفت أن ${age}! افحص عمرك الحيوي هنا:`;
        window.open(`https://wa.me/?text=${encodeURIComponent(txt + " " + window.location.href)}`);
    }
</script>
</body>
</html>
