<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>WA Guardian | AI Anti Hack System</title>
<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600;700&display=swap" rel="stylesheet" />
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<!-- Leaflet CSS -->
<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
<!-- Leaflet JS -->
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>

<style>
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    font-family: 'Poppins', sans-serif;
  }
  body {
    background: #0b0f1a;
    color: white;
  }
  header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 20px 60px;
    background: #111827;
  }
  .logo {
    font-weight: 700;
    font-size: 22px;
    color: #00f2ff;
  }
  .container {
    padding: 40px 60px;
  }
  /* Card styles */
  .card {
    background: #141b2d;
    padding: 30px;
    border-radius: 15px;
    margin-bottom: 30px;
    box-shadow: 0 0 20px rgba(0, 242, 255, 0.1);
  }
  h2 {
    margin-bottom: 20px;
    color: #00f2ff;
  }
  textarea {
    width: 100%;
    height: 120px;
    padding: 15px;
    border-radius: 10px;
    border: none;
    background: #0f1626;
    color: white;
  }
  button {
    margin-top: 15px;
    padding: 12px 25px;
    background: #00f2ff;
    border: none;
    border-radius: 30px;
    font-weight: 600;
    cursor: pointer;
  }
  button:hover {
    box-shadow: 0 0 15px #00f2ff;
  }
  .result {
    margin-top: 20px;
    font-weight: 600;
    font-size: 18px;
  }
  .risk-high {
    color: #ff4d4d;
  }
  .risk-medium {
    color: #ffcc00;
  }
  .risk-low {
    color: #00ff88;
  }
  /* Log styles */
  .log {
    font-size: 13px;
    line-height: 1.6;
    color: #00ff99;
    max-height: 150px;
    overflow: auto;
    background: #0f1626;
    padding: 10px;
    border-radius: 10px;
  }
  /* Chart styles */
  canvas {
    background: #0f1626;
    border-radius: 10px;
    padding: 10px;
  }
  /* Layout styles */
  .grid {
    display: grid;
    grid-template-columns: 2fr 1fr;
    gap: 20px;
  }
  .map-section {
    height: 300px;
  }
  #map {
    height: 100%;
    border-radius: 10px;
  }
  /* Weekly attack chart container */
  .weekly-chart {
    margin-top: 30px;
  }
  /* Detection & Warning Recent */
  .updates {
    margin-top: 30px;
  }
</style>
</head>

<body>

<header>
  <div class="logo">WA GUARDIAN AI</div>
</header>

<div class="container">

  <!-- 1. SMS Scan -->
  <div class="card">
    <h2>Scan SMS / WhatsApp Text</h2>
    <textarea id="inputText" placeholder="Tempel pesan mencurigakan di sini..."></textarea>
    <button onclick="scanText()">Scan Sekarang</button>
    <div id="result" class="result"></div>
  </div>

  <!-- 2. Statistik Deteksi & Peta Risiko -->
  <div class="grid">
    <div class="card">
      <h2>Statistik Deteksi</h2>
      <canvas id="attackChart" height="150"></canvas>
    </div>
    <div class="card">
      <h2>Zona Risiko dan Lokasi Pengguna</h2>
      <div id="mapPlaceholder" style="height:150px; margin-bottom:10px;">Lokasi pengguna akan muncul di sini</div>
      <div id="map"></div>
    </div>
  </div>

  <!-- 3. Grafik Serangan Mingguan -->
  <div class="card weekly-chart">
    <h2>Serangan Mingguan</h2>
    <canvas id="weeklyAttackChart" height="150"></canvas>
  </div>

  <!-- 4. Log Aktivitas & Peringatan Terbaru -->
  <div class="updates">
    <div class="card">
      <h2>Log Aktivitas</h2>
      <div class="log" id="logBox"></div>
    </div>
    <div class="card">
      <h2>Deteksi & Peringatan Terbaru</h2>
      <div id="recentAlerts">
        <p><strong>Hari ini:</strong></p>
        <div id="alertList">Tidak ada peringatan terbaru.</div>
      </div>
    </div>
  </div>

</div>

<script>
// ====== Pengaturan =======
const phishingKeywords = [
  "hadiah", "klik link", "verifikasi akun", "akun diblokir",
  "transfer sekarang", "menang undian", "gratis saldo",
  "bank", "pin", "otp", "kode verifikasi", "whatsapp support",
  "login dari perangkat baru", "reset password", "data anda"
];

const suspiciousDomains = [
  ".xyz", ".top", ".click", ".gq", ".ru", ".tk"
];

let highCount = 0;
let mediumCount = 0;
let lowCount = 0;

// ================== Chart Deteksi ==================
const ctx = document.getElementById('attackChart').getContext('2d');
const attackChart = new Chart(ctx, {
  type: 'bar',
  data: {
    labels: ['High Risk', 'Medium Risk', 'Low Risk'],
    datasets: [{
      label: 'Detections',
      data: [0, 0, 0],
      backgroundColor: ['#ff4d4d', '#ffcc00', '#00ff88']
    }]
  },
  options: {
    plugins: { legend: { labels: { color: 'white' } } },
    scales: {
      x: { ticks: { color: 'white' } },
      y: { ticks: { color: 'white' }, beginAtZero: true }
    }
  }
});

// ================== Chart Serangan Mingguan ==================
const weeklyCtx = document.getElementById('weeklyAttackChart').getContext('2d');
const weeklyAttackChart = new Chart(weeklyCtx, {
  type: 'line',
  data: {
    labels: ['Sen', 'Sel', 'Rab', 'Kam', 'Jum', 'Sab', 'Min'],
    datasets: [{
      label: 'Jumlah Serangan',
      data: [0,0,0,0,0,0,0],
      fill: true,
      backgroundColor: 'rgba(0, 242, 255, 0.2)',
      borderColor: '#00f2ff',
      tension: 0.4,
      pointBackgroundColor: '#00f2ff'
    }]
  },
  options: {
    plugins: { legend: { labels: { color: 'white' } } },
    scales: {
      x: { ticks: { color: 'white' } },
      y: { ticks: { color: 'white' }, beginAtZero:true }
    }
  }
});

// ================== Data Mingguan ==================
let weeklyData = JSON.parse(localStorage.getItem('weeklyData')) || [0,0,0,0,0,0,0];

// Fungsi update data mingguan
function updateWeeklyData() {
  weeklyData.shift();
  const totalToday = highCount + mediumCount + lowCount;
  weeklyData.push(totalToday);
  localStorage.setItem('weeklyData', JSON.stringify(weeklyData));
  weeklyAttackChart.data.datasets[0].data = weeklyData;
  weeklyAttackChart.update();
}

// ================== Map & Lokasi ==================
function initMap() {
  window.myMap = L.map('map').setView([0,0], 2);
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '© OpenStreetMap contributors'
  }).addTo(window.myMap);
}

// Fungsi ambil lokasi pengguna
async function getUserLocation() {
  try {
    const response = await fetch('https://ipinfo.io/json?token=YOUR_TOKEN');
    const data = await response.json();
    if(window.myMap && data.loc){
      const [lat, lon] = data.loc.split(',');
      window.myMap.setView([lat, lon], 13);
      L.marker([lat, lon]).addTo(window.myMap)
        .bindPopup(`${data.city}, ${data.region}, ${data.country}`)
        .openPopup();
      // Tampilkan lokasi di bawah zona risiko
      document.getElementById('mapPlaceholder').innerHTML = `
        <strong>Lokasi Anda:</strong><br>
        ${data.city}, ${data.region}<br>
        Latitude: ${lat}<br>
        Longitude: ${lon}
      `;
    }
    window.userCoords = [lat, lon];
  } catch (err) {
    console.log('Gagal mendapatkan lokasi:', err);
  }
}

// ================== Fungsi Scan ==================
function scanText() {
  let text = document.getElementById("inputText").value.toLowerCase();
  let score = 0;
  phishingKeywords.forEach(k => { if(text.includes(k)){ score+=15; } });
  suspiciousDomains.forEach(d => { if(text.includes(d)){ score+=25; } });
  if(text.match(/http[s]?:\/\//)){ score+=20; }
  if(text.match(/\d{4,6}/) && text.includes("otp")){ score+=30; }

  let resultBox = document.getElementById("result");
  let logBox = document.getElementById("logBox");
  let alertDiv = document.getElementById('alertList');

  let status="";
  let className="";
  if(score>=60){
    status="HIGH RISK - Kemungkinan Phishing / Hack WA!";
    className="risk-high"; highCount++;
    notifyUser("⚠️ HIGH RISK terdeteksi!");
    addRecentAlert("HIGH", status);
  }else if(score>=30){
    status="MEDIUM RISK - Perlu waspada.";
    className="risk-medium"; mediumCount++;
    addRecentAlert("MEDIUM", status);
  }else{
    status="LOW RISK - Aman.";
    className="risk-low"; lowCount++;
  }

  resultBox.innerHTML = "Skor Risiko: "+score+" → "+status;
  resultBox.className="result "+className;

  updateChart();
  addLog(status);
  updateWeeklyData();

  // Jika lokasi sudah diambil, tampilkan di peta
  if(window.userCoords){
    window.myMap.setView([window.userCoords[0], window.userCoords[1]], 13);
  }
}

// ================== Tambahan ==================
function updateChart() {
  attackChart.data.datasets[0].data = [highCount, mediumCount, lowCount];
  attackChart.update();
}

function addLog(message){
  let time = new Date().toLocaleTimeString();
  let logBox = document.getElementById("logBox");
  logBox.innerHTML += "["+time+"] "+message+"<br>";
  logBox.scrollTop=logBox.scrollHeight;
  saveLog(message);
}

function saveLog(msg){
  let logs = JSON.parse(localStorage.getItem("logs")) || [];
  logs.push({time:new Date().toLocaleString(), msg:msg});
  localStorage.setItem("logs", JSON.stringify(logs));
}

function addRecentAlert(level, message){
  const container = document.getElementById('alertList');
  const p = document.createElement('p');
  p.innerHTML=`<strong>${new Date().toLocaleTimeString()}:</strong> ${message} (${level})`;
  container.prepend(p);
  while(container.children.length > 5){
    container.removeChild(container.lastChild);
  }
}

function notifyUser(msg){
  if(Notification.permission !== "granted"){
    Notification.requestPermission();
  }else{
    new Notification("WA Guardian Alert",{body: msg});
  }
}

// ================== Inisialisasi ==================
window.onload = function() {
  initMap();
  getUserLocation();
  // Load data mingguan dari storage
  let storedWeekly = JSON.parse(localStorage.getItem('weeklyData'));
  if(storedWeekly){
    weeklyData = storedWeekly;
    weeklyAttackChart.data.datasets[0].data = weeklyData;
    weeklyAttackChart.update();
  }
};
</script>

</body>
</html>
