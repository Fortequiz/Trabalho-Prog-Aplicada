#include <stdio.h>
#include <stdlib.h>
#include <windows.h>
#include <locale.h>
#include <string.h>

#define PIN_DIG_ENT_1 11
#define PIN_DIG_ENT_2 10
#define PIN_ANA_ENT_1 A0
#define PIN_POT A1
#define PIN_ANA_SAI_1 A3
#define PIN_ANA_SAI_2 A5
#define PIN_DIG_SAI_1 12
#define PIN_DIG_SAI_2 13
#define PIN_SENSOR_PRESENCA 2
#define PIN_BOTAO 3
#define PIN_LED_1 4
#define PIN_LED_2 5
#define PIN_COOLER 6

int estadoPinDigital1 = 0;
int estadoPinDigital2 = 0;
int estadoPinAnalogico1 = 0;
int estadoPinSensorPresenca = 0;
int estadoPinBotao = 0;

void setup() {
  pinMode(PIN_DIG_ENT_1, INPUT);
  pinMode(PIN_DIG_ENT_2, INPUT);
  pinMode(PIN_ANA_ENT_1, INPUT);
  pinMode(PIN_POT, INPUT);
  pinMode(PIN_SENSOR_PRESENCA, INPUT);
  pinMode(PIN_BOTAO, INPUT_PULLUP);

  pinMode(PIN_ANA_SAI_1, OUTPUT);
  pinMode(PIN_ANA_SAI_2, OUTPUT);
  pinMode(PIN_DIG_SAI_1, OUTPUT);
  pinMode(PIN_DIG_SAI_2, OUTPUT);
  pinMode(PIN_LED_1, OUTPUT);
  pinMode(PIN_LED_2, OUTPUT);
  pinMode(PIN_COOLER, OUTPUT);

  Serial.begin(9600); // Inicializa a comunicação serial
}

void loop() {
  if (Serial.available()) {
    char Funcao = Serial.read();

    switch (Funcao) {
      case 'a':
        AcionarSaidasPerifericos();
        break;
      case 'b':
        LerEstadoPerifericos();
        break;
      case 'c':
        LerValoresAnalogicos();
        break;
      case 'd':
        EscreverValoresAnalogicos();
        break;
      case 'x':
        // Encerra o programa
        Serial.println("Programa encerrado");
        while (true) {} // Aguarda indefinidamente
        break;
      default:
        Serial.println("Entrada inválida! Digite novamente.");
        break;
    }
  }

  // Lê o estado dos periféricos
  estadoPinDigital1 = digitalRead(PIN_DIG_ENT_1);
  estadoPinDigital2 = digitalRead(PIN_DIG_ENT_2);
  estadoPinAnalogico1 = analogRead(PIN_ANA_ENT_1);
  estadoPinSensorPresenca = digitalRead(PIN_SENSOR_PRESENCA);
  estadoPinBotao = digitalRead(PIN_BOTAO);

  // Verifica o estado do sensor de presença
  estadoPinSensorPresenca = digitalRead(PIN_SENSOR_PRESENCA);

  // Verifica o estado do botão
  estadoPinBotao = digitalRead(PIN_BOTAO);
}

int digitalRead(int pin) {
  return digitalRead(pin); // Usa a função digitalRead do Arduino para ler o pino digital
}

int analogRead(int pin) {
  return analogRead(pin); // Usa a função analogRead do Arduino para ler o pino analógico
}

void PassaPinoArduino(int PIN) {
  char PINO[4]; // Recebe a Saída Digital
  sprintf(PINO, "%03d", PIN); // Converte Int para String com 3 dígitos
  Serial.print(PINO);
}

void PassaRespostaArduino(char resposta) {
  Serial.print(resposta);
}

void LerRespostaArduino() {
  char response[255];
  if (Serial.available()) {
    int bytesRead = Serial.readBytes(response, sizeof(response) - 1);
    response[bytesRead] = '\0';
    float valor = atof(response);
    Serial.print("Valor em volts: ");
    Serial.println(valor);
  } else {
    ErroComunic();
  }
}

void LerEstadoPerifericos() {
  Serial.print("Estado dos periféricos:\n");
  Serial.print("Entrada Digital 1: ");
  Serial.println(estadoPinDigital1);
  Serial.print("Entrada Digital 2: ");
  Serial.println(estadoPinDigital2);
  Serial.print("Entrada Analógica 1: ");
  Serial.println(estadoPinAnalogico1);
  Serial.print("Sensor de Presença: ");
  Serial.println(estadoPinSensorPresenca);
  Serial.print("Botão: ");
  Serial.println(estadoPinBotao);
}

void AcionarSaidasPerifericos() {
  int Escolha;

  Serial.println("Digite 1 para ligar/desligar o LED Amarelo");
  Serial.println("Digite 2 para ligar/desligar o LED Vermelho");
  Serial.println("Digite 3 para ligar/desligar o Cooler");
  Serial.println("Digite 0 para voltar");

  while (true) {
    if (Serial.available()) {
      Escolha = Serial.parseInt();
      if (Escolha == 0) {
        break;
      } else if (Escolha == 1) {
        digitalWrite(PIN_LED_1, !digitalRead(PIN_LED_1)); // Inverte o estado lógico do LED 1
        PassaRespostaArduino('a');
        PassaPinoArduino(PIN_LED_1);
      } else if (Escolha == 2) {
        digitalWrite(PIN_LED_2, !digitalRead(PIN_LED_2)); // Inverte o estado lógico do LED 2
        PassaRespostaArduino('a');
        PassaPinoArduino(PIN_LED_2);
      } else if (Escolha == 3) {
        digitalWrite(PIN_COOLER, !digitalRead(PIN_COOLER)); // Inverte o estado lógico do Cooler
        PassaRespostaArduino('a');
        PassaPinoArduino(PIN_COOLER);
      }
    }
  }
}

void LerValoresAnalogicos() {
  PassaRespostaArduino('c');
  PassaPinoArduino(PIN_ANA_SAI_2);
  LerRespostaArduino();
}

void EscreverValoresAnalogicos() {
  int valorPotenciometro = analogRead(PIN_POT);
  int valorLuminosidade = map(valorPotenciometro, 0, 1023, 0, 255); // Mapeia o valor do potenciômetro para a faixa de 0 a 255

  analogWrite(PIN_ANA_SAI_1, valorLuminosidade); // Define o valor de luminosidade do LED
  PassaRespostaArduino('d');
  PassaPinoArduino(PIN_ANA_SAI_1);
}

