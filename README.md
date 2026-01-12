<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Painel de Controle - Gusta</title>
    <style>
        body { font-family: sans-serif; background: #000; color: white; margin: 0; padding: 20px; }
        h2 { color: #007bff; border-bottom: 2px solid #007bff; padding-bottom: 10px; }
        
        #grid-monitores { 
            display: grid; 
            grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); 
            gap: 20px; 
            margin-top: 20px;
        }

        .monitor-card { 
            background: #111; 
            border: 2px solid #333; 
            border-radius: 10px; 
            padding: 10px; 
            transition: 0.3s;
        }

        .monitor-card img { 
            width: 100%; 
            border-radius: 5px; 
            background: #000;
            display: block;
        }

        .info {
            margin-top: 10px;
            font-size: 12px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .status-online { color: #00ff00; font-weight: bold; }
        .status-offline { color: #ff4444; font-weight: bold; }

        .badge {
            background: #007bff;
            padding: 2px 6px;
            border-radius: 4px;
            font-size: 10px;
        }
    </style>
</head>
<body>

    <h2>Câmeras Ativas em Tempo Real</h2>
    <div id="grid-monitores">Aguardando conexão dos dispositivos...</div>

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

        function iniciarMonitoramento() {
            const grid = document.getElementById('grid-monitores');

            // Escuta a coleção "ips_ativos" em tempo real
            db.collection("ips_ativos").onSnapshot((snapshot) => {
                if (snapshot.empty) {
                    grid.innerHTML = "Nenhum dispositivo conectado no momento.";
                    return;
                }

                grid.innerHTML = "";
                const agora = Date.now();

                snapshot.forEach((doc) => {
                    const dados = doc.data();
                    const segundosDesdeUltimoPing = (agora - dados.last_ping) / 1000;
                    const isOnline = segundosDesdeUltimoPing < 12; // 12 segundos de tolerância

                    const card = document.createElement('div');
                    card.className = "monitor-card";
                    card.style.borderColor = isOnline ? "#007bff" : "#444";

                    card.innerHTML = `
                        <img src="${dados.imagem}">
                        <div class="info">
                            <div>
                                <span class="badge">IP: ${dados.ip_original}</span>
                                <div style="margin-top:5px; font-size:10px; color:#aaa;">Visto: ${new Date(dados.last_ping).toLocaleTimeString()}</div>
                            </div>
                            <span class="${isOnline ? 'status-online' : 'status-offline'}">
                                ${isOnline ? '● AO VIVO' : '● DESCONECTADO'}
                            </span>
                        </div>
                    `;
                    grid.appendChild(card);
                });
            });
        }

        window.onload = iniciarMonitoramento;
    </script>
</body>
</html>
