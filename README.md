<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>استخراج النص من الصورة</title>

<link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;700&display=swap" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/tesseract.js@5/dist/tesseract.min.js"></script>

<style>
:root{
  --primary:#2563EB;
  --bg:#f5f7fb;
  --card:#ffffff;
  --text:#111;
  --error:#e53935;
}
body.dark{
  --bg:#0f172a;
  --card:#020617;
  --text:#e5e7eb;
}
body{
  margin:0;
  font-family:'Cairo',sans-serif;
  background:var(--bg);
  color:var(--text);
}

/* شاشة الترحيب */
#welcome{
  position:fixed;
  inset:0;
  background:linear-gradient(135deg,#2563EB,#1e40af,#38bdf8);
  display:flex;
  align-items:center;
  justify-content:center;
  z-index:9999;
  animation:fadeIn .6s ease;
}
.welcome-box{
  background:rgba(255,255,255,.15);
  backdrop-filter:blur(20px);
  padding:25px;
  border-radius:25px;
  max-width:420px;
  color:#fff;
  text-align:center;
  animation:slideUp .6s ease;
}
.welcome-box h2{margin-bottom:10px}
.welcome-box p{line-height:1.8;font-size:14px}
.welcome-box a{color:#fff;text-decoration:underline}
.welcome-btn{
  margin-top:18px;
  background:#fff;
  color:#2563EB;
  border:none;
  padding:14px 22px;
  border-radius:30px;
  font-size:16px;
  cursor:pointer;
  animation:pulse 1.6s infinite;
}
.welcome-footer{
  margin-top:15px;
  font-size:13px;
}
@keyframes pulse{
  0%{transform:scale(1)}
  50%{transform:scale(1.05)}
  100%{transform:scale(1)}
}
@keyframes slideUp{
  from{transform:translateY(40px);opacity:0}
  to{opacity:1}
}
@keyframes fadeIn{
  from{opacity:0}
  to{opacity:1}
}

/* التطبيق */
.container{
  max-width:420px;
  margin:30px auto;
  background:var(--card);
  padding:20px;
  border-radius:20px;
}
h1{text-align:center;color:var(--primary)}

.lang-btn{
  position:absolute;
  top:15px;
  right:15px;
  background:var(--primary);
  color:#fff;
  border:none;
  padding:8px 14px;
  border-radius:20px;
  cursor:pointer;
}

.upload{
  border:2px dashed var(--primary);
  padding:20px;
  text-align:center;
  border-radius:15px;
  cursor:pointer;
}

img{
  width:100%;
  margin-top:12px;
  border-radius:15px;
  display:none;
}

textarea{
  width:100%;
  min-height:140px;
  border-radius:15px;
  padding:12px;
  margin-top:15px;
}

button.main{
  width:100%;
  padding:14px;
  border:none;
  border-radius:15px;
  background:var(--primary);
  color:#fff;
  font-size:16px;
  margin-top:12px;
  cursor:pointer;
}

#errorBox{
  display:none;
  background:var(--error);
  color:#fff;
  padding:14px;
  border-radius:15px;
  margin-top:12px;
  text-align:center;
}

#errorBox button{
  margin-top:8px;
  padding:10px 14px;
  border:none;
  border-radius:12px;
  background:#fff;
  color:#e53935;
  font-weight:bold;
  cursor:pointer;
}

footer{
  text-align:center;
  margin-top:15px;
  font-size:14px;
}
</style>
</head>

<body>

<!-- شاشة الترحيب -->
<div id="welcome">
  <div class="welcome-box">
    <h2>❓❌ ملاحظة التطبيق قيد التجربة ❌❓</h2>
    <p>
      هذا تطبيق ويب لاستخراج النص من الصور باستخدام تقنيات حديثة.<br>
      يتيح للمستخدم رفع صورة تحتوي على نص، ثم تحليلها تلقائيًا باستخدام OCR.<br><br>
      🔗 https://www.appcreator24.com/app3862641-fn1j5y
    </p>
    <button class="welcome-btn" onclick="closeWelcome()">الدخول إلى التطبيق</button>
    <div class="welcome-footer">
      تطوير: خميس مراد جروان<br>
      <a href="https://wa.me/967784070972" target="_blank">💬 واتساب</a>
    </div>
  </div>
</div>

<button class="lang-btn" onclick="toggleLang()">🌐 العربية</button>

<div class="container">
  <h1 id="title">استخراج النص من الصورة</h1>

  <div class="upload" onclick="file.click()">📷 اضغط لاختيار صورة</div>
  <input type="file" id="file" accept="image/*" hidden>

  <img id="preview">

  <textarea id="result" placeholder="سيظهر النص هنا..."></textarea>

  <div id="errorBox">
    ⚠️ حدث خطأ أثناء استخراج النص
    <br>
    <button onclick="resetError()">إعادة المحاولة</button>
  </div>

  <button class="main" onclick="copyText()">📋 نسخ النص</button>

  <footer>
    © تطوير: خميس مراد جروان<br>
    <a href="https://wa.me/967784070972" target="_blank">💬 واتساب</a>
  </footer>
</div>

<script>
let lang='ar';

function closeWelcome(){
  document.getElementById('welcome').style.display='none';
}

function vibrate(ms=50){
  if(navigator.vibrate) navigator.vibrate(ms);
}

file.onchange = e => {
  const img = preview;
  img.src = URL.createObjectURL(e.target.files[0]);
  img.style.display = 'block';
  result.value = '⏳ جارٍ المعالجة...';
  errorBox.style.display='none';

  Tesseract.recognize(
    img,
    lang==='ar' ? 'ara+eng' : 'eng'
  ).then(r=>{
    const text = r.data.text
      .replace(/[^\u0600-\u06FFa-zA-Z0-9\s.,]/g,'')
      .trim();

    if(!text){
      showError();
      return;
    }
    result.value = text;
    vibrate(80);
  }).catch(showError);
};

function showError(){
  errorBox.style.display='block';
  result.value='';
  vibrate([200,100,200]);
}

function resetError(){
  errorBox.style.display='none';
  file.value='';
}

function copyText(){
  if(!result.value) return;
  navigator.clipboard.writeText(result.value);
  vibrate(60);
}

function toggleLang(){
  lang = lang==='ar' ? 'en' : 'ar';
  document.documentElement.dir = lang==='ar'?'rtl':'ltr';
  title.textContent = lang==='ar'?'استخراج النص من الصورة':'Extract Text From Image';
  result.placeholder = lang==='ar'?'سيظهر النص هنا...':'Text will appear here...';
}
</script>

</body>
</html>
