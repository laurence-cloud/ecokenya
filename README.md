<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Eco Kenya - Fixed</title>

  <!-- Firebase -->
  <script src="https://www.gstatic.com/firebasejs/10.14.1/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.14.1/firebase-database-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.14.1/firebase-auth-compat.js"></script>

  <style>
    body { margin:0; font-family:Arial,sans-serif; background:#f0f0f0; color:#333; min-height:100vh; padding-bottom:90px; }
    .dark-mode { background:#121212; color:#e0e0e0; }
    .dark-mode .card { background:#1e1e1e; }
    .dark-mode header { background:#003300; }
    header { background:#006600; color:white; padding:10px; text-align:center; position:fixed; top:0; width:100%; box-shadow:0 2px 5px black; z-index:10; }
    .points-box { background:rgba(255,255,255,0.3); padding:5px 10px; border-radius:15px; margin-top:5px; display:inline-block; }
    main { padding:80px 10px 100px 10px; max-width:400px; margin:auto; }
    .card { background:white; border-radius:10px; padding:15px; margin-bottom:20px; box-shadow:0 2px 8px #ccc; }
    button { background:#006600; color:white; border:none; padding:12px; border-radius:8px; font-size:16px; cursor:pointer; width:100%; margin:8px 0; }
    button.yellow { background:#ffcc00; color:black; }
    .bottom-menu { position:fixed; bottom:0; width:100%; background:white; display:flex; border-top:1px solid #ccc; box-shadow:0 -2px 5px #aaa; }
    .bottom-menu div { flex:1; text-align:center; padding:12px 0; color:#555; }
    .bottom-menu .active { color:#006600; font-weight:bold; }
    .bottom-menu span { font-size:1.9rem; display:block; }
    .hidden { display:none; }
    .map-area { height:250px; border-radius:10px; overflow:hidden; margin:10px 0; }
    .rain-alert { background:#660066; color:white; padding:12px; margin:80px 15px 15px 15px; border-radius:8px; text-align:center; }
  </style>
</head>
<body>

<header>
  <h1>Eco Kenya</h1>
  <div class="points-box">
    Points: <span id="myPoints">000</span> 🔥 | Level: Mtaani Rookie
  </div>
  <button onclick="toggleDark()">Dark Mode</button>
</header>

<div class="rain-alert">
  ⚠️ Mvua inakuja – funika bin na report drain blocked haraka!
</div>

<main>
  <section id="home">
    <div class="card">
      <h2>Leo Collection</h2>
      <p>Mixed & plastics: 6pm - 9pm</p>
      <p>Organic: kesho asubuhi</p>
      <p>Near: Kibos Rd / Kondele</p>
    </div>

    <div class="card">
      <h2>Drop-off Karibu</h2>
      <p>Zoom uone Kondele West, Obunga etc</p>
      <div class="map-area" id="map"></div>
    </div>

    <div style="display:grid; grid-template-columns:1fr 1fr; gap:12px;">
      <button onclick="showPage('report')">Report 📸</button>
      <button onclick="showPage('schedule')">Schedule 🗓️</button>
      <button class="yellow" onclick="showPage('recycle')">Sell ♻️</button>
      <button onclick="alert('Truck tracking inakuja soon')">Track 🚛</button>
    </div>

    <div class="card">
      <h2>Top Recyclers</h2>
      <p>1. Kerry Brian - 1240kg</p>
      <p>2. Laurence Anan - 980 points</p>
      <p>3. Elijah Okono - grind tu boss</p>
    </div>
  </section>

  <section id="report" class="card hidden">
    <h2>Report Dustbin</h2>
    <p>Estate:</p>
    <input type="text" placeholder="Manyatta B" id="loc">

    <p>Type:</p>
    <select id="type">
      <option>Mixed</option>
      <option>Plastics</option>
      <option>Organic</option>
    </select>

    <p>Maoni:</p>
    <textarea id="notes" rows="4" placeholder="Inanuka sana..."></textarea>

    <button onclick="sendReport()">Send</button>
  </section>

  <section id="schedule" class="card hidden">
    <h2>Schedule</h2>
    <p>Area:</p>
    <select>
      <option>Kondele</option>
      <option>Milimani</option>
      <option>CBD</option>
    </select>
    <p style="margin-top:15px;">Plastics: 2× week<br>Mixed: daily CBD</p>
  </section>

  <section id="recycle" class="card hidden">
    <h2>Sell Taka</h2>
    <ul>
      <li>Plastics: KSh 15-25/kg</li>
      <li>Cardboard: KSh 8-12/kg</li>
      <li>Metals: KSh 40-70/kg</li>
    </ul>
    <button onclick="alert('call your local recycler')">Book</button>
  </section>
</main>

<div class="bottom-menu">
  <div class="active" onclick="showPage('home', this)">
    <span>🏠</span> Home
  </div>
  <div onclick="showPage('report', this)">
    <span>📸</span> Report
  </div>
  <div onclick="showPage('recycle', this)">
    <span>♻️</span> Sell
  </div>
  <div onclick="showPage('schedule', this)">
    <span>🗓️</span> Sch
  </div>
</div>

<script>
const firebaseConfig = {
  apiKey: "AIzaSyCs-lvezMJAY07KLr4dAQHq-Waty7IuYo",
  authDomain: "eco-kenya.firebaseapp.com",
  databaseURL: "https://eco-kenya-default-rtdb.firebaseio.com",
  projectId: "eco-kenya",
  storageBucket: "eco-kenya.appspot.com",
  messagingSenderId: "568012899201",
  appId: "1:568012899201:web:2fd56fcdfd8d4cd8fde80",
  measurementId: "G-QHHH4YFTV"
};

firebase.initializeApp(firebaseConfig);
const auth = firebase.auth();
const db = firebase.database();

auth.signInAnonymously().catch(err => console.error('Auth error:', err));

let points = 0;
document.getElementById('myPoints').innerText = points.toString().padStart(3, '0');

function showPage(page, clickedDiv) {
  document.querySelectorAll('section').forEach(sec => sec.classList.add('hidden'));
  document.getElementById(page).classList.remove('hidden');
  document.querySelectorAll('.bottom-menu div').forEach(n => n.classList.remove('active'));
  if (clickedDiv) clickedDiv.classList.add('active');
}

function toggleDark() {
  document.body.classList.toggle('dark-mode');
}

function getLocation(callback) {
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(
      pos => callback(pos.coords.latitude, pos.coords.longitude),
      err => {
        alert("Location access denied. Using default.");
        callback(-1.286389, 36.817223);
      }
    );
  } else {
    alert("Geolocation not supported.");
    callback(-1.286389, 36.817223);
  }
}

function sendReport() {
  const type = document.getElementById('type').value;
  const notes = document.getElementById('notes').value.trim() || 'No notes';

  getLocation((lat, lng) => {
    fetch(`https://nominatim.openstreetmap.org/reverse?format=json&lat=\( {lat}&lon= \){lng}&zoom=18&addressdetails=1`)
      .then(res => res.json())
      .then(data => {
        const address = data.display_name || 'Unknown location';

        const user = auth.currentUser;
        const userId = user ? user.uid : 'anonymous';

        db.ref('reports').push({
          location: address,
          type: type,
          notes: notes,
          lat: lat,
          lng: lng,
          timestamp: Date.now(),
          userId: userId
        }).then(() => {
          alert(`Report sent from ${address}! +50 points 🔥`);
          points += 50;
          document.getElementById('myPoints').innerText = points.toString().padStart(3, '0');
          showPage('home', document.querySelector('.bottom-menu div.active'));
        });
      })
      .catch(() => {
        alert("Could not get address. Report sent anyway.");
        db.ref('reports').push({
          location: 'Unknown',
          type: type,
          notes: notes,
          lat: lat,
          lng: lng,
          timestamp: Date.now(),
          userId: auth.currentUser?.uid || 'anonymous'
        }).then(() => {
          points += 50;
          document.getElementById('myPoints').innerText = points.toString().padStart(3, '0');
          showPage('home', document.querySelector('.bottom-menu div.active'));
        });
      });
  });
}
</script>

©️ 2026 Xaverian Secondary School 🇰🇪

</body>
</html># ecokenya