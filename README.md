
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gerador de Links Seguros do WhatsApp</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f0f2f5;
        }
        .container {
            background-color: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
        textarea, input, select {
            width: 100%;
            margin-bottom: 10px;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 5px;
            box-sizing: border-box;
        }
        button {
            padding: 10px 15px;
            background-color: #25D366;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            margin-right: 5px;
            transition: background-color 0.3s;
        }
        button:hover {
            background-color: #128C7E;
        }
        #output, #savedLinks {
            margin-top: 20px;
            border: 1px solid #ddd;
            padding: 15px;
            border-radius: 5px;
        }
        .number-input {
            display: flex;
            margin-bottom: 15px;
            gap: 10px;
        }
        .link-item {
            padding: 10px;
            border-bottom: 1px solid #eee;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .link-item a {
            color: #128C7E;
            text-decoration: none;
            font-weight: bold;
        }
        .link-item a:hover {
            text-decoration: underline;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Gerador de Links Seguros do WhatsApp</h1>
        <div id="inputContainer">
            <div class="number-input">
                <input type="text" placeholder="Número de telefone" class="phone">
                <input type="text" placeholder="Nome (opcional)" class="name">
            </div>
        </div>
        <button onclick="addInput()">Adicionar mais um número</button>
        <input type="text" id="prefix" placeholder="Prefixo para os nomes (opcional)">
        <button onclick="transformNumbers()">Gerar Links</button>
        <button onclick="saveLinks()">Salvar Links</button>
        <div id="output"></div>
        <h2>Links Salvos</h2>
        <div id="savedLinks"></div>
    </div>

    <script>
        let linkData = [];

        function generateUniqueId() {
            return 'w' + Math.random().toString(36).substr(2, 9);
        }

        function addInput() {
            const container = document.getElementById('inputContainer');
            const div = document.createElement('div');
            div.className = 'number-input';
            div.innerHTML = `
                <input type="text" placeholder="Número de telefone" class="phone">
                <input type="text" placeholder="Nome (opcional)" class="name">
            `;
            container.appendChild(div);
        }

        function transformNumbers() {
            const inputs = document.querySelectorAll('.number-input');
            const prefix = document.getElementById('prefix').value;
            let output = '';
            linkData = [];

            const defaultNames = [
                'João', 'Maria', 'Pedro', 'Ana', 'Carlos', 'Mariana', 'José', 'Beatriz',
                'Antônio', 'Larissa', 'Francisco', 'Juliana', 'Paulo', 'Fernanda'
            ];

            inputs.forEach((input, index) => {
                const number = input.querySelector('.phone').value.trim();
                let name = input.querySelector('.name').value.trim();

                if (number !== '') {
                    if (name === '') {
                        name = defaultNames[Math.floor(Math.random() * defaultNames.length)];
                    }
                    const displayName = prefix ? `${prefix} ${name}` : name;
                    const uniqueId = generateUniqueId();
                    const secureLink = `${window.location.origin}/redirect/${uniqueId}`;
                    
                    output += `
                        <div class="link-item">
                            <a href="${secureLink}" target="_blank">${displayName}</a>
                            <div>
                                <button onclick="editLink(${index})">Editar</button>
                                <button onclick="shareLink('${secureLink}', '${displayName}')">Compartilhar</button>
                            </div>
                        </div>`;
                    
                    linkData.push({
                        id: uniqueId,
                        number,
                        name: displayName,
                        link: secureLink
                    });
                }
            });

            document.getElementById('output').innerHTML = output;
            saveToLocalStorage();
        }

        function saveToLocalStorage() {
            localStorage.setItem('whatsappRedirects', JSON.stringify(linkData));
        }

        function shareLink(link, name) {
            if (navigator.share) {
                navigator.share({
                    title: 'Contato WhatsApp',
                    text: `Entre em contato com ${name}`,
                    url: link
                }).catch((error) => console.log('Erro ao compartilhar', error));
            } else {
                const tempInput = document.createElement('input');
                document.body.appendChild(tempInput);
                tempInput.value = link;
                tempInput.select();
                document.execCommand('copy');
                document.body.removeChild(tempInput);
                alert('Link copiado para a área de transferência!');
            }
        }

        function saveLinks() {
            saveToLocalStorage();
            alert('Links salvos com sucesso!');
            displaySavedLinks();
        }

        function displaySavedLinks() {
            const savedLinks = JSON.parse(localStorage.getItem('whatsappRedirects')) || [];
            const savedLinksDiv = document.getElementById('savedLinks');
            let output = '';

            savedLinks.forEach((link, index) => {
                output += `
                    <div class="link-item">
                        <a href="${link.link}" target="_blank">${link.name}</a>
                        <button onclick="deleteSavedLink(${index})">Excluir</button>
                    </div>`;
            });

            savedLinksDiv.innerHTML = output;
        }

        function deleteSavedLink(index) {
            linkData.splice(index, 1);
            saveToLocalStorage();
            displaySavedLinks();
        }

        // Inicialização
        addInput();
        displaySavedLinks();

        // Tratamento de redirecionamento
        if (window.location.pathname.startsWith('/redirect/')) {
            const id = window.location.pathname.split('/').pop();
            const redirects = JSON.parse(localStorage.getItem('whatsappRedirects')) || [];
            const redirectData = redirects.find(r => r.id === id);
            
            if (redirectData) {
                window.location.href = `https://api.whatsapp.com/send?phone=${redirectData.number.replace(/\D/g, '')}`;
            }
        }
    </script>
</body>
</html>
