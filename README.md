<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>تطبيق حساب درجات السنة</title>
<style>
:root{
  --bg:#0f172a;
  --text:#e2e8f0;
  --card-bg:rgba(255,255,255,0.05);
  --accent:linear-gradient(90deg,#38bdf8,#818cf8);
}
body{font-family:"Cairo",sans-serif;background:var(--bg);color:var(--text);margin:0;padding:20px;transition:background 0.3s,color 0.3s;}
h1{text-align:center;font-size:32px;margin-bottom:30px;background:var(--accent);-webkit-background-clip:text;color:transparent;font-weight:800;transition:all 0.3s;}
.subjects-container{display:grid;grid-template-columns:repeat(auto-fit,minmax(320px,1fr));gap:25px;}
.subject-card{border-radius:25px;padding:20px;background:var(--card-bg);backdrop-filter:blur(10px);box-shadow:0 15px 30px rgba(0,0,0,0.6);transition:0.4s;}
.subject-card:hover{transform:translateY(-8px) scale(1.02);box-shadow:0 25px 40px rgba(0,0,0,0.7);}
.subject-card h3{text-align:center;margin-bottom:12px;font-size:22px;transition:all 0.3s;}
label{display:block;margin:8px 0;font-weight:500;}
input{width:100%;padding:10px;border-radius:12px;border:none;background:rgba(255,255,255,0.08);color:var(--text);font-size:15px;transition:0.3s;}
input:focus{outline:none;background:rgba(255,255,255,0.15);}
button{padding:12px 16px;border:none;border-radius:14px;background:var(--accent);color:white;cursor:pointer;margin-top:12px;width:100%;font-weight:600;transition:0.3s;}
button:hover{opacity:0.95;transform:scale(1.03);}
.result{margin-top:12px;padding:10px;border-radius:12px;font-weight:bold;text-align:center;font-size:15px;transition:color 0.3s;}
.progress-container{margin-top:10px;height:14px;background:rgba(255,255,255,0.1);border-radius:999px;overflow:hidden;}
.progress-bar{height:100%;border-radius:999px;width:0%;transition:width 1s,background 0.8s;}
#overall{margin-top:35px;padding:25px;background:var(--card-bg);backdrop-filter:blur(10px);border-radius:20px;text-align:center;font-size:20px;font-weight:600;box-shadow:0 15px 25px rgba(0,0,0,0.5);transition:all 0.3s;}
.control-buttons{display:flex;gap:12px;flex-wrap:wrap;margin-bottom:30px;}
.control-buttons button{flex:1;}
.toggle-panel{position:fixed;top:15px;left:15px;background:var(--card-bg);padding:10px 12px;border-radius:16px;box-shadow:0 8px 20px rgba(0,0,0,0.5);z-index:1000;transition:all 0.3s;}
.toggle-panel select, .toggle-panel button{margin:4px 0;padding:6px;border-radius:8px;border:none;width:100%;cursor:pointer;}
body.dark{--bg:#1f2937;--text:#f1f5f9;--card-bg:rgba(255,255,255,0.1);--accent:linear-gradient(90deg,#818cf8,#f43f5e);}
</style>
</head>
<body>

<div class="toggle-panel">
  <button onclick="toggleDark()">🌙/☀️ الوضع الداكن</button>
  <select id="themeSelect" onchange="changeTheme(this.value)">
    <option value="default">🎨 السيمة الافتراضية</option>
    <option value="blue">🔵 أزرق</option>
    <option value="green">🟢 أخضر</option>
    <option value="pink">🌸 وردي</option>
    <option value="orange">🟠 برتقالي</option>
  </select>
</div>

<h1>🧾 حساب درجات السنة - طلاب الطب</h1>

<div class="control-buttons">
  <button onclick="computeAll()">📊 حساب كل المواد</button>
  <button onclick="resetAll()">🧹 إعادة التعيين</button>
  <button onclick="saveAll()">💾 حفظ</button>
  <button onclick="exportData()">📤 تصدير</button>
  <button onclick="importData()">📥 استيراد</button>
</div>

<div class="subjects-container" id="subjects"></div>
<div id="overall"></div>
<input type="file" id="fileInput" style="display:none" />

<script>
// تعريف المواد كما في النسخة السابقة
const subjects=[
  {name:"التشريح",color:"#3b82f6",parts:[{label:"عملي مد (10)",max:10},{label:"نظري مد (20)",max:20},{label:"عملي فاينل (20)",max:20},{label:"نظري فاينل (50)",max:50}]},
  {name:"الفسلجة",color:"#10b981",parts:[{label:"عملي مد (10)",max:10},{label:"نظري مد (25)",max:25},{label:"عملي فاينل أورال (6)",max:6},{label:"عملي فاينل سبوتات (4)",max:4},{label:"نظري فاينل (50)",max:50},{label:"سمنر (2)",max:2},{label:"كويزات (3)",max:3}]},
  {name:"الكيمياء",color:"#f97316",parts:[{label:"عملي مد (13)",max:13},{label:"نظري مد (20)",max:20},{label:"عملي فاينل (13)",max:13},{label:"نظري فاينل (50)",max:50},{label:"سمنر/بوستر (4)",max:4}]},
  {name:"الأنسجة",color:"#8b5cf6",parts:[{label:"عملي مد (10)",max:10},{label:"نظري مد (20)",max:20},{label:"عملي فاينل (13)",max:13},{label:"نظري فاينل (55)",max:55},{label:"لوك بوك (2)",max:2}]},
  {name:"الأجنة",color:"#ec4899",parts:[{label:"مد (30)",max:30},{label:"فاينل (70)",max:70}]},
  {name:"الأخلاقيات",color:"#facc15",parts:[{label:"مد (30)",max:30},{label:"فاينل (70)",max:70}]},
  {name:"جرائم حزب البعث",color:"#16a34a",parts:[{label:"مد (40)",max:40},{label:"فاينل (60)",max:60}]}
];
const container=document.getElementById("subjects");

// إنشاء البطاقات
subjects.forEach((subj,si)=>{
  const card=document.createElement("div");
  card.className="subject-card";
  card.style.borderTop=`6px solid ${subj.color}`;
  let html=`<h3>${subj.name}</h3>`;
  subj.parts.forEach((p,pi)=>{
    html+=`<label>${p.label}</label><input type="number" min="0" max="${p.max}" id="s${si}p${pi}">`;
  });
  html+=`<button onclick="compute(${si})">حساب</button>
         <div class="result" id="res${si}"></div>
         <div class="progress-container"><div class="progress-bar" id="bar${si}"></div></div>`;
  card.innerHTML=html;
  container.appendChild(card);
});

// التقديرات
function getGrade(percent){
  if(percent>=90) return {text:"🌟 ممتاز",color:"#16a34a"};
  else if(percent>=80) return {text:"🥇 جيد جدًا",color:"#22c55e"};
  else if(percent>=70) return {text:"🥈 جيد",color:"#facc15"};
  else if(percent>=60) return {text:"🙂 مقبول",color:"#f59e0b"};
  else return {text:"💀 راسب",color:"#ef4444"};
}

function compute(si){
  const subj=subjects[si]; let total=0,maxTotal=0;
  subj.parts.forEach((p,pi)=>{
    const val=parseFloat(document.getElementById(`s${si}p${pi}`).value)||0;
    total+=val; maxTotal+=p.max;
  });
  const percent=(total/maxTotal*100).toFixed(2);
  const grade=getGrade(percent);
  const resEl=document.getElementById(`res${si}`);
  resEl.innerHTML=`${total}/${maxTotal} (${percent}%) - <span style="color:${grade.color}">${grade.text}</span>`;
  const bar=document.getElementById(`bar${si}`);
  bar.style.width=`${percent}%`;
  bar.style.background=grade.color;
  saveAll();
}

function computeAll(){
  let total=0,full=0;
  subjects.forEach((subj,si)=>{
    subj.parts.forEach((p,pi)=>{
      const val=parseFloat(document.getElementById(`s${si}p${pi}`).value)||0;
      total+=val; full+=p.max;
    });
    compute(si);
  });
  const percent=(total/full*100).toFixed(2);
  const grade=getGrade(percent);
  document.getElementById("overall").innerHTML=`المعدل العام: ${percent}% - <span style="color:${grade.color}">${grade.text}</span>`;
}

function resetAll(){
  subjects.forEach((subj,si)=>{
    subj.parts.forEach((p,pi)=>{document.getElementById(`s${si}p${pi}`).value="";});
    document.getElementById(`res${si}`).innerHTML="";
    document.getElementById(`bar${si}`).style.width="0%";
  });
  document.getElementById("overall").innerHTML="";
  localStorage.clear();
}

function saveAll(){
  const data={};
  subjects.forEach((subj,si)=>{
    subj.parts.forEach((p,pi)=>{
      data[`s${si}p${pi}`]=document.getElementById(`s${si}p${pi}`).value;
    });
  });
  localStorage.setItem("grades",JSON.stringify(data));
}

window.onload=function(){
  const saved=JSON.parse(localStorage.getItem("grades")||"{}");
  for(let key in saved){
    if(document.getElementById(key)) document.getElementById(key).value=saved[key];
  }
  computeAll();
}

function exportData(){
  const data=localStorage.getItem("grades")||"{}";
  const blob=new Blob([data],{type:"application/json"});
  const url=URL.createObjectURL(blob);
  const a=document.createElement("a");
  a.href=url;
  a.download="grades.json";
  a.click();
  URL.revokeObjectURL(url);
}

function importData(){
  const input=document.getElementById("fileInput");
  input.click();
  input.onchange=()=>{
    const file=input.files[0];
    const reader=new FileReader();
    reader.onload=function(e){
      try{
        const data=JSON.parse(e.target.result);
        for(let key in data){
          if(document.getElementById(key)) document.getElementById(key).value=data[key];
        }
        computeAll();
        saveAll();
      }catch(err){alert("خطأ في استيراد الملف");}
    }
    reader.readAsText(file);
  }
}

// تبديل الوضع الداكن
function toggleDark(){
  document.body.classList.toggle('dark');
}

// تغيير السيمات
function changeTheme(val){
  const root=document.documentElement;
  if(val=="default"){
    root.style.setProperty('--accent','linear-gradient(90deg,#38bdf8,#818cf8)');
  }else if(val=="blue"){
    root.style.setProperty('--accent','linear-gradient(90deg,#3b82f6,#60a5fa)');
  }else if(val=="green"){
    root.style.setProperty('--accent','linear-gradient(90deg,#10b981,#34d399)');
  }else if(val=="pink"){
    root.style.setProperty('--accent','linear-gradient(90deg,#ec4899,#f472b6)');
  }else if(val=="orange"){
    root.style.setProperty('--accent','linear-gradient(90deg,#f97316,#fb923c)');
  }
}
</script>
</body>
</html>
