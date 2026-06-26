#include <PID_v1.h>
#include <DHT.h>
#include <LiquidCrystal.h>

#define DHTPIN 8
#define DHTTYPE DHT22
#define HEATER_PIN 9

DHT dht(DHTPIN, DHTTYPE);
LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

// Variáveis do PID
double setpoint = 27.0;
double input, output;

// Ganhos — ajuste conforme necessário
double Kp = 2.0, Ki = 0.5, Kd = 1.0;

PID myPID(&input, &output, &setpoint, Kp, Ki, Kd, DIRECT);

void setup() {
  Serial.begin(9600);
  dht.begin();
  pinMode(HEATER_PIN, OUTPUT);

  lcd.begin(16, 2);
  lcd.print("Iniciando...");
  delay(2000);
  lcd.clear();

  myPID.SetMode(AUTOMATIC);
  myPID.SetOutputLimits(0, 255);
}

void loop() {
  input = dht.readTemperature();

  if (isnan(input)) {
    Serial.println("Erro no sensor!");
    lcd.setCursor(0, 0);
    lcd.print("Erro no sensor! ");
    return;
  }

  myPID.Compute();
  analogWrite(HEATER_PIN, output);

  // Display LCD
  lcd.setCursor(0, 0);
  lcd.print("Temp: ");
  lcd.print(input, 1);
  lcd.print((char)223); // símbolo °
  lcd.print("C   ");

  lcd.setCursor(0, 1);
  lcd.print("Set:  ");
  lcd.print(setpoint, 1);
  lcd.print((char)223);
  lcd.print("C   ");

  // Serial Plotter — para gráfico no portfólio
  Serial.print("Setpoint:"); Serial.print(setpoint);
  Serial.print(",Temperatura:"); Serial.print(input);
  Serial.print(",Saida:"); Serial.println(output);

  delay(500);
}
