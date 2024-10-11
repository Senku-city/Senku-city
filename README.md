<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Convert Images to PDF</title>
    <style>
        * {
            box-sizing: border-box;
        }

        body {
            font-family: 'Roboto', sans-serif; /* Use a Google Font similar to Android */
            background-color: #f5f5f5; /* Light background */
            color: #333; /* Dark text */
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            transition: background-color 0.3s, color 0.3s;
        }

        .container {
            max-width: 400px;
            width: 100%;
            background-color: white; /* White container */
            padding: 20px;
            border-radius: 16px;
            box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
            transition: background-color 0.3s;
        }

        h1 {
            text-align: center;
            color: #6200ee; /* Primary color */
            font-size: 24px;
            margin-bottom: 20px;
        }

        label {
            display: block;
            margin-top: 10px;
            color: #555;
            font-size: 16px;
        }

        input[type="file"],
        button {
            width: 100%;
            padding: 12px;
            margin-bottom: 15px;
            border: none;
            border-radius: 30px; /* Rounded button */
            background-color: #6200ee; /* Primary button color */
            color: white;
            font-size: 16px;
            cursor: pointer;
            transition: background-color 0.3s;
            font-weight: bold; /* Bold text */
        }

        input[type="file"] {
            background-color: #f0f0f0; /* Light background for file input */
            color: #333; /* Dark text for file input */
            border: 2px dashed #6200ee; /* Dashed border for file input */
            transition: border 0.3s;
        }

        input[type="file"]:hover {
            border: 2px solid #6200ee; /* Solid border on hover */
        }

        button:hover {
            background-color: #3700b3; /* Darker purple on hover */
        }

        #loading {
            display: none;
            text-align: center;
            color: #6200ee;
            font-weight: bold;
        }

        #message {
            margin-top: 15px;
            text-align: center;
            color: #d9534f; /* Error color */
        }

        .preview {
            display: flex;
            flex-direction: column;
            align-items: center;
            margin-bottom: 15px;
        }

        .preview img {
            max-width: 100%;
            max-height: 200px;
            margin-bottom: 10px;
            border-radius: 10px; /* Rounded image corners */
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
        }

        #progress {
            width: 100%;
            background-color: #ddd;
            border-radius: 30px; /* Rounded progress bar */
            overflow: hidden;
            height: 20px; /* Set height for progress bar */
            margin-bottom: 15px;
        }

        #progressBar {
            height: 100%;
            background-color: #6200ee;
            width: 0;
            transition: width 0.3s;
        }

        .dark-mode {
            background-color: #121212; /* Dark background */
            color: #f0f0f0; /* Light text */
        }

        .dark-mode .container {
            background-color: #1e1e1e; /* Dark container */
        }

        .toggle {
            display: flex;
            justify-content: space-between;
            margin-bottom: 15px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Convert Images to PDF</h1>
        <div class="toggle">
            <label for="modeSwitch">Dark Mode:</label>
            <input type="checkbox" id="modeSwitch">
        </div>
        <form id="pdfForm">
            <label for="imageInput">Select Images:</label>
            <input type="file" id="imageInput" multiple accept="image/*" required>
            <button type="submit">Convert to PDF</button>
        </form>
        <div id="loading">Creating PDF, please wait...</div>
        <div id="progress">
            <div id="progressBar"></div>
        </div>
        <div id="message"></div>
        <div id="preview" class="preview"></div>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf-lib/1.17.1/pdf-lib.min.js"></script>
    <script>
        // Toggle Dark Mode
        const modeSwitch = document.getElementById('modeSwitch');
        modeSwitch.addEventListener('change', () => {
            document.body.classList.toggle('dark-mode');
        });

        document.getElementById('pdfForm').addEventListener('submit', async function (e) {
            e.preventDefault();

            const files = document.getElementById('imageInput').files;
            const messageElement = document.getElementById('message');
            const loadingElement = document.getElementById('loading');
            const previewElement = document.getElementById('preview');
            const progressBar = document.getElementById('progressBar');

            if (files.length === 0) {
                messageElement.textContent = 'Please select at least one image.';
                return;
            }

            loadingElement.style.display = 'block';
            messageElement.textContent = '';
            previewElement.innerHTML = ''; // Clear previous previews
            const pdfDoc = await PDFLib.PDFDocument.create();

            for (let i = 0; i < files.length; i++) {
                const file = files[i];
                const imgBytes = await readFileAsArrayBuffer(file);
                let img;

                if (file.type === 'image/jpeg') {
                    img = await pdfDoc.embedJpg(imgBytes);
                } else if (file.type === 'image/png') {
                    img = await pdfDoc.embedPng(imgBytes);
                } else {
                    messageElement.textContent = 'Unsupported file type: ' + file.type;
                    loadingElement.style.display = 'none';
                    return;
                }

                const page = pdfDoc.addPage([img.width, img.height]);
                page.drawImage(img, {
                    x: 0,
                    y: 0,
                    width: img.width,
                    height: img.height,
                });

                // Preview the selected image
                const imgElement = document.createElement('img');
                imgElement.src = URL.createObjectURL(file);
                previewElement.appendChild(imgElement);

                // Update progress bar
                const progress = ((i + 1) / files.length) * 100;
                progressBar.style.width = progress + '%';
            }

            const pdfBytes = await pdfDoc.save();
            download(pdfBytes, 'converted.pdf', 'application/pdf');

            loadingElement.style.display = 'none';
            messageElement.textContent = 'PDF created successfully!';
            messageElement.style.color = 'green';
        });

        function readFileAsArrayBuffer(file) {
            return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.onload = () => resolve(reader.result);
                reader.onerror = reject;
                reader.readAsArrayBuffer(file);
            });
        }

        function download(data, filename, mime) {
            const blob = new Blob([data], { type: mime });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = filename;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
        }
    </script>
</body>
</html> 
