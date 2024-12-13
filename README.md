### El cuerpo presente como gatillador de rutinas técnicas en las artes mediales.
Metron: escaneo instantáneo. Instalación interactiva.


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
