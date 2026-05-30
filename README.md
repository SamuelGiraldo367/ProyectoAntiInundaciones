# ProyectoAntiInundaciones
si
/*
   Nombre:          Samuel Giraldo, Sara Hernández y Alejandro Camacho
   Fecha:           2026-06-03
   NombreEjercicio: Proyecto Sistema Anti inundaciones
   Observaciones:   Lograr que el sistema de alerta parpadeara, pues al final para solucionarlo tocó crear un bool que cambiara el estado de alarma de true o false
   además tocó configurar el tema de las lecturas para que determinara si era o no una lluvia real y constante
*/
#include <Servo.h>

// Pines
const int PIN_SENSOR = A0;
const int PIN_BUZZER = 5;
const int PIN_LED = 2;

// Servo
Servo servoMotor;

// ==========================
// CONFIGURACIÓN DEL SENSOR
// ==========================

// En seco: valores altos (~1023)
// Mojado: valores bajos (~0)

const int UMBRAL_LLUVIA_FUERTE = 600;
const int HISTERESIS = 50;

// ==========================
// TIEMPOS DEL SISTEMA
// ==========================

// Tiempo mínimo para activar alarma
const unsigned long TIEMPO_MIN_ALERTA = 1000;

// Tiempo mínimo para volver a reposo
const unsigned long TIEMPO_MIN_REPOSO = 2000;

// Intervalo de la sirena intermitente
const unsigned long INTERVALO_SIRENA = 500;

// ==========================
// VARIABLES DE CONTROL
// ==========================

// 0 = reposo
// 1 = alarma activa
int estadoAlarma = 0;

// Guarda tiempos usando millis()
unsigned long tiempoInicioLectura = 0;
unsigned long ultimoCambioSirena = 0;

// Estado ON/OFF de la sirena
bool estadoIntermitente = false;

// ==========================
// SETUP
// ==========================

void setup() {

  Serial.begin(9600);

  pinMode(PIN_BUZZER, OUTPUT);
  pinMode(PIN_LED, OUTPUT);

  digitalWrite(PIN_BUZZER, LOW);
  digitalWrite(PIN_LED, LOW);

  // Configuración del servo
  servoMotor.attach(9);

  // Servo inicia en 0°
  servoMotor.write(0);

  Serial.println("=== SISTEMA DE LLUVIA INICIADO ===");
}

// ==========================
// LOOP PRINCIPAL
// ==========================

void loop() {

  // Leer sensor
  int lectura = analogRead(PIN_SENSOR);

  // Mostrar lectura
  Serial.print("Lectura del sensor: ");
  Serial.println(lectura);

  // =====================================
  // ESTADO 0 -> REPOSO
  // =====================================

  if (estadoAlarma == 0) {

    // ¿Hay lluvia fuerte?
    if (lectura <= UMBRAL_LLUVIA_FUERTE) {

      // Guarda el instante inicial
      if (tiempoInicioLectura == 0) {
        tiempoInicioLectura = millis();
      }

      // ¿Lleva suficiente tiempo lloviendo?
      if (millis() - tiempoInicioLectura >= TIEMPO_MIN_ALERTA) {

        activarAlarma();

        tiempoInicioLectura = 0;
      }

    } else {

      // Reinicia contador
      tiempoInicioLectura = 0;
    }
  }

  // =====================================
  // ESTADO 1 -> ALARMA ACTIVA
  // =====================================

  else {

    // ==========================
    // EFECTO SIRENA INTERMITENTE
    // ==========================

    if (millis() - ultimoCambioSirena >= INTERVALO_SIRENA) {

      ultimoCambioSirena = millis();

      // Cambia TRUE/FALSE
      estadoIntermitente = !estadoIntermitente;

      // LED intermitente
      digitalWrite(PIN_LED, estadoIntermitente);

      // BUZZER ACTIVO intermitente
      digitalWrite(PIN_BUZZER, estadoIntermitente);

      // Mensajes constantes
      Serial.println("[ALERTA] ¡Mucha lluvia detectada!");
      Serial.println("Rampa activada");
    }

    // =====================================
    // VERIFICAR SI DEJÓ DE LLOVER
    // =====================================

    if (lectura >= (UMBRAL_LLUVIA_FUERTE + HISTERESIS)) {

      if (tiempoInicioLectura == 0) {
        tiempoInicioLectura = millis();
      }

      // ¿Ya pasó suficiente tiempo seco?
      if (millis() - tiempoInicioLectura >= TIEMPO_MIN_REPOSO) {

        desactivarAlarma();

        tiempoInicioLectura = 0;
      }

    } else {

      // Sigue lloviendo
      tiempoInicioLectura = 0;
    }
  }

  // Pequeña pausa de lectura
  delay(200);
}

// ==========================
// ACTIVAR ALARMA
// ==========================

void activarAlarma() {

  estadoAlarma = 1;

  // Mover servo
  servoMotor.write(90);

  Serial.println("================================");
  Serial.println("[ALERTA] SISTEMA ACTIVADO");
  Serial.println("================================");
}

// ==========================
// DESACTIVAR ALARMA
// ==========================

void desactivarAlarma() {

  estadoAlarma = 0;

  // Apagar LED y buzzer
  digitalWrite(PIN_LED, LOW);
  digitalWrite(PIN_BUZZER, LOW);

  // Reiniciar estados
  estadoIntermitente = false;

  // Regresar servo
  servoMotor.write(0);

  Serial.println("================================");
  Serial.println("[INFO] La lluvia ha cesado");
  Serial.println("Rampa desactivada");
  Serial.println("================================");
}
