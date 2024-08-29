<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gerador de Links do WhatsApp</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        textarea, input, select {
            width: 100%;
            margin-bottom: 10px;
            padding: 5px;
        }
        button {
            padding: 10px;
            background-color: #4CAF50;
            color: white;
            border: none;
            cursor: pointer;
            margin-right: 5px;
        }
        #output, #savedLinks {
            margin-top: 20px;
            border: 1px solid #ddd;
            padding: 10px;
        }
        .number-input {
            display: flex;
            margin-bottom: 10px;
        }
        .number-input input {
            flex-grow: 1;
            margin-right: 10px;
        }
        .edit-mode input {
            border: 2px solid #4CAF50;
        }
    </style>
</head>
<body>
    <h1>Gerador de Links do WhatsApp</h1>
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

    <script>
        let linkData = [];

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

        function generateShortId() {
            const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
            let result = '';
            for (let i = 0; i < 5; i++) {
                result += characters.charAt(Math.floor(Math.random() * characters.length));
            }
            return result;
        }

        function transformNumbers() {
            const inputs = document.querySelectorAll('.number-input');
            const prefix = document.getElementById('prefix').value;
            let output = '';
            linkData = [];

            const defaultNames = [
                'João', 'Maria', 'Pedro', 'Ana', 'Carlos', 'Mariana', 'José', 'Beatriz',
                'Antônio', 'Larissa', 'Francisco', 'Juliana', 'Paulo', 'Fernanda', 'Lucas',
                'Patrícia', 'Luiz', 'Camila', 'Marcelo', 'Amanda'
            ];

            inputs.forEach((input, index) => {
                const number = input.querySelector('.phone').value.trim();
                let name = input.querySelector('.name').value.trim();

                if (number !== '') {
                    if (name === '') {
                        name = defaultNames[Math.floor(Math.random() * defaultNames.length)];
                    }
                    const displayName = prefix ? `${prefix} ${name}` : name;
                    const shortId = generateShortId();
                    const redirectLink = `${window.location.origin}/w/${shortId}`;
                    output += `
                        <div>
                            <a href="${redirectLink}" target="_blank">${displayName}</a>
                            <button onclick="editLink(${index})">Editar</button>
                            <button onclick="shareLink('${redirectLink}', '${displayName}')">Compartilhar</button>
                        </div>`;
                    linkData.push({ id: shortId, number, name: displayName, link: redirectLink });
                }
            });

            document.getElementById('output').innerHTML = output;
        }

        function editLink(index) {
            const linkDiv = document.getElementById('output').children[index];
            const link = linkData[index];
            linkDiv.innerHTML = `
                <div class="edit-mode">
                    <input type="text" value="${link.number}" class="phone">
                    <input type="text" value="${link.name}" class="name">
                    <button onclick="saveEdit(${index})">Salvar</button>
                </div>
            `;
        }

        function saveEdit(index) {
            const editDiv = document.getElementById('output').children[index].querySelector('.edit-mode');
            const number = editDiv.querySelector('.phone').value.trim();
            const name = editDiv.querySelector('.name').value.trim();
            linkData[index].number = number;
            linkData[index].name = name;
            transformNumbers(); // Atualiza a exibição
        }

        function shareLink(link, name) {
            if (navigator.share) {
                navigator.share({
                    title: 'Link do WhatsApp',
                    text: `Entre em contato com ${name} no WhatsApp`,
                    url: link
                }).then(() => console.log('Compartilhado com sucesso'))
                  .catch((error) => console.log('Erro ao compartilhar', error));
            } else {
                alert('Seu navegador não suporta a funcionalidade de compartilhamento. Copie o link manualmente: ' + link);
            }
        }

        function saveLinks() {
            localStorage.setItem('whatsappLinks', JSON.stringify(linkData));
            alert('Links salvos com sucesso!');
            displaySavedLinks();
        }

        function displaySavedLinks() {
            const savedLinks = JSON.parse(localStorage.getItem('whatsappLinks')) || [];
            const savedLinksDiv = document.getElementById('savedLinks');
            let output = '';

            savedLinks.forEach((link, index) => {
                output += `
                    <div>
                        <a href="${link.link}" target="_blank">${link.name}</a>
                        <button onclick="deleteSavedLink(${index})">Excluir</button>
                    </div>`;
            });

            savedLinksDiv.innerHTML = output;
        }

        function deleteSavedLink(index) {
            const savedLinks = JSON.parse(localStorage.getItem('whatsappLinks')) || [];
            savedLinks.splice(index, 1);
            localStorage.setItem('whatsappLinks', JSON.stringify(savedLinks));
            displaySavedLinks();
        }

        function handleRedirect() {
            const path = window.location.pathname;
            const shortId = path.split('/w/')[1];
            if (shortId) {
                const link = linkData.find(item => item.id === shortId);
                if (link) {
                    window.location.href = `https://api.whatsapp.com/send?phone=${link.number.replace(/\D/g, '')}`;
                }
            }
        }

        // Inicialização
        addInput();
        displaySavedLinks();
        window.onload = handleRedirect;
    </script>
</body>
</html> 
