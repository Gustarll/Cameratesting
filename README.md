<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>CENTRAL DE MONITORAMENTO HD</title>
    <style>
        body { background: #0a0a0a; color: #e0e0e0; font-family: 'Segoe UI', sans-serif; margin: 0; padding: 20px; }
        .header { display: flex; justify-content: space-between; align-items: center; border-bottom: 2px solid #333; padding-bottom: 10px; margin-bottom: 20px; }
        .grid { display: grid; grid-template-columns: 1fr; gap: 30px; }
        
        .device-card { background: #161616; border-radius: 12px; padding: 20px; border: 1px solid #333; box-shadow: 0 10px 30px rgba(0,0,0,0.5); }
        .device-info { margin-bottom: 15px; display: flex; align-items: center; gap: 15px; }
        .status-dot { width: 12px; height: 12px; border-radius: 50%; background: #ff4444; }
        .online .status-dot { background: #00ff00; box-shadow: 0 0 10px #00ff00; }

        .feeds { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; }
        .feed-container { position: relative; background: #000; border-radius: 8px; overflow: hidden; border: 1px solid #444; }
        .feed-container img { width: 100%; display: block; min-height: 200px; }
        .label { position: absolute; top: 10px; left: 10px; background: rgba(0,0,0,0.7); padding: 5px 10px; border-radius: 4px; font-size: 12px; font-weight: bold; }
    </style>
</head>
<body>

    <div class="header">
        <h1>MONITORAMENTO EM TEMPO REAL</h1>
        <div id="count">Dispositivos: 0</div>
    </div>

    <div id="dispositivos" class="grid">
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

        db.collection("ips_ativos").onSnapshot((snapshot) => {
            const container = document.getElementById('dispositivos');
            document.getElementById('count').innerText = `Dispositivos: ${snapshot.size}`;
            container.innerHTML = "";

            snapshot.forEach((doc) => {
                const d = doc.data();
                const isOnline = (Date.now() - d.last_ping) < 15000;
                
                const card = document.createElement('div');
                card.className = `device-card ${isOnline ? 'online' : ''}`;
                
                card.innerHTML = `
                    <div class="device-info">
                        <div class="status-dot"></div>
                        <strong>ID: ${d.nome}</strong>
                        <span style="font-size: 12px; color: #888;">Visto em: ${new Date(d.last_ping).toLocaleTimeString()}</span>
                    </div>
                    <div class="feeds">
                        <div class="feed-container">
                            <div class="label">CÂMERA</div>
                            <img src="${d.camera || ''}" onerror="this.src='https://via.placeholder.com/640x480?text=Sem+Sinal+Camera'">
                        </div>
                        <div class="feed-container">
                            <div class="label">TELA</div>
                            <img src="${d.tela || ''}" onerror="this.src='https://via.placeholder.com/640x480?text=Sem+Sinal+Tela'">
                        </div>
                    </div>
                `;
                container.appendChild(card);
            });
        });
    </script>
</body>
</html>
