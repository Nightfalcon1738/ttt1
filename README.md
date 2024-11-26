# ttt1
import os
from flask import Flask, request, jsonify, send_from_directory
from PIL import Image

app = Flask(__name__)

# Directories for uploads and outputs
UPLOAD_FOLDER = 'uploads'
OUTPUT_FOLDER = 'outputs'
os.makedirs(UPLOAD_FOLDER, exist_ok=True)
os.makedirs(OUTPUT_FOLDER, exist_ok=True)

@app.route('/upload-logo', methods=['POST'])
def upload_logo():
    if 'file' not in request.files:
        return jsonify({'error': 'No file uploaded'}), 400
    file = request.files['file']
    if file.filename == '':
        return jsonify({'error': 'No file selected'}), 400
    file_path = os.path.join(UPLOAD_FOLDER, file.filename)
    file.save(file_path)
    return jsonify({'message': 'File uploaded successfully', 'file_path': file_path}), 200

@app.route('/generate-pattern', methods=['POST'])
def generate_pattern():
    data = request.json
    logo_path = data.get('logo_path')
    if not logo_path or not os.path.exists(logo_path):
        return jsonify({'error': 'Invalid logo path'}), 400

    # Load the logo and create the pattern
    logo = Image.open(logo_path).convert("RGBA")
    canvas_size = (1000, 1000)
    canvas = Image.new("RGBA", canvas_size, (255, 255, 255, 0))  # Transparent background
    logo_width, logo_height = logo.size
    spacing_x, spacing_y = 200, 200

    for y in range(0, canvas_size[1], logo_height + spacing_y):
        for x in range(0, canvas_size[0], logo_width + spacing_x):
            canvas.paste(logo, (x, y), mask=logo)

    output_path = os.path.join(OUTPUT_FOLDER, 'pattern.png')
    canvas.save(output_path)
    return jsonify({'message': 'Pattern generated successfully', 'pattern_path': '/outputs/pattern.png'}), 200

@app.route('/outputs/<filename>', methods=['GET'])
def get_output(filename):
    return send_from_directory(OUTPUT_FOLDER, filename)

if __name__ == '__main__':
    app.run(debug=True)



<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pattern Generator</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            padding: 20px;
        }
        .container {
            max-width: 600px;
            margin: 0 auto;
        }
        .preview {
            margin-top: 20px;
            text-align: center;
        }
        img {
            max-width: 100%;
        }
        .button {
            display: block;
            margin: 20px auto;
            padding: 10px 20px;
            background-color: #007bff;
            color: white;
            text-align: center;
            text-decoration: none;
            border-radius: 5px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Pattern Generator</h1>
        <form id="upload-form">
            <label for="logo">Upload Logo:</label>
            <input type="file" id="logo" name="logo" accept="image/*" required>
            <br><br>
            <label for="pattern-type">Select Pattern Type:</label>
            <select id="pattern-type">
                <option value="scatter">Scatter</option>
                <!-- Add more pattern options here -->
            </select>
            <br><br>
            <button type="submit" class="button">Generate Pattern</button>
        </form>
        <div class="preview">
            <h2>Preview:</h2>
            <img id="pattern-preview" src="" alt="Pattern Preview">
            <a id="download-link" href="#" class="button" style="display: none;">Download Pattern</a>
        </div>
    </div>
    <script>
        const form = document.getElementById('upload-form');
        const previewImg = document.getElementById('pattern-preview');
        const downloadLink = document.getElementById('download-link');

        form.addEventListener('submit', async (e) => {
            e.preventDefault();
            const formData = new FormData(form);

            // Upload the logo
            const uploadResponse = await fetch('/upload-logo', {
                method: 'POST',
                body: formData,
            });
            const uploadResult = await uploadResponse.json();

            if (!uploadResponse.ok) {
                alert('Error uploading logo: ' + uploadResult.error);
                return;
            }

            const logoPath = uploadResult.file_path;

            // Generate the pattern
            const patternType = document.getElementById('pattern-type').value;
            const patternResponse = await fetch('/generate-pattern', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ logo_path: logoPath, pattern_type: patternType }),
            });
            const patternResult = await patternResponse.json();

            if (!patternResponse.ok) {
                alert('Error generating pattern: ' + patternResult.error);
                return;
            }

            const patternPath = patternResult.pattern_path;

            // Update the preview and download link
            previewImg.src = patternPath;
            downloadLink.href = patternPath;
            downloadLink.style.display = 'block';
        });
    </script>
</body>
</html>


Flask==2.1.1
Pillow==9.4.0

