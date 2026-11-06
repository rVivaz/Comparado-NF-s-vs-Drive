# Comparado-NF-s-vs-Drive
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Leitor de NFs + Upload para o Drive</title>
    <!-- Biblioteca Tesseract.js v5 -->
    <script src="https://cdn.jsdelivr.net/npm/tesseract.js@5/dist/tesseract.min.js"></script>
    <style>
        * {
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        body {
            background-color: #f0f2f5;
            margin: 0;
            padding: 20px;
            color: #333;
        }
        .container {
            max-width: 900px;
            margin: 0 auto;
            background: #ffffff;
            border-radius: 12px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.08);
            padding: 30px;
        }
        h1 {
            color: #1a73e8;
            margin-top: 0;
            font-size: 24px;
            text-align: center;
        }
        p.subtitle {
            text-align: center;
            color: #666;
            margin-bottom: 25px;
            font-size: 14px;
        }
        .drop-zone {
            border: 2px dashed #1a73e8;
            border-radius: 10px;
            padding: 30px 20px;
            text-align: center;
            background-color: #f8fafe;
            cursor: pointer;
            transition: all 0.2s ease;
        }
        .drop-zone:hover, .drop-zone.dragover {
            background-color: #e8f0fe;
            border-color: #1557b0;
        }
        .drop-zone input {
            display: none;
        }
        .drop-icon {
            font-size: 36px;
            color: #1a73e8;
            margin-bottom: 8px;
        }
        .status-container {
            margin: 20px 0;
            display: none;
        }
        .status-text {
            font-size: 14px;
            color: #555;
            display: flex;
            justify-content: space-between;
            margin-bottom: 6px;
        }
        .progress-bar-bg {
            background: #e0e0e0;
            border-radius: 10px;
            height: 10px;
            overflow: hidden;
        }
        .progress-bar {
            background: #1a73e8;
            height: 100%;
            width: 0%;
            transition: width 0.2s;
        }
        .drive-status {
            font-size: 13px;
            margin-top: 6px;
            font-weight: 600;
            color: #d97706;
        }
        .content-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
            margin-top: 20px;
        }
        @media (max-width: 768px) {
            .content-grid { grid-template-columns: 1fr; }
        }
        .card {
            background: #fafafa;
            border: 1px solid #e0e0e0;
            border-radius: 8px;
            padding: 15px;
        }
        .card h3 {
            margin-top: 0;
            font-size: 16px;
            color: #333;
            border-bottom: 1px solid #ddd;
            padding-bottom: 8px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        img#preview {
            max-width: 100%;
            max-height: 280px;
            object-fit: contain;
            border-radius: 6px;
            display: block;
            margin: 0 auto;
        }
        textarea {
            width: 100%;
            height: 160px;
            border: 1px solid #ccc;
            border-radius: 6px;
            padding: 10px;
            font-size: 14px;
            resize: vertical;
            font-family: monospace;
            background-color: #fff;
        }
        .format-selector {
            margin: 12px 0;
            display: flex;
            flex-direction: column;
            gap: 6px;
        }
        .format-selector label {
            font-size: 13px;
            color: #444;
            cursor: pointer;
            display: flex;
            align-items: center;
            gap: 6px;
        }
        button {
            background-color: #34a853;
            color: white;
            border: none;
            padding: 10px 16px;
            border-radius: 6px;
            font-size: 14px;
            font-weight: 600;
            cursor: pointer;
            width: 100%;
            margin-top: 10px;
        }
        button:hover { background-color: #2d8e47; }
        .badge {
            background-color: #e8f0fe;
            color: #1a73e8;
            padding: 2px 8px;
            border-radius: 12px;
            font-size: 12px;
            font-weight: bold;
        }
    </style>
</head>
<body>

    <div class="container">
        <h1>📄 Leitor de NFs + Backup no Google Drive</h1>
        <p class="subtitle">Envie a imagem para extrair os números e salvar uma cópia na sua pasta do Drive</p>

        <!-- Área de Upload -->
        <div class="drop-zone" id="dropZone" onclick="document.getElementById('fileInput').click()">
            <div class="drop-icon">📁</div>
            <p>Clique para selecionar ou arraste a imagem aqui</p>
            <span>Dica: Você também pode colar com <b>Ctrl + V</b></span>
            <input type="file" id="fileInput" accept="image/*">
        </div>

        <!-- Barra de Progresso -->
        <div class="status-container" id="statusContainer">
            <div class="status-text">
                <span id="statusLabel">Iniciando...</span>
                <span id="statusPercent">0%</span>
            </div>
            <div class="progress-bar-bg">
                <div class="progress-bar" id="progressBar"></div>
            </div>
            <div class="drive-status" id="driveStatus">☁️ Aguardando envio para o Drive...</div>
        </div>

        <!-- Conteúdo -->
        <div class="content-grid">
            <div class="card">
                <h3>Imagem Selecionada</h3>
                <img id="preview" src="" alt="Imagem enviada" style="display: none;">
                <p id="noImageText" style="color: #888; text-align: center; margin-top: 50px;">Nenhuma imagem carregada</p>
            </div>

            <div class="card">
                <h3>
                    Resultados 
                    <span class="badge" id="countBadge">0 NFs</span>
                </h3>

                <div class="format-selector">
                    <label>
                        <input type="radio" name="formatOption" value="prefix" checked onchange="atualizarFormato()">
                        Com "NF" e vírgula (Ex: NF 13988, 13990...)
                    </label>
                    <label>
                        <input type="radio" name="formatOption" value="slash" onchange="atualizarFormato()">
                        Com "NF" e barra (Ex: NF 11245/ 11524/ 15423...)
                    </label>
                    <label>
                        <input type="radio" name="formatOption" value="comma" onchange="atualizarFormato()">
                        Apenas números com vírgula (Ex: 13988, 13990...)
                    </label>
                    <label>
                        <input type="radio" name="formatOption" value="lines" onchange="atualizarFormato()">
                        Uma por linha (Ideal para colar na coluna do Excel)
                    </label>
                </div>

                <textarea id="resultado" placeholder="Os números das NFs aparecerão aqui..." readonly></textarea>

                <button onclick="copiarResultado(event)">📋 Copiar para Planilha</button>
            </div>
        </div>
    </div>

    <canvas id="prepCanvas" style="display:none;"></canvas>

    <script>
        const APPS_SCRIPT_URL = "https://drive.google.com/drive/folders/1nQeLCb6Pyn3pdeqAJMk3oF2IQgp_7KU7";

        const fileInput = document.getElementById('fileInput');
        const dropZone = document.getElementById('dropZone');
        const preview = document.getElementById('preview');
        const noImageText = document.getElementById('noImageText');
        const statusContainer = document.getElementById('statusContainer');
        const statusLabel = document.getElementById('statusLabel');
        const statusPercent = document.getElementById('statusPercent');
        const progressBar = document.getElementById('progressBar');
        const driveStatus = document.getElementById('driveStatus');
        const resultadoArea = document.getElementById('resultado');
        const countBadge = document.getElementById('countBadge');
        const prepCanvas = document.getElementById('prepCanvas');

        let listaNFsExtraidas = [];

        fileInput.addEventListener('change', (e) => {
            if (e.target.files.length > 0) processarImagem(e.target.files[0]);
        });

        dropZone.addEventListener('dragover', (e) => {
            e.preventDefault();
            dropZone.classList.add('dragover');
        });
        dropZone.addEventListener('dragleave', () => dropZone.classList.remove('dragover'));
        dropZone.addEventListener('drop', (e) => {
            e.preventDefault();
            dropZone.classList.remove('dragover');
            if (e.dataTransfer.files.length > 0) processarImagem(e.dataTransfer.files[0]);
        });

        window.addEventListener('paste', (e) => {
            const items = e.clipboardData.items;
            for (let i = 0; i < items.length; i++) {
                if (items[i].type.indexOf('image') !== -1) {
                    processarImagem(items[i].getAsFile());
                    break;
                }
            }
        });

        async function processarImagem(file) {
            const imgUrl = URL.createObjectURL(file);
            preview.src = imgUrl;
            preview.style.display = 'block';
            noImageText.style.display = 'none';

            statusContainer.style.display = 'block';
            progressBar.style.width = '0%';
            statusPercent.innerText = '0%';
            statusLabel.innerText = 'Otimizando contraste da imagem...';
            driveStatus.innerText = '☁️ Enviando imagem para o Google Drive...';
            driveStatus.style.color = '#d97706';
            resultadoArea.value = '';
            listaNFsExtraidas = [];

            fazerUploadParaDrive(file);

            const imgOtimizada = await otimizarImagem(imgUrl);

            try {
                const worker = await Tesseract.createWorker('por+eng', 1, {
                    logger: m => {
                        if (m.status === 'recognizing text') {
                            const pct = Math.round((m.progress || 0) * 100);
                            progressBar.style.width = pct + '%';
                            statusPercent.innerText = pct + '%';
                            statusLabel.innerText = 'Lendo documentos e tabelas...';
                        }
                    }
                });

                // PSM 11: Varredura de texto esparso/tabelas em múltiplas colunas
                await worker.setParameters({
                    tessedit_pageseg_mode: '11',
                    tessedit_char_whitelist: '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz-/. '
                });

                const ret = await worker.recognize(imgOtimizada);
                await worker.terminate();

                extrairNFs(ret.data.text);
                statusLabel.innerText = 'Leitura OCR concluída!';

            } catch (erro) {
                console.error(erro);
                statusLabel.innerText = 'Erro no leitor OCR.';
            }
        }

        async function fazerUploadParaDrive(file) {
            if (APPS_SCRIPT_URL.includes("SUA_URL_AQUI")) {
                driveStatus.innerText = '⚠️ URL do Google Apps Script não foi configurada no código HTML.';
                driveStatus.style.color = '#dc3545';
                return;
            }

            try {
                const base64 = await converterParaBase64(file);
                const payload = {
                    fileName: `NF_${Date.now()}_${file.name || 'foto.jpg'}`,
                    mimeType: file.type || 'image/jpeg',
                    base64: base64
                };

                const response = await fetch(APPS_SCRIPT_URL, {
                    method: 'POST',
                    headers: { 'Content-Type': 'text/plain;charset=utf-8' },
                    body: JSON.stringify(payload)
                });

                const result = await response.json();

                if (result.status === 'success') {
                    driveStatus.innerText = '✅ Salvo no Google Drive com sucesso!';
                    driveStatus.style.color = '#1e7e34';
                } else {
                    throw new Error(result.message);
                }
            } catch (err) {
                console.error("Erro no Upload para o Drive:", err);
                driveStatus.innerText = '❌ Erro ao enviar para o Drive.';
                driveStatus.style.color = '#dc3545';
            }
        }

        function converterParaBase64(file) {
            return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.onload = () => resolve(reader.result);
                reader.onerror = error => reject(error);
                reader.readAsDataURL(file);
            });
        }

        function otimizarImagem(src) {
            return new Promise((resolve) => {
                const img = new Image();
                img.onload = () => {
                    const ctx = prepCanvas.getContext('2d');
                    const scale = 2.0;

                    prepCanvas.width = img.width * scale;
                    prepCanvas.height = img.height * scale;

                    ctx.imageSmoothingEnabled = true;
                    ctx.imageSmoothingQuality = 'high';
                    ctx.drawImage(img, 0, 0, prepCanvas.width, prepCanvas.height);

                    const imgData = ctx.getImageData(0, 0, prepCanvas.width, prepCanvas.height);
                    const d = imgData.data;

                    for (let i = 0; i < d.length; i += 4) {
                        let gray = 0.299 * d[i] + 0.587 * d[i + 1] + 0.114 * d[i + 2];
                        gray = (gray - 128) * 1.5 + 128;
                        gray = Math.min(255, Math.max(0, gray));

                        d[i] = gray;
                        d[i + 1] = gray;
                        d[i + 2] = gray;
                    }

                    ctx.putImageData(imgData, 0, 0);
                    resolve(prepCanvas.toDataURL('image/png'));
                };
                img.src = src;
            });
        }

        function extrairNFs(textoBruto) {
            console.log("=== TEXTO EXTRAÍDO PELO OCR ===");
            console.log(textoBruto);

            let textoTratado = textoBruto
                .replace(/[—–_]/g, '-')
                .replace(/[oOQDbU]/g, '0');

            let nfsEncontradas = new Set();
            let match;

            // REGEX: Exige estritamente 4 zeros (0000) no início + número da NF + sufixo -5 (ou -S do OCR)
            const regexNf = /(?:^|[^\d])0{4}([1-9]\d{4,5})\s*-\s*[\dsS]/gi;

            while ((match = regexNf.exec(textoTratado)) !== null) {
                nfsEncontradas.add(match[1]);
            }

            listaNFsExtraidas = Array.from(nfsEncontradas);
            countBadge.innerText = `${listaNFsExtraidas.length} NFs`;
            atualizarFormato();
        }

        function atualizarFormato() {
            if (listaNFsExtraidas.length === 0) {
                resultadoArea.value = 'Nenhum número de NF identificado no padrão desejado.';
                return;
            }

            const opcao = document.querySelector('input[name="formatOption"]:checked').value;

            if (opcao === 'prefix') {
                resultadoArea.value = 'NF ' + listaNFsExtraidas.join(', ');
            } else if (opcao === 'slash') {
                resultadoArea.value = 'NF ' + listaNFsExtraidas.join('/ ');
            } else if (opcao === 'comma') {
                resultadoArea.value = listaNFsExtraidas.join(', ');
            } else if (opcao === 'lines') {
                resultadoArea.value = listaNFsExtraidas.join('\n');
            }
        }

        function copiarResultado(evt) {
            if (!resultadoArea.value || listaNFsExtraidas.length === 0) {
                alert('Não há números para copiar.');
                return;
            }

            navigator.clipboard.writeText(resultadoArea.value).then(() => {
                const btn = evt ? evt.target : document.querySelector('button');
                const textoOriginal = btn.innerText;
                btn.innerText = '✅ Copiado!';

                setTimeout(() => {
                    btn.innerText = textoOriginal;
                }, 2000);
            });
        }
    </script>
</body>
</html>
