# verificador-qr
Sistema de verificación vehicular con generación de PDF, QR y conexión a Telegram
from flask import Flask, request, send_file
import qrcode
import pdfkit
import os
from datetime import datetime
import telegram

app = Flask(__name__)

# Token y chat ID de Telegram
TOKEN = "7281333656:AAGPxPdGcA82iRhen0p3yhOMIeA0pwdKrG0"
CHAT_ID = "8101486173"
bot = telegram.Bot(token=TOKEN)

@app.route('/generar', methods=['POST'])
def generar_documento():
    data = request.json
    nombre = data.get('nombre')
    curp = data.get('curp')
    folio = data.get('folio')
    estatus = data.get('estatus')
    
    if not all([nombre, curp, folio, estatus]):
        return {"error": "Faltan campos obligatorios"}, 400

    # Generar QR que apunta a una URL de verificación
    url_verificacion = f"https://tusistema.com/verificar?curp={curp}&folio={folio}"
    qr = qrcode.make(url_verificacion)
    qr_path = f"qr_{folio}.png"
    qr.save(qr_path)

    # Crear HTML para el PDF
    html = f"""
    <h1 style='text-align:center;'>Documento Oficial</h1>
    <p><b>Nombre:</b> {nombre}</p>
    <p><b>CURP:</b> {curp}</p>
    <p><b>Folio:</b> {folio}</p>
    <p><b>Estatus:</b> {estatus}</p>
    <p><b>Fecha:</b> {datetime.now().strftime('%Y-%m-%d')}</p>
    <img src='{qr_path}' width='200'/>
    """

    # Generar PDF
    pdf_path = f"{folio}.pdf"
    pdfkit.from_string(html, pdf_path)

    # Enviar notificación por Telegram
    bot.send_message(chat_id=CHAT_ID, text=f"📄 Documento generado: {folio}\nNombre: {nombre}")

    return send_file(pdf_path, as_attachment=True)

if __name__ == '__main__':
    app.run(debug=True)
    flask
qrcode
pdfkit
python-telegram-bot
{
  "nombre": "Juan Pérez",
  "curp": "JUAP900101HDFRRL09",
  "folio": "ABC123",
  "estatus": "Verificado"
}
