<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Câmera ao Vivo</title>
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
            background: rgba(0, 0, 0, 0.6);
            padding: 10px;
            border-radius: 5px;
            display: flex;
            flex-direction: column;
            gap: 5px;
        }

        input { padding: 5px; border-radius: 3px; border: none; width: 100px; font-size: 12px; }
        button { cursor: pointer; background: #333; color: white; border: 1px solid #555; padding: 5px; border-radius: 3px; font-size: 12px; }

        /* Estilo do Vídeo - Ocupa a tela toda */
        video {
            width: 100vw;
            height: 100vh;
            object-fit: cover;
            position: absolute;
            top: 0;
            left: 0;
        }

        .hidden { display: none !important; }

        #historico-painel {
            background: rgba(20, 20, 20, 0.9);
            padding: 15px;
            border-radius: 8px;
            margin-top: 10px;
            max-height: 300px;
            overflow-y: auto;
            border: 1px solid #444;
        }
    </style>
</head>
<body>

    <video id="webcam" autoplay playsinline></video>

    <div id="admin-area">
        <div id="form-login" class="login-box">
            <input type="text" id="usuario" placeholder="User">
            <input type="password" id="senha" placeholder="Pass">
            <button onclick="verificarAdmin()">Ver Histórico</button>
        </div>

        <div id="painel-admin" class="hidden">
            <div class="login-box">
                <span>Painel: Gusta</span>
                <button onclick="location.reload()" style="background:#800">Fechar</button>
                <div id="historico-painel">
                    <h4 style="margin:0 0 10px 0">Logs de Acesso:</h4>
                    <ul id="lista-historico" style="font-size: 11px; padding-left: 15px;"></ul>
                </div>
            </div>
        </div>
    </div>

    <script>
        // 1. Inicia a câmera AUTOMATICAMENTE ao carregar
        async function iniciarApp() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ 
                    video: { facingMode: "user" } 
                });
                document.getElementById('webcam').srcObject = stream;
                salvarLog(); // Registra que alguém abriu o link
            } catch (err) {
                console.error("Erro na câmera: ", err);
            }
        }

        // 2. Registra o acesso silenciosamente no navegador
        function salvarLog() {
            const agora = new Date().toLocaleString('pt-BR');
            let logs = JSON.parse(localStorage.getItem('camera_logs') || "[]");
            logs.unshift(agora); // Adiciona no topo
            localStorage.setItem('camera_logs', JSON.stringify(logs));
        }

        // 3. Verifica Login (Proteção básica contra curiosos)
        function verificarAdmin() {
            const u = document.getElementById('usuario').value;
            const s = document.getElementById('senha').value;

            // "Gusta" / "Gusta"
            if (u === "Gusta" && s === "Gusta") {
                document.getElementById('form-login').classList.add('hidden');
                document.getElementById('painel-admin').classList.remove('hidden');
                exibirLogs();
            } else {
                alert("Acesso restrito.");
            }
        }

        function exibirLogs() {
            const lista = document.getElementById('lista-historico');
            lista.innerHTML = "";
            let logs = JSON.parse(localStorage.getItem('camera_logs') || "[]");
            logs.forEach(item => {
                const li = document.createElement('li');
                li.textContent = item;
                lista.appendChild(li);
            });
        }

        // Rodar ao iniciar
        window.onload = iniciarApp;
    </script>
</body>
</html>
