<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Acessar Câmera</title>
    <style>
        video { width: 100%; max-width: 500px; border: 2px solid #333; }
    </style>
</head>
<body>
    <h2>Acesso à Câmera do Smartphone</h2>
    <video id="webcam" autoplay playsinline></video>

    <script>
        async function iniciarCamera() {
            try {
                // Solicita acesso apenas ao vídeo, preferindo a câmera traseira ('environment')
                const stream = await navigator.mediaDevices.getUserMedia({ 
                    video: { facingMode: "environment" } 
                });
                
                const videoElement = document.getElementById('webcam');
                videoElement.srcObject = stream;
            } catch (err) {
                console.error("Erro ao acessar a câmera: ", err);
                alert("Permissão negada ou erro no dispositivo.");
            }
        }

        // Inicia a função ao carregar a página
        window.onload = iniciarCamera;
    </script>
</body>
</html>
