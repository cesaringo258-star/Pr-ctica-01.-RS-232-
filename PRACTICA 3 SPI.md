//CODIGO DEL MAESTRO 
#include <SPI.h>

// Definición de pines para la selección de cada esclavo
const int pinEsclavo1 = 10;
const int pinEsclavo2 = 9;

void setup() {
  Serial.begin(9600);
  
  // Configurar pines de selección como salidas
  pinMode(pinEsclavo1, OUTPUT);
  pinMode(pinEsclavo2, OUTPUT);
  
  // Desactivar ambos esclavos al inicio (HIGH)
  digitalWrite(pinEsclavo1, HIGH);
  digitalWrite(pinEsclavo2, HIGH);

  // Inicializar el bus SPI
  SPI.begin();
  
  // Configuración SPI: Velocidad 1MHz, Bit más significativo primero (MSBFIRST), Modo SPI 0
  SPI.beginTransaction(SPISettings(1000000, MSBFIRST, SPI_MODE0));
  
  Serial.println("--- Maestro SPI Inicializado ---");
}

void loop() {
  // ==========================================
  // COMUNICACIÓN CON EL ESCLAVO 1
  // ==========================================
  digitalWrite(pinEsclavo1, LOW);        // Seleccionar Esclavo 1
  SPI.transfer('A');                    // Envía comando 'A' para que el esclavo prepare su respuesta
  delayMicroseconds(50);                // Pequeña pausa para que el esclavo procese la interrupción
  char respuesta1 = SPI.transfer(0x00); // Envía byte vacío para recoger la respuesta cargada
  digitalWrite(pinEsclavo1, HIGH);       // Deseleccionar Esclavo 1
  
  // Mostrar verificación en el Monitor Serie
  Serial.print("Esclavo 1 verificado. Respuestarecibida: ");
  Serial.println(respuesta1);
  
  delay(1000); // Esperar 1 segundo antes de pasar al siguiente esclavo

  // ==========================================
  // COMUNICACIÓN CON EL ESCLAVO 2
  // ==========================================
  digitalWrite(pinEsclavo2, LOW);        // Seleccionar Esclavo 2
  SPI.transfer('B');                    // Envía comando 'B'
  delayMicroseconds(50);                
  char respuesta2 = SPI.transfer(0x00); // Recoge la respuesta del Esclavo 2
  digitalWrite(pinEsclavo2, HIGH);       // Deseleccionar Esclavo 2
  
  // Mostrar verificación en el Monitor Serie
  Serial.print("Esclavo 2 verificado. Respuesta recibida: ");
  Serial.println(respuesta2);
  
  delay(1000); // Esperar 1 segundo para reiniciar el ciclo
}

//CODIGO ESCLAVO 1
#include <SPI.h>

volatile char datoRecibido = 0;
volatile bool nuevoDato = false;

void setup() {
  Serial.begin(9600);
  
  // El pin MISO (POCI) DEBE ser salida en el esclavo para poder responder
  pinMode(MISO, OUTPUT);
  
  // Activar el modo Esclavo modificando el registro de control SPI (SPCR)
  SPCR |= _BV(SPE);
  
  // Habilitar las interrupciones del módulo SPI
  SPI.attachInterrupt();
  
  Serial.println("Esclavo 1 Listo...");
}

// Rutina de Servicio de Interrupción (ISR) se ejecuta automáticamente al recibir un byte
ISR(SPI_STC_vect) {
  datoRecibido = SPDR; // Guardar el comando enviado por el maestro
  
  // Preparar inmediatamente el carácter '1' en el registro de datos (SPDR)
  // Este carácter será enviado de vuelta en el segundo ciclo de reloj del maestro
  SPDR = '1'; 
  nuevoDato = true;
}

void loop() {
  // Si llegó un dato, lo imprimimos en el monitor serie local de este Arduino
  if (nuevoDato) {
    Serial.print("Maestro envio: ");
    Serial.println(datoRecibido);
    nuevoDato = false;
  }
}


//CODIGO ESCLAVO 2 
#include <SPI.h>

volatile char datoRecibido = 0;
volatile bool nuevoDato = false;

void setup() {
  Serial.begin(9600);
  
  // El pin MISO (POCI) DEBE ser salida en el esclavo para poder responder
  pinMode(MISO, OUTPUT);
  
  // Activar el modo Esclavo modificando el registro de control SPI (SPCR)
  SPCR |= _BV(SPE);
  
  // Habilitar las interrupciones del módulo SPI
  SPI.attachInterrupt();
  
  Serial.println("Esclavo 2 Listo...");
}

// Rutina de Servicio de Interrupción (ISR)
ISR(SPI_STC_vect) {
  datoRecibido = SPDR; 
  
  // Preparar inmediatamente el carácter '2' para responderle al maestro
  SPDR = '2'; 
  nuevoDato = true;
}

void loop() {
  if (nuevoDato) {
    Serial.print("Maestro envio: ");
    Serial.println(datoRecibido);
    nuevoDato = false;
  }
}



