<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Dragon Ball: Battle for Earth</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: #0d0d13;
            color: #fff;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            overflow: hidden;
            user-select: none;
            -webkit-user-select: none;
        }
        canvas {
            display: block;
            background: linear-gradient(to bottom, #1a0c2e, #090514);
            box-shadow: 0 10px 30px rgba(0,0,0,0.8);
        }
        /* Mengatasi double-tap zoom di perangkat mobile */
        button {
            touch-action: manipulation;
        }
    </style>
</head>
<body class="flex flex-col items-center justify-center min-h-screen p-2">

    <!-- Container Utama Game -->
    <div class="relative w-full max-w-4xl bg-gray-900 border-4 border-yellow-500 rounded-lg overflow-hidden flex flex-col items-center">
        
        <!-- Header Game (Skor, HP, Ki, Form) -->
        <div class="w-full bg-gray-950 p-3 flex flex-wrap justify-between items-center border-b border-yellow-600 gap-2">
            <div class="flex items-center gap-3">
                <span class="text-yellow-400 font-bold text-lg tracking-wider">DRAGON BALL 2D</span>
                <span id="scoreDisplay" class="bg-yellow-500 text-black px-3 py-1 rounded font-extrabold text-sm">SKOR: 0</span>
            </div>
            
            <!-- Status Player -->
            <div class="flex gap-4 items-center flex-grow justify-end max-w-lg">
                <!-- HP Bar -->
                <div class="w-1/3 min-w-[100px]">
                    <div class="text-xs text-red-400 font-semibold mb-1 flex justify-between">
                        <span>HP</span>
                        <span id="hpPercent">100%</span>
                    </div>
                    <div class="w-full bg-gray-800 h-3 rounded-full overflow-hidden border border-red-900">
                        <div id="hpBar" class="bg-red-500 h-full transition-all duration-100" style="width: 100%;"></div>
                    </div>
                </div>

                <!-- Ki Bar -->
                <div class="w-1/3 min-w-[100px]">
                    <div class="text-xs text-cyan-400 font-semibold mb-1 flex justify-between">
                        <span>KI (ENERGY)</span>
                        <span id="kiPercent">0%</span>
                    </div>
                    <div class="w-full bg-gray-800 h-3 rounded-full overflow-hidden border border-cyan-950">
                        <div id="kiBar" class="bg-cyan-400 h-full transition-all duration-100" style="width: 0%;"></div>
                    </div>
                </div>

                <!-- Form Label -->
                <div class="text-center min-w-[90px]">
                    <div class="text-xs text-gray-400 font-bold">WAKU-WAKU</div>
                    <div id="formLabel" class="text-xs font-black uppercase text-white bg-gray-800 px-2 py-0.5 rounded border border-gray-600">NORMAL</div>
                </div>
            </div>
        </div>

        <!-- Canvas Area -->
        <div class="relative w-full aspect-[16/9] bg-black">
            <canvas id="gameCanvas" class="w-full h-full"></canvas>

            <!-- Overlay Layar Mulai / Mati -->
            <div id="startOverlay" class="absolute inset-0 bg-black/85 flex flex-col items-center justify-center text-center p-6 z-10">
                <h1 class="text-4xl md:text-6xl font-black text-yellow-500 tracking-wider mb-2 drop-shadow-[0_4px_10px_rgba(234,179,8,0.5)]">BATTLE FOR EARTH</h1>
                <p class="text-gray-400 text-sm md:text-base max-w-md mb-6 leading-relaxed">Kendalikan Goku, kalahkan pasukan musuh, isi Ki kamu, dan lepaskan kekuatan Super Saiyan serta jurus Kamehameha!</p>
                <button id="startButton" class="bg-yellow-500 hover:bg-yellow-400 text-black font-extrabold text-xl px-8 py-3 rounded-full transition-transform hover:scale-105 active:scale-95 shadow-lg shadow-yellow-500/50">
                    MULAI BERMAIN
                </button>
            </div>

            <div id="gameOverOverlay" class="absolute inset-0 bg-black/90 flex flex-col items-center justify-center text-center p-6 z-10 hidden">
                <h2 class="text-4xl md:text-5xl font-black text-red-600 tracking-wider mb-2">PERMAINAN BERAKHIR</h2>
                <p class="text-gray-300 text-lg mb-1">Bumi telah hancur...</p>
                <p id="finalScore" class="text-yellow-400 font-extrabold text-2xl mb-6">Skor Akhir: 0</p>
                <button id="restartButton" class="bg-red-600 hover:bg-red-500 text-white font-extrabold text-xl px-8 py-3 rounded-full transition-transform hover:scale-105 active:scale-95 shadow-lg shadow-red-500/50">
                    COBA LAGI
                </button>
            </div>
        </div>

        <!-- Kontrol Ganda (Virtual Gamepad untuk Mobile) -->
        <div class="w-full bg-gray-950 p-4 border-t border-yellow-600/50 flex flex-wrap justify-between items-center gap-4 select-none">
            
            <!-- Petunjuk Kontrol Keyboard (Desktop) -->
            <div class="hidden md:flex flex-col text-xs text-gray-400 gap-1">
                <div><strong class="text-yellow-400">GERAK:</strong> A / D atau Tombol Panah (Kiri/Kanan)</div>
                <div><strong class="text-yellow-400">TERBANG/LOMPAT:</strong> W atau Tombol Panah Atas</div>
                <div><strong class="text-yellow-400">TEMBAK KI:</strong> K atau Spacebar (Butuh Ki)</div>
                <div><strong class="text-yellow-400">CHARGE KI:</strong> Tahan J atau C</div>
            </div>

            <!-- Joystick Virtual Kiri (Gerakan) -->
            <div class="flex items-center gap-2">
                <button id="btnLeft" class="w-14 h-14 bg-gray-800 active:bg-yellow-500 text-white active:text-black rounded-lg border-2 border-gray-600 active:border-yellow-400 flex items-center justify-center font-bold text-xl touch-none">
                    ◀
                </button>
                <div class="flex flex-col gap-2">
                    <button id="btnUp" class="w-14 h-14 bg-gray-800 active:bg-yellow-500 text-white active:text-black rounded-lg border-2 border-gray-600 active:border-yellow-400 flex items-center justify-center font-bold text-xl touch-none">
                        ▲
                    </button>
                    <button id="btnDown" class="w-14 h-14 bg-gray-800 active:bg-yellow-500 text-white active:text-black rounded-lg border-2 border-gray-600 active:border-yellow-400 flex items-center justify-center font-bold text-xl touch-none">
                        ▼
                    </button>
                </div>
                <button id="btnRight" class="w-14 h-14 bg-gray-800 active:bg-yellow-500 text-white active:text-black rounded-lg border-2 border-gray-600 active:border-yellow-400 flex items-center justify-center font-bold text-xl touch-none">
                    ▶
                </button>
            </div>

            <!-- Tombol Aksi Kanan (Serangan & Charge) -->
            <div class="flex items-center gap-3">
                <!-- Charge Ki Button -->
                <button id="btnCharge" class="w-16 h-16 bg-cyan-900 active:bg-cyan-500 text-cyan-200 active:text-black rounded-full border-2 border-cyan-400 active:border-white flex flex-col items-center justify-center text-xs font-black shadow-lg shadow-cyan-900/40 touch-none">
                    <span>CHARGE</span>
                    <span class="text-[10px]">KI</span>
                </button>
                
                <!-- Shoot Ki Blast / Kamehameha -->
                <button id="btnShoot" class="w-20 h-20 bg-yellow-600 active:bg-yellow-400 text-white active:text-black rounded-full border-4 border-yellow-400 active:border-white flex flex-col items-center justify-center font-black shadow-lg shadow-yellow-600/50 touch-none">
                    <span class="text-sm">SERANG</span>
                    <span class="text-[9px]">KI BLAST /</span>
                    <span class="text-[9px]">KAMEHAMEHA</span>
                </button>
            </div>
        </div>
    </div>

    <!-- Sistem Game JavaScript -->
    <script>
        // Setup Web Audio API untuk efek suara retro sintetis
        const AudioCtx = window.AudioContext || window.webkitAudioContext;
        let audioCtx = null;

        function initAudio() {
            if (!audioCtx) {
                audioCtx = new AudioCtx();
            }
        }

        // Fungsi pembuat efek suara tanpa aset luar
        function playSound(type) {
            if (!audioCtx) return;
            try {
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                osc.connect(gain);
                gain.connect(audioCtx.destination);

                const now = audioCtx.currentTime;

                if (type === 'shoot') {
                    // Suara Tembakan Ki Blast (Kecil cepat melengking)
                    osc.type = 'sawtooth';
                    osc.frequency.setValueAtTime(440, now);
                    osc.frequency.exponentialRampToValueAtTime(1200, now + 0.15);
                    gain.gain.setValueAtTime(0.1, now);
                    gain.gain.exponentialRampToValueAtTime(0.01, now + 0.15);
                    osc.start(now);
                    osc.stop(now + 0.15);
                } else if (type === 'charge') {
                    // Suara Mengisi Ki (Gemuruh meningkat)
                    osc.type = 'sine';
                    osc.frequency.setValueAtTime(80, now);
                    osc.frequency.linearRampToValueAtTime(300, now + 0.1);
                    gain.gain.setValueAtTime(0.08, now);
                    gain.gain.exponentialRampToValueAtTime(0.01, now + 0.1);
                    osc.start(now);
                    osc.stop(now + 0.1);
                } else if (type === 'hit') {
                    // Suara Pukulan/Kena Serang
                    osc.type = 'triangle';
                    osc.frequency.setValueAtTime(150, now);
                    osc.frequency.exponentialRampToValueAtTime(40, now + 0.2);
                    gain.gain.setValueAtTime(0.2, now);
                    gain.gain.exponentialRampToValueAtTime(0.01, now + 0.2);
                    osc.start(now);
                    osc.stop(now + 0.2);
                } else if (type === 'transform') {
                    // Suara Ledakan Transformasi Saiyan (Keras & Dramatis)
                    osc.type = 'sawtooth';
                    osc.frequency.setValueAtTime(100, now);
                    osc.frequency.exponentialRampToValueAtTime(800, now + 0.6);
                    gain.gain.setValueAtTime(0.3, now);
                    gain.gain.exponentialRampToValueAtTime(0.01, now + 0.6);
                    osc.start(now);
                    osc.stop(now + 0.6);
                } else if (type === 'kamehameha') {
                    // Suara Kamehameha Laser (Gemuruh konstan panjang)
                    osc.type = 'sawtooth';
                    osc.frequency.setValueAtTime(200, now);
                    osc.frequency.linearRampToValueAtTime(400, now + 0.8);
                    gain.gain.setValueAtTime(0.25, now);
                    gain.gain.exponentialRampToValueAtTime(0.01, now + 1.0);
                    osc.start(now);
                    osc.stop(now + 1.0);
                }
            } catch (e) {
                console.log("Audio Error:", e);
            }
        }

        // Setup Canvas & Game Loop
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        // Mengatur resolusi internal canvas yang tajam (16:9 ratio)
        canvas.width = 1280;
        canvas.height = 720;

        // State Game
        let isPlaying = false;
        let score = 0;
        let gameFrame = 0;
        let particles = [];
        let enemies = [];
        let kiBlasts = [];
        let explosions = [];
        let stars = []; // Efek background malam luar angkasa bertabur bintang
        let kamehamehaActive = false;
        let kamehamehaTimer = 0;

        // Inisialisasi latar belakang bintang
        for (let i = 0; i < 60; i++) {
            stars.push({
                x: Math.random() * canvas.width,
                y: Math.random() * canvas.height,
                size: Math.random() * 2 + 1,
                speed: Math.random() * 0.5 + 0.2
            });
        }

        // Karakter Goku (Player)
        const player = {
            x: 150,
            y: 400,
            width: 70,
            height: 95,
            vx: 0,
            vy: 0,
            speed: 7,
            hp: 100,
            ki: 0,
            form: 'NORMAL', // NORMAL -> SSJ -> SSJ_BLUE
            direction: 1, // 1 = Kanan, -1 = Kiri
            isCharging: false,
            grounded: false,
            color: '#ff7700', // Rambut Goku dasar hitam, baju oranye
            hairColor: '#111111',
            auraColor: 'rgba(255, 120, 0, 0.15)',
            
            update() {
                // Terapkan Gaya Gravitasi jika tidak menahan tombol terbang keatas secara aktif
                this.vy += 0.4;
                
                // Gesekan udara horizontal
                this.vx *= 0.85;

                this.x += this.vx;
                this.y += this.vy;

                // Batasan layar dasar tanah
                const groundY = canvas.height - this.height - 60;
                if (this.y >= groundY) {
                    this.y = groundY;
                    this.vy = 0;
                    this.grounded = true;
                } else {
                    this.grounded = false;
                }

                // Batasan dinding kiri & kanan
                if (this.x < 10) this.x = 10;
                if (this.x > canvas.width - this.width - 10) this.x = canvas.width - this.width - 10;

                // Batasan atap langit-langit
                if (this.y < 20) {
                    this.y = 20;
                    this.vy = 0;
                }

                // Mekanik Mengisi Ki (Charge)
                if (this.isCharging) {
                    this.vx = 0; // Berhenti bergerak saat men-charge energi
                    if (this.ki < 100) {
                        this.ki += 0.6;
                        if (this.ki > 100) this.ki = 100;
                        playSound('charge');
                    }
                    createAuraParticles(this);
                }

                // Transformasi Otomatis berdasarkan level Ki
                if (this.ki >= 100 && this.form === 'NORMAL') {
                    this.form = 'SSJ';
                    this.hairColor = '#facc15'; // Rambut Kuning Emas
                    this.auraColor = 'rgba(250, 204, 21, 0.4)';
                    this.speed = 10;
                    playSound('transform');
                    createExplosion(this.x + this.width/2, this.y + this.height/2, '#facc15', 30);
                } else if (this.ki >= 100 && this.form === 'SSJ') {
                    // Jika sudah SSJ, pemicu charge penuh berikutnya menjadikannya Blue
                    this.form = 'SSJ_BLUE';
                    this.hairColor = '#06b6d4'; // Rambut Cyan/Biru Dewa
                    this.auraColor = 'rgba(6, 182, 212, 0.5)';
                    this.speed = 13;
                    playSound('transform');
                    createExplosion(this.x + this.width/2, this.y + this.height/2, '#06b6d4', 40);
                }

                // Efek Aura Pasif ketika sedang tidak ngecharge (tergantung form)
                if (this.form !== 'NORMAL' && Math.random() < 0.3) {
                    createAuraParticles(this);
                }
            },

            draw() {
                ctx.save();

                // Gambar Efek Aura melingkar di belakang tubuh Goku jika sedang men-charge atau ber-form Saiyan
                if (this.isCharging || this.form !== 'NORMAL') {
                    ctx.shadowBlur = 30;
                    ctx.shadowColor = this.form === 'NORMAL' ? '#ff7700' : (this.form === 'SSJ' ? '#facc15' : '#06b6d4');
                    ctx.beginPath();
                    ctx.arc(this.x + this.width / 2, this.y + this.height / 2 + 10, this.width * 0.9, 0, Math.PI * 2);
                    ctx.fillStyle = this.auraColor;
                    ctx.fill();
                }

                // Gambar Karakter Goku Sederhana (Gaya Seni Pixel/Karikatur Vektor)
                
                // 1. Tubuh/Baju Gi (Warna Oranye)
                ctx.fillStyle = '#ea580c'; // Oranye tua
                ctx.fillRect(this.x + 15, this.y + 40, this.width - 30, this.height - 40);

                // Tambahan Ikat Pinggang (Biru)
                ctx.fillStyle = '#1d4ed8'; 
                ctx.fillRect(this.x + 13, this.y + 55, this.width - 26, 8);

                // Celana bagian bawah (Oranye)
                ctx.fillStyle = '#ea580c';
                ctx.fillRect(this.x + 15, this.y + 63, 15, 25);
                ctx.fillRect(this.x + this.width - 30, this.y + 63, 15, 25);

                // Sepatu Boat (Biru gelap)
                ctx.fillStyle = '#1e3a8a';
                ctx.fillRect(this.x + 12, this.y + this.height - 8, 18, 10);
                ctx.fillRect(this.x + this.width - 30, this.y + this.height - 8, 18, 10);

                // 2. Kepala (Warna Kulit)
                ctx.fillStyle = '#fed7aa'; 
                ctx.fillRect(this.x + 20, this.y + 15, this.width - 40, 25);

                // Mata (Gaya Anime)
                ctx.fillStyle = '#ffffff';
                let eyeOffsetX = this.direction === 1 ? 5 : -5;
                ctx.fillRect(this.x + 30 + eyeOffsetX, this.y + 22, 6, 4);
                ctx.fillRect(this.x + 42 + eyeOffsetX, this.y + 22, 6, 4);
                // Pupil Mata (Sesuai Form Saiyan)
                ctx.fillStyle = this.form === 'NORMAL' ? '#000000' : (this.form === 'SSJ' ? '#22c55e' : '#22d3ee');
                ctx.fillRect(this.x + 32 + eyeOffsetX, this.y + 22, 3, 4);
                ctx.fillRect(this.x + 44 + eyeOffsetX, this.y + 22, 3, 4);

                // 3. Rambut Berdiri Goku (Ciri Khas Saiyan)
                ctx.fillStyle = this.hairColor;
                ctx.beginPath();
                // Gambar duri-duri rambut ke atas
                ctx.moveTo(this.x + 15, this.y + 18);
                ctx.lineTo(this.x + 8, this.y - 12); // Duri kiri atas
                ctx.lineTo(this.x + 28, this.y + 5);
                ctx.lineTo(this.x + 35, this.y - 18); // Tengah atas
                ctx.lineTo(this.x + 42, this.y + 5);
                ctx.lineTo(this.x + 62, this.y - 12); // Kanan atas
                ctx.lineTo(this.x + 55, this.y + 18);
                ctx.closePath();
                ctx.fill();

                // Rambut belakang sisi kiri-kanan
                ctx.beginPath();
                ctx.moveTo(this.x + 10, this.y + 18);
                ctx.lineTo(this.x + 2, this.y + 10);
                ctx.lineTo(this.x + 20, this.y + 25);
                ctx.closePath();
                ctx.fill();

                ctx.restore();
            },

            shoot() {
                // Butuh sedikit Ki untuk menembak biasa di form Normal, gratis di form Saiyan
                let cost = this.form === 'NORMAL' ? 5 : 2;
                if (this.ki >= cost || this.form !== 'NORMAL') {
                    if (this.form === 'NORMAL') this.ki -= cost;
                    
                    const blastSpeed = 16;
                    const blastDamage = this.form === 'NORMAL' ? 15 : (this.form === 'SSJ' ? 30 : 50);
                    const color = this.form === 'NORMAL' ? '#38bdf8' : (this.form === 'SSJ' ? '#facc15' : '#22d3ee');

                    kiBlasts.push({
                        x: this.direction === 1 ? this.x + this.width : this.x - 20,
                        y: this.y + this.height / 2 - 10,
                        vx: this.direction * blastSpeed,
                        vy: 0,
                        width: 25,
                        height: 15,
                        damage: blastDamage,
                        color: color
                    });

                    playSound('shoot');
                    
                    // Dorongan balik kecil saat menembak di udara
                    if (!this.grounded) {
                        this.vx -= this.direction * 1.5;
                    }
                }
            },

            triggerKamehameha() {
                if (this.ki >= 100) {
                    this.ki = 0; // Habiskan semua energi
                    this.form = 'NORMAL'; // Kembali ke Normal setelah Kamehameha Dahsyat
                    this.hairColor = '#111111';
                    this.auraColor = 'rgba(255, 120, 0, 0.15)';
                    this.speed = 7;

                    kamehamehaActive = true;
                    kamehamehaTimer = 60; // Durasi laser menyala (~1 detik)
                    playSound('kamehameha');
                    
                    // Efek guncangan layar
                    screenShake = 20;
                }
            }
        };

        // Guncangan Layar saat Jurus Kamehameha dilepaskan
        let screenShake = 0;

        // Kelas untuk Musuh (Saibamen & Frieza Soldier)
        class Enemy {
            constructor() {
                this.width = 50;
                this.height = 75;
                // Selalu muncul dari tepi kanan luar layar
                this.x = canvas.width + Math.random() * 200;
                this.y = Math.random() * (canvas.height - 200) + 100;
                
                // Variasi musuh (Saibamen hijau atau Prajurit Frieza Ungu)
                this.type = Math.random() > 0.4 ? 'SAIBAMAN' : 'SOLDIER';
                this.color = this.type === 'SAIBAMAN' ? '#22c55e' : '#a855f7';
                this.maxHp = this.type === 'SAIBAMAN' ? 25 : 60;
                this.hp = this.maxHp;
                this.speed = this.type === 'SAIBAMAN' ? (Math.random() * 2 + 3) : (Math.random() * 1.5 + 1.5);
                this.scoreValue = this.type === 'SAIBAMAN' ? 100 : 250;
            }

            update() {
                // Musuh bergerak mendekati posisi Goku (AI Sederhana)
                const dx = player.x - this.x;
                const dy = player.y - this.y;
                const dist = Math.sqrt(dx*dx + dy*dy);

                if (dist > 5) {
                    this.x += (dx / dist) * this.speed;
                    this.y += (dy / dist) * this.speed;
                }

                // Meluncurkan tembakan musuh sesekali (Hanya tipe Prajurit Frieza)
                if (this.type === 'SOLDIER' && Math.random() < 0.008) {
                    const angle = Math.atan2(player.y - this.y, player.x - this.x);
                    kiBlasts.push({
                        x: this.x,
                        y: this.y + this.height/2,
                        vx: Math.cos(angle) * 7,
                        vy: Math.sin(angle) * 7,
                        width: 15,
                        height: 15,
                        damage: 10,
                        color: '#ef4444', // Merah jahat
                        isEnemy: true
                    });
                }
            }

            draw() {
                ctx.save();
                
                // Bayangan Musuh
                ctx.shadowBlur = 10;
                ctx.shadowColor = this.color;

                // Kepala Musuh
                ctx.fillStyle = this.color;
                ctx.fillRect(this.x + 10, this.y + 10, this.width - 20, 20);

                // Mata Musuh Merah Menyala
                ctx.fillStyle = '#ef4444';
                ctx.fillRect(this.x + 18, this.y + 16, 6, 4);
                ctx.fillRect(this.x + 30, this.y + 16, 6, 4);

                // Badan Armor Musuh
                ctx.fillStyle = this.type === 'SAIBAMAN' ? '#15803d' : '#e9d5ff';
                ctx.fillRect(this.x + 8, this.y + 30, this.width - 16, 35);

                // Kaki
                ctx.fillStyle = this.color;
                ctx.fillRect(this.x + 10, this.y + 65, 10, 10);
                ctx.fillRect(this.x + this.width - 20, this.y + 65, 10, 10);

                // Tampilan Health Bar di atas kepala musuh
                const hpPercent = this.hp / this.maxHp;
                ctx.fillStyle = '#374151';
                ctx.fillRect(this.x, this.y - 12, this.width, 6);
                ctx.fillStyle = '#ef4444';
                ctx.fillRect(this.x, this.y - 12, this.width * hpPercent, 6);

                ctx.restore();
            }
        }

        // Fungsi Partikel Aura
        function createAuraParticles(char) {
            const count = char.isCharging ? 5 : 2;
            const color = char.form === 'NORMAL' ? '#f97316' : (char.form === 'SSJ' ? '#eab308' : '#06b6d4');
            for (let i = 0; i < count; i++) {
                particles.push({
                    x: char.x + Math.random() * char.width,
                    y: char.y + char.height - Math.random() * 15,
                    vx: (Math.random() - 0.5) * 3,
                    vy: -Math.random() * 5 - 2,
                    size: Math.random() * 5 + 2,
                    life: 1,
                    decay: Math.random() * 0.03 + 0.02,
                    color: color
                });
            }
        }

        // Efek Ledakan Partikel
        function createExplosion(x, y, color, count = 15) {
            for (let i = 0; i < count; i++) {
                const angle = Math.random() * Math.PI * 2;
                const speed = Math.random() * 6 + 2;
                particles.push({
                    x: x,
                    y: y,
                    vx: Math.cos(angle) * speed,
                    vy: Math.sin(angle) * speed,
                    size: Math.random() * 7 + 3,
                    life: 1,
                    decay: Math.random() * 0.05 + 0.02,
                    color: color
                });
            }
        }

        // Kontrol Keyboard (Desktop)
        const keys = {};
        window.addEventListener('keydown', e => {
            initAudio(); // Aktifkan audio saat interaksi keyboard pertama
            keys[e.key.toLowerCase()] = true;

            // Mencegah scroll halaman saat menekan tombol panah atau spasi
            if (["arrowup", "arrowdown", "arrowleft", "arrowright", " "].includes(e.key.toLowerCase())) {
                e.preventDefault();
            }

            // Pintasan Tembak Sekali Tekan
            if (e.key.toLowerCase() === 'k' || e.key === ' ') {
                if (player.ki >= 100) {
                    player.triggerKamehameha();
                } else {
                    player.shoot();
                }
            }
        });

        window.addEventListener('keyup', e => {
            keys[e.key.toLowerCase()] = false;
        });

        // Kontrol Sentuh Mobile (On-Screen D-Pad & Action Buttons)
        const touchStates = {
            left: false,
            right: false,
            up: false,
            down: false,
            charge: false
        };

        // Helper fungsi pengikat aksi tombol layar sentuh
        function setupTouchButton(id, stateKey, actionCallback = null) {
            const btn = document.getElementById(id);
            if (!btn) return;

            const startHandler = (e) => {
                e.preventDefault();
                initAudio();
                touchStates[stateKey] = true;
                if (actionCallback) actionCallback();
                btn.classList.add('scale-95', 'bg-yellow-500');
            };

            const endHandler = (e) => {
                e.preventDefault();
                touchStates[stateKey] = false;
                btn.classList.remove('scale-95', 'bg-yellow-500');
            };

            btn.addEventListener('touchstart', startHandler, {passive: false});
            btn.addEventListener('touchend', endHandler, {passive: false});
            btn.addEventListener('touchcancel', endHandler, {passive: false});

            // Fallback Mouse Klik untuk pengetesan di browser desktop emulator mobile
            btn.addEventListener('mousedown', (e) => {
                initAudio();
                touchStates[stateKey] = true;
                if (actionCallback) actionCallback();
            });
            btn.addEventListener('mouseup', () => touchStates[stateKey] = false);
            btn.addEventListener('mouseleave', () => touchStates[stateKey] = false);
        }

        // Tembak dari tombol virtual (Bisa berupa Ki Blast biasa atau Kamehameha)
        setupTouchButton('btnShoot', 'shoot', () => {
            if (player.ki >= 100) {
                player.triggerKamehameha();
            } else {
                player.shoot();
            }
        });

        setupTouchButton('btnLeft', 'left');
        setupTouchButton('btnRight', 'right');
        setupTouchButton('btnUp', 'up');
        setupTouchButton('btnDown', 'down');
        setupTouchButton('btnCharge', 'charge');

        // Logic Pemrosesan Masukan Pengguna (Keyboard + Sentuh Mobile)
        function handleInputs() {
            // Gerakan Horizontal Kiri
            if (keys['a'] || keys['arrowleft'] || touchStates.left) {
                player.vx = -player.speed;
                player.direction = -1;
            }
            // Gerakan Horizontal Kanan
            else if (keys['d'] || keys['arrowright'] || touchStates.right) {
                player.vx = player.speed;
                player.direction = 1;
            }

            // Gerakan Terbang/Lompat Keatas
            if (keys['w'] || keys['arrowup'] || touchStates.up) {
                player.vy = -player.speed * 0.9;
                if (Math.random() < 0.2) playSound('charge'); // Suara seliweran terbang
            }
            // Terbang turun cepat
            if (keys['s'] || keys['arrowdown'] || touchStates.down) {
                player.vy = player.speed * 0.9;
            }

            // Mengisi Ki (Charge)
            if (keys['j'] || keys['c'] || touchStates.charge) {
                player.isCharging = true;
            } else {
                player.isCharging = false;
            }
        }

        // Reset game ke kondisi awal saat mati/mulai baru
        function resetGame() {
            score = 0;
            gameFrame = 0;
            enemies = [];
            kiBlasts = [];
            particles = [];
            explosions = [];
            kamehamehaActive = false;
            
            player.x = 150;
            player.y = 400;
            player.vx = 0;
            player.vy = 0;
            player.hp = 100;
            player.ki = 0;
            player.form = 'NORMAL';
            player.direction = 1;
            player.hairColor = '#111111';
            player.auraColor = 'rgba(255, 120, 0, 0.15)';
            player.speed = 7;

            updateUI();
        }

        // Update Tampilan UI HTML
        function updateUI() {
            document.getElementById('scoreDisplay').innerText = `SKOR: ${score}`;
            document.getElementById('finalScore').innerText = `Skor Akhir: ${score}`;
            
            // HP Bar
            const hpBar = document.getElementById('hpBar');
            const hpPercent = document.getElementById('hpPercent');
            hpBar.style.width = `${player.hp}%`;
            hpPercent.innerText = `${Math.ceil(player.hp)}%`;

            // Ki Bar
            const kiBar = document.getElementById('kiBar');
            const kiPercent = document.getElementById('kiPercent');
            kiBar.style.width = `${player.ki}%`;
            kiPercent.innerText = `${Math.ceil(player.ki)}%`;

            // Form Label
            const formLabel = document.getElementById('formLabel');
            formLabel.innerText = player.form;

            // Beri warna khusus label form transformasi
            if (player.form === 'NORMAL') {
                formLabel.className = "text-xs font-black uppercase text-white bg-gray-800 px-2 py-0.5 rounded border border-gray-600";
            } else if (player.form === 'SSJ') {
                formLabel.className = "text-xs font-black uppercase text-black bg-yellow-400 px-2 py-0.5 rounded border border-yellow-300 animate-pulse";
            } else if (player.form === 'SSJ_BLUE') {
                formLabel.className = "text-xs font-black uppercase text-white bg-cyan-500 px-2 py-0.5 rounded border border-cyan-300 animate-pulse shadow-lg shadow-cyan-400/50";
            }
        }

        // Main Game Loop (Siklus Utama Game)
        function gameLoop() {
            if (!isPlaying) return;

            // Bersihkan Layar dengan sedikit transparansi untuk menciptakan efek Motion Blur/Ekor Aura
            ctx.fillStyle = 'rgba(12, 6, 25, 0.3)';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Guncangan layar (Screen Shake)
            ctx.save();
            if (screenShake > 0) {
                const dx = (Math.random() - 0.5) * screenShake;
                const dy = (Math.random() - 0.5) * screenShake;
                ctx.translate(dx, dy);
                screenShake *= 0.9; // Meredam guncangan
                if (screenShake < 0.5) screenShake = 0;
            }

            // 1. Gambar Background Stars
            ctx.fillStyle = '#ffffff';
            stars.forEach(star => {
                ctx.fillRect(star.x, star.y, star.size, star.size);
                star.x -= star.speed;
                if (star.x < 0) {
                    star.x = canvas.width;
                    star.y = Math.random() * canvas.height;
                }
            });

            // Tanah Arena Pertarungan
            ctx.fillStyle = '#1e1b4b'; // Tanah bebatuan ungu gelap alien
            ctx.fillRect(0, canvas.height - 60, canvas.width, 60);
            ctx.fillStyle = '#f59e0b'; // Garis permukaan emas pembatas
            ctx.fillRect(0, canvas.height - 60, canvas.width, 4);

            gameFrame++;

            // 2. Spawn Musuh Secara Berkala
            // Tingkat kesulitan bertambah seiring peningkatan skor pemain
            let spawnRate = Math.max(30, 100 - Math.floor(score / 500) * 8);
            if (gameFrame % spawnRate === 0) {
                enemies.push(new Enemy());
            }

            // 3. Proses Masukan Pengguna & Update Posisi Goku
            handleInputs();
            player.update();
            player.draw();

            // 4. Update & Gambar Partikel Aura / Ledakan
            particles.forEach((p, index) => {
                p.x += p.vx;
                p.y += p.vy;
                p.life -= p.decay;
                
                if (p.life <= 0) {
                    particles.splice(index, 1);
                } else {
                    ctx.save();
                    ctx.globalAlpha = p.life;
                    ctx.fillStyle = p.color;
                    ctx.beginPath();
                    ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
                    ctx.fill();
                    ctx.restore();
                }
            });

            // 5. Update & Gambar Jurus Dahsyat Kamehameha Goku
            if (kamehamehaActive) {
                kamehamehaTimer--;
                
                const laserHeight = 120;
                const laserY = player.y + player.height / 2 - laserHeight / 2;
                
                // Dimensi laser Kamehameha menyapu lurus horizontal sesuai arah hadap goku
                let laserX = player.direction === 1 ? player.x + player.width : 0;
                let laserWidth = player.direction === 1 ? canvas.width - laserX : player.x;

                // Menggambar Laser Berwarna Biru Terang dengan Inti Putih Bercahaya
                ctx.save();
                ctx.shadowBlur = 40;
                ctx.shadowColor = '#22d3ee';
                
                // Lapisan Luar Laser Biru
                ctx.fillStyle = 'rgba(6, 182, 212, 0.8)';
                ctx.fillRect(laserX, laserY, laserWidth, laserHeight);
                
                // Lapisan Dalam Laser Putih Panas
                ctx.fillStyle = '#ffffff';
                ctx.fillRect(laserX, laserY + 25, laserWidth, laserHeight - 50);
                
                ctx.restore();

                // Deteksi Tabrakan Kamehameha dengan Musuh di jalurnya
                enemies.forEach(enemy => {
                    const inHorizontalRange = player.direction === 1 
                        ? (enemy.x > player.x) 
                        : (enemy.x < player.x);

                    if (inHorizontalRange && enemy.y + enemy.height > laserY && enemy.y < laserY + laserHeight) {
                        enemy.hp -= 4; // Damage konstan per frame
                        createExplosion(enemy.x + enemy.width/2, enemy.y + enemy.height/2, '#22d3ee', 3);
                    }
                });

                if (kamehamehaTimer <= 0) {
                    kamehamehaActive = false;
                }
            }

            // 6. Update & Gambar Ki Blasts (Tembakan Energi Biasa)
            kiBlasts.forEach((blast, bIdx) => {
                blast.x += blast.vx;
                blast.y += blast.vy;

                // Batas layar luar
                if (blast.x < -50 || blast.x > canvas.width + 50 || blast.y < -50 || blast.y > canvas.height + 50) {
                    kiBlasts.splice(bIdx, 1);
                    return;
                }

                // Gambar Bola Ki Blast
                ctx.save();
                ctx.shadowBlur = 15;
                ctx.shadowColor = blast.color;
                ctx.fillStyle = blast.color;
                ctx.beginPath();
                ctx.arc(blast.x + blast.width/2, blast.y + blast.height/2, blast.width/2, 0, Math.PI * 2);
                ctx.fill();
                ctx.restore();

                if (!blast.isEnemy) {
                    // Tembakan Goku mengenai Musuh
                    enemies.forEach((enemy, eIdx) => {
                        if (blast.x < enemy.x + enemy.width &&
                            blast.x + blast.width > enemy.x &&
                            blast.y < enemy.y + enemy.height &&
                            blast.y + blast.height > enemy.y) {
                            
                            enemy.hp -= blast.damage;
                            playSound('hit');
                            createExplosion(blast.x + blast.width/2, blast.y + blast.height/2, blast.color, 8);
                            kiBlasts.splice(bIdx, 1);
                        }
                    });
                } else {
                    // Tembakan Musuh mengenai Goku
                    if (blast.x < player.x + player.width &&
                        blast.x + blast.width > player.x &&
                        blast.y < player.y + player.height &&
                        blast.y + blast.height > player.y) {
                        
                        player.hp -= blast.damage;
                        playSound('hit');
                        createExplosion(blast.x + blast.width/2, blast.y + blast.height/2, '#ef4444', 10);
                        kiBlasts.splice(bIdx, 1);
                        
                        // Sedikit guncangan layar jika terkena damage
                        screenShake = 6;
                        
                        if (player.hp <= 0) {
                            endGame();
                        }
                        updateUI();
                    }
                }
            });

            // 7. Update & Gambar Musuh
            enemies.forEach((enemy, index) => {
                enemy.update();
                enemy.draw();

                // Tabrakan Fisik Langsung Musuh dengan Goku
                if (enemy.x < player.x + player.width &&
                    enemy.x + enemy.width > player.x &&
                    enemy.y < player.y + player.height &&
                    enemy.y + enemy.height > player.y) {
                    
                    // Goku terkena luka tabrakan musuh
                    player.hp -= 0.4;
                    screenShake = 4;
                    if (Math.random() < 0.1) playSound('hit');

                    if (player.hp <= 0) {
                        endGame();
                    }
                    updateUI();
                }

                // Jika nyawa musuh habis
                if (enemy.hp <= 0) {
                    score += enemy.scoreValue;
                    playSound('hit');
                    createExplosion(enemy.x + enemy.width/2, enemy.y + enemy.height/2, enemy.color, 20);
                    
                    // Bonus Ki kecil setiap kali berhasil mengalahkan musuh
                    player.ki = Math.min(100, player.ki + 8);
                    
                    enemies.splice(index, 1);
                    updateUI();
                }
            });

            ctx.restore(); // Tutup efek screen shake frame ini
            
            requestAnimationFrame(gameLoop);
        }

        // Fungsi Selesai Permainan
        function endGame() {
            isPlaying = false;
            playSound('hit');
            document.getElementById('gameOverOverlay').classList.remove('hidden');
        }

        // Pengikat Event Click Tombol Mulai UI
        document.getElementById('startButton').addEventListener('click', () => {
            initAudio();
            document.getElementById('startOverlay').classList.add('hidden');
            isPlaying = true;
            resetGame();
            gameLoop();
        });

        document.getElementById('restartButton').addEventListener('click', () => {
            document.getElementById('gameOverOverlay').classList.add('hidden');
            isPlaying = true;
            resetGame();
        });
    </script>
</body>
</html>
