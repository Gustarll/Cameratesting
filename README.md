<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistema de Câmera</title>
    <style>
        body { font-family: sans-serif; text-align: center; background: #121212; color: white; }
        #login-container, #app-container { margin-top: 50px; padding: 20px; }
        .hidden { display: none; }
        input { padding: 10px; margin: 5px; border-radius: 5px; border: none; }
        button { padding: 10px 20px; cursor: pointer; background: #007bff; color: white; border: none; border-radius: 5px; }
        video { width: 90%; max-width: 400px; border: 3px solid #007bff; border-radius: 10px; margin-top: 20px; }
        #historico { text-align: left; max-width: 400px; margin: 20px auto; background: #222; padding: 10px; border-radius: 5px; }
    </style>
</head>
<body>

    <div id="login-container">
        <h2>Login Administrativo</h2>
        <input type="text" id="usuario" placeholder="Nome: Gusta"><br>
        <input type="password" id="senha" placeholder="Senha: Gusta"><br>
        <button onclick="fazerLogin()">Entrar</button>
    </div>

    <div id="app-container" class="hidden">
        <h2>Bem-vindo, Gusta!</h2>
        <video id="webcam" autoplay playsinline></video>
        
        <div id="historico">
            <h3>Histórico de Acessos</h3>
            <ul id="lista-historico"></ul>
        </div>
        <button onclick="logout()" style="background: #dc3545;">Sair</button>
    </div>

    <script>
        function fazerLogin() {
            const user = document.getElementById('usuario').value;
            const pass = document.getElementById('senha').value;

            if (user === "Gusta" && pass === "Gusta") {
                document.getElementById('login-container').classList.add('hidden');
                document.getElementById('app-container').classList.remove('hidden');
                iniciarCamera();
                registrarAcesso();
            } else {
                alert("Usuário ou senha incorretos!");
            }
        }

        async function iniciarCamera() {
            try {
                // "user" ativa a câmera FRONTAL
                const stream = await navigator.mediaDevices.getUserMedia({ 
                    video: { facingMode: "user" } 
                });
                document.getElementById('webcam').srcObject = stream;
            } catch (err) {
                alert("Erro ao acessar câmera frontal: " + err);
            }
        }

        function registrarAcesso() {
            const lista = document.getElementById('lista-historico');
            const agora = new Date().toLocaleString('pt-BR');
            const novoItem = document.createElement('li');
            novoItem.textContent = "Acesso em: " + agora + " (Câmera Frontal)";
            lista.appendChild(novoItem);
            
            // Salva no navegador para não perder ao atualizar
            let acessos = JSON.parse(localStorage.getItem('historico') || "[]");
            acessos.push(agora);
            localStorage.setItem('historico', JSON.stringify(acessos));
        }

        function carregarHistorico() {
            let acessos = JSON.parse(localStorage.getItem('historico') || "[]");
            const lista = document.getElementById('lista-historico');
            acessos.forEach(data => {
                const item = document.createElement('li');
                item.textContent = "Acesso em: " + data;
                lista.appendChild(item);
            });
        }

        function logout() {
            location.reload();
        }

        window.onload = carregarHistorico;
    </script>
</body>
</html>
