### Escaneo instantáneo.
Instalación interactiva en la que un sensor de proximidad mide la distancia de un usuario y activa un proceso de escaneo (representado por el script ft.sh) cuando el usuario está lo suficientemente cerca.

<img src="https://raw.githubusercontent.com/mauricixx/metron/refs/heads/main/img/metron_prototipo_mam_dic_2024.jpg" width="1080" height="810" />



#### Técnica:
Código Arduino para sensor de distancia ultrasónico.

```js
#define TRIGGER_PIN 9
#define ECHO_PIN 10

void setup() {
  Serial.begin(9600);
  pinMode(TRIGGER_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
}

void loop() {
  digitalWrite(TRIGGER_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIGGER_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIGGER_PIN, LOW);

  long duration = pulseIn(ECHO_PIN, HIGH);
  float distance = duration * 0.034 / 2;  // Conversión a cm
  
  Serial.println(distance);  // Enviar la distancia por serial
  delay(100);  // Retardo para lectura
}
```

------------------

##### 1.	Configurar la conexión serial con Arduino para ejecutar script de Python en una Raspberry Pi:
•	Usa la biblioteca serial para conectarse al puerto especificado (/dev/ttyACM0) con una velocidad de comunicación de 9600 baudios.
•	Si no logra establecer la conexión, muestra un error y cierra el programa.
###### 2.	Lee datos del puerto serial en un bucle infinito:
•	Intenta leer líneas de texto enviadas desde el dispositivo conectado al puerto serial.
•	Los datos leídos se procesan como cadenas, se eliminan espacios en blanco y saltos de línea, y se verifica si contienen solo números (con .isdigit()).

###### 3.	Interpreta los datos como una distancia:
•	Convierte los datos recibidos en números enteros si son válidos.
•	Imprime la distancia en centímetros con el mensaje:
"La distancia entre tu corazon y la maquina es: {distance} cm".

 ###### 4.	Condición para ejecutar un script externo:
•	Si la distancia es menor o igual a 30 cm y el script aún no ha sido ejecutado, muestra el mensaje "Escaneando un cuerpo..." y ejecuta un script de Bash llamado ft.sh ubicado en la ruta /home/x_x/SCAN/FT/.	
•	Después de ejecutar el script, establece una bandera (script_executed) en True para evitar que el script se ejecute repetidamente mientras la condición siga siendo válida.

 ###### 5.	Reinicio de la bandera:
•	Si la distancia aumenta a más de 30 cm, la bandera script_executed se reinicia, permitiendo que el script vuelva a ejecutarse si la distancia cae nuevamente por debajo de 30 cm.

 ###### 6.	Manejo de errores:
•	Si ocurre algún error al leer los datos seriales o procesarlos, imprime el error pero no detiene el programa.

###### ¿Qué se requiere para que funcione?

•	Un dispositivo (como un Arduino) conectado al puerto serial, que envíe datos de distancia en formato numérico.
•	Un script de Bash (ft.sh) que el programa ejecutará al cumplirse la condición.
•	Ajustar el puerto serial (/dev/ttyACM0) según el sistema operativo, por ejemplo, en Windows podría ser algo como COM3.




##### Código de Python:

```py
import serial  # Biblioteca para manejar la comunicaci  n serial con dispositivos como Arduino.
import os  # Biblioteca para interactuar con el sistema operativo, como ejecutar scripts.

# Configuraci  n de la conexi  n serial
serial_port = "/dev/ttyACM0"  # Especifica el puerto serial donde est   conectado el Arduino. Cambia esto seg  n tu sistema.
baud_rate = 9600  # Velocidad de comunicaci  n en bits por segundo. Debe coincidir con la configuraci  n del Arduino.

try:
    # Intenta abrir la conexi  n serial con el dispositivo.
    ser = serial.Serial(serial_port, baud_rate, timeout=1)  
    print(f"Listening on {serial_port}")  # Confirma que se est   escuchando en el puerto especificado.
except Exception as e:
    # Si hay un error al abrir la conexi  n, muestra el error y cierra el programa.
    print(f"Error: {e}")  
    exit()  

# Inicializa una bandera para evitar ejecutar el script repetidamente.
script_executed = False  

while True:  # Bucle infinito para leer datos continuamente.
    try:
        # Leer una l  nea de datos desde la conexi  n serial.
        line = ser.readline().decode('utf-8').strip()  
        # `.readline()` obtiene una l  nea de datos seriales.
        # `.decode('utf-8')` convierte los datos de bytes a una cadena.
        # `.strip()` elimina espacios en blanco y caracteres de nueva l  nea.

        if line.isdigit():  # Verifica si la l  nea le  da es un n  mero.
            distance = int(line)  # Convierte la cadena a un entero.
            print(f"La distancia entre tu corazon y la maquina es: {distance} cm")  
            # Muestra la distancia recibida en la consola.

            # Si la distancia es menor o igual a 30 y el script aun no ha sido ejecutado:
            if distance <= 30 and not script_executed:
                print("Escaneando un cuerpo...")  # Mensaje informativo.
                os.system("bash /home/x_x/SCAN/FT/ft.sh")
                # Ejecuta un script de Bash ubicado en la ruta especificada.
                script_executed = True  # Cambia la bandera para evitar reejecutar el script.

        # Si la distancia es mayor de 30, reinicia la bandera.
        if line.isdigit() and int(line) > 30:
            script_executed = False  

    except Exception as e:
        # Maneja cualquier error que ocurra durante la lectura de los datos seriales.
        print(f"Error reading serial data: {e}")  
```
