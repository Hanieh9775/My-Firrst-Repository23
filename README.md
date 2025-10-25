import qrcode
import os
from flask import Flask, request, render_template_string, send_from_directory

app = Flask(__name__)
app.config["QR_FOLDER"] = "qrcodes"
os.makedirs(app.config["QR_FOLDER"], exist_ok=True)

HTML_TEMPLATE = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>QR Code Generator</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        body { background: #111; color: #fff; height: 100vh; display: flex; justify-content: center; align-items: center; }
        .card { background: #222; border-radius: 20px; padding: 30px; text-align: center; width: 500px; }
        input { border-radius: 10px; }
        img { margin-top: 20px; width: 200px; border-radius: 10px; }
        .btn-custom { background: #00ccff; color: #000; font-weight: bold; }
        .btn-custom:hover { background: #00a3cc; }
    </style>
</head>
<body>
    <div class="card">
        <h2>🔗 QR Code Generator</h2>
        <form method="POST">
            <input name="data" type="text" class="form-control mt-3" placeholder="Enter any text or URL..." required>
            <button class="btn btn-custom w-100 mt-3">Generate</button>
        </form>

        {% if qr_filename %}
            <img src="/qrcodes/{{ qr_filename }}" alt="QR Code">
            <a class="btn btn-outline-light w-100 mt-3" href="/qrcodes/{{ qr_filename }}" download>Download QR Code</a>
        {% endif %}
    </div>
</body>
</html>
"""

@app.route("/", methods=["GET", "POST"])
def index():
    qr_filename = None
    if request.method == "POST":
        data = request.form["data"]
        qr_filename = f"qr_{abs(hash(data))}.png"
        filepath = os.path.join(app.config["QR_FOLDER"], qr_filename)

        qr = qrcode.QRCode(
            version=1,
            error_correction=qrcode.constants.ERROR_CORRECT_H,
            box_size=12,
            border=3,
        )
        qr.add_data(data)
        qr.make(fit=True)
        img = qr.make_image(fill_color="cyan", back_color="black")
        img.save(filepath)

    return render_template_string(HTML_TEMPLATE, qr_filename=qr_filename)

@app.route("/qrcodes/<path:filename>")
def serve_qr(filename):
    return send_from_directory(app.config["QR_FOLDER"], filename)

if __name__ == "__main__":
    app.run(debug=True)
