import smtplib
from iqoptionapi.stable_api import IQ_Option
import time

# Credenciales de tu cuenta de IQ Option
username = "tu_email"
password = "tu_contraseña"

# Conectar a la API de IQ Option
iq = IQ_Option(username, password)
iq.connect()

# Comprobar conexión
if iq.check_connect():
    print("Conexión exitosa!")
else:
    print("Conexión fallida, verifica tus credenciales.")

# Obtener datos de un activo, por ejemplo EUR/USD
def get_market_data():
    iq.start_candles_stream("EURUSD", 60, 10)
    candles = iq.get_realtime_candles("EURUSD", 60)
    prices = [candle['close'] for candle in candles.values()]
    return prices

# Calcular RSI (simplificado)
def calculate_rsi(prices, period=14):
    gains = []
    losses = []
    for i in range(1, len(prices)):
        change = prices[i] - prices[i - 1]
        if change > 0:
            gains.append(change)
        else:
            losses.append(abs(change))

    avg_gain = sum(gains[-period:]) / period
    avg_loss = sum(losses[-period:]) / period
    rs = avg_gain / avg_loss
    rsi = 100 - (100 / (1 + rs))
    return rsi

# Enviar señal por correo electrónico
def send_email(subject, body):
    sender_email = "tu_correo"
    receiver_email = "destinatario_correo"
    password = "tu_contraseña"
    
    message = f"Subject: {subject}\n\n{body}"
    
    with smtplib.SMTP_SSL("smtp.gmail.com", 465) as server:
        server.login(sender_email, password)
        server.sendmail(sender_email, receiver_email, message)

# Monitorear mercado y enviar señal
def check_for_signals():
    while True:
        prices = get_market_data()
        rsi = calculate_rsi(prices)
        
        # Enviar señal si el RSI está en sobreventa (< 30) o sobrecompra (> 70)
        if rsi < 30:
            send_email("Señal de Compra", "El RSI está en sobreventa, considera una operación de compra.")
            print("Señal de Compra enviada!")
        elif rsi > 70:
            send_email("Señal de Venta", "El RSI está en sobrecompra, considera una operación de venta.")
            print("Señal de Venta enviada!")
        
        # Esperar un minuto antes de verificar de nuevo
        time.sleep(60)

# Iniciar la monitorización
check_for_signals()
