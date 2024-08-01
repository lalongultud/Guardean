# Guardean
guardean es un sistema IOT con la capacidad brindar informacion de manera activa el estado de tus plantas y sobre todo de como poder mejorar el cuidado de ellas


Proyecto de Monitorización de Plantas con Raspberry Pi y Arduino
Este proyecto utiliza una Raspberry Pi para recibir datos de sensores conectados a un Arduino y visualizarlos a través de un servidor web usando Flask. A continuación, se detallan los pasos para configurarlo

1. Configuración Inicial de la Raspberry Pi
    Instalación del Sistema Operativo:

    Instala Raspberry Pi OS en una tarjeta SD.
    Inserta la tarjeta en la Raspberry Pi y enciéndela.
    Actualización del Sistema:

    Abre una terminal y actualiza el sistema:
  
      bash
      sudo apt update
      sudo apt upgrade

    Habilitación de la Interfaz Serial:

    Abre la configuración de Raspberry Pi:

      bash
      sudo raspi-config

     Navega a Interfacing Options > Serial.
    Habilita la interfaz serial y desactiva la consola serial.
    Instalación de Dependencias:

    Instala Python y herramientas necesarias:

      bash
      sudo apt install python3 python3-pip python3-venv


3. Configuración del Entorno de Desarrollo
    Creación de un Entorno Virtual:

    Crea y activa un entorno virtual para tu proyecto:

      bash
      python3 -m venv myenv
      source myenv/bin/activate

    Instalación de Paquetes Python:

    Instala Flask y pyserial:

      bash
      pip install Flask pyserial


5. Conexión y Configuración del Arduino
    Carga del Código en Arduino:
    Conecta tu Arduino a la Raspberry Pi.

    Usa el IDE de Arduino para cargar el siguiente código (ajustado para tus sensores específicos):

        cpp
        #include <DHT.h>

        #define DHTPIN 2
        #define DHTTYPE DHT11
        #define SOIL_SENSOR_PIN A0
        #define READ_INTERVAL 5000

        DHT dht(DHTPIN, DHTTYPE);

        void setup() {
            Serial.begin(9600);
            dht.begin();
        }

        void loop() {
            int soilMoisture = map(analogRead(SOIL_SENSOR_PIN), 0, 1023, 100, 0);
            float h = dht.readHumidity();
            float t = dht.readTemperature();

            if (isnan(h) || isnan(t)) {
                Serial.println("Error obteniendo los datos del sensor DHT11");
                delay(READ_INTERVAL);
                return;
            }

            float hic = dht.computeHeatIndex(t, h, false);

            String output = "SoilMoisture: " + String(soilMoisture) + 
                            "% Humidity: " + String(h) + 
                            "% Temperature: " + String(t) + 
                            "*C HeatIndex: " + String(hic) + "*C";

            Serial.println(output);

            delay(READ_INTERVAL);
        }


6. Desarrollo del Servidor Flask
    Creación del Servidor Flask:

    Crea un archivo llamado web-viewer.py en tu proyecto:
        python
        from flask import Flask, jsonify
        import serial
        import threading

        app = Flask(__name__)

        # Variable para almacenar los datos leídos del Arduino
        sensor_data = {}

        # Función para leer datos del Arduino
        def read_arduino():
            global sensor_data
            arduino = serial.Serial('/dev/ttyACM0', 9600, timeout=1)
            arduino.flush()
            while True:
                if arduino.in_waiting > 0:
                    try:
                        line = arduino.readline().decode('utf-8').strip()
                        data_parts = line.split(' ')
                        sensor_data = {
                            'soil_moisture': data_parts[1],
                            'humidity': data_parts[3],
                            'temperature': data_parts[5].split('*')[0],
                            'heat_index': data_parts[7].split('*')[0]
                        }
                    except Exception as e:
                        print(f"Error: {e}")

        # Ruta para obtener los datos
        @app.route('/data', methods=['GET'])
        def get_data():
            return jsonify(sensor_data)

        if __name__ == '__main__':
            # Iniciar la lectura del Arduino en un hilo separado
            thread = threading.Thread(target=read_arduino)
            thread.daemon = True
            thread.start()
            # Iniciar el servidor Flask
            app.run(host='0.0.0.0', port=5000)


    Ejecución del Servidor Flask:

    Desde el entorno virtual, ejecuta el servidor Flask:

        bash
        python web-viewer.py
   
    Accede a la página web desde un navegador usando la dirección IP de la Raspberry Pi, por ejemplo: http://<IP_DE_TU_RASPBERRY_PI>:5000/data


7. Monitoreo y Mantenimiento
    Verificación de Conexiones:

    Asegúrate de que el Arduino esté correctamente conectado y que la Raspberry Pi pueda leer los datos.
    Solución de Problemas Comunes:

    Verifica el puerto serial y el formato de los datos si no ves la información esperada.
    Seguridad:

    Este proyecto fue realizado en el marco del curso IoT Essentials Developer impartido por 
    [Codigo IoT ](https://www.codigoiot.com/) y organizado por la 
    [Asociación Mexicana del Internet de las Cosas](https://www.asociacioniot.org/).
