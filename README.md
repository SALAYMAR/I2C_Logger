# I2C_Logger

Система для логирования данных с I2C-устройств на Windows Server 2016 с разграничением доступа.

## Настройка Windows Server 2016

Все команды выполняются в командной строке (cmd) от имени Администратора.

### 1. Создание пользователей

```cmd
net user operator1 P@ssw0rd /add
net user operator2 P@ssw0rd /add
```
2. Создание папки Data
cmd
mkdir C:\Data
3. Настройка NTFS-прав
cmd
icacls C:\Data /reset /t /q
icacls C:\Data /inheritance:r
icacls C:\Data /grant operator1:F
icacls C:\Data /grant operator2:RX
4. Проверка прав
cmd
icacls C:\Data
Ожидаемый результат:
C:\Data
    NT AUTHORITY\SYSTEM:(F)
    BUILTIN\Administrators:(F)
    WIN-...\operator1:(F)
    WIN-...\operator2:(RX)
Скетч для Задания №1 Arduino 
Базовый пример кода
```
#include <Wire.h>
#include <RTClib.h>

RTC_DS3231 rtc;

const int tempPin = A0;

void setup() {
  Serial.begin(9600);
  Serial.println("Система запущена...");
  
  Wire.begin();
  
  // ПРОВЕРКА: пробуем инициализировать RTC
  if (!rtc.begin()) {
    Serial.println("Ошибка: RTC НЕ ОТВЕЧАЕТ!");
    Serial.println("Проверьте подключение модуля RTC:");
    Serial.println("- VCC -> 5V");
    Serial.println("- GND -> GND");
    Serial.println("- SDA -> A4");
    Serial.println("- SCL -> A5");
    Serial.println("Программа продолжит работу без RTC");
  } else {
    Serial.println("RTC модуль обнаружен!");
    
    // Устанавливаем время из времени компиляции, если RTC потерял питание
    if (rtc.lostPower()) {
      Serial.println("RTC потерял время, устанавливаю заново...");
      rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));
    }
  }
  
  Serial.println("Готов к работе! Начинаем вывод данных...");
}

void loop() {
  // Читаем температуру
  int sensorValue = analogRead(tempPin);
  float voltage = (sensorValue * 5.0) / 1024.0;
  float temperature = (voltage - 0.5) * 100.0;
  
  // Пытаемся получить время с RTC
  if (rtc.begin()) {  // Если RTC доступен
    DateTime now = rtc.now();
    
    Serial.print(now.hour(), DEC);
    Serial.print(" ,, ");
    Serial.print(now.minute(), DEC);
    Serial.print(" ,, ");
    Serial.print(now.second(), DEC);
    Serial.print(" темп ");
    Serial.println(temperature, 1);
  } else {
    // Если RTC не отвечает — выводим только температуру
    Serial.print("RTC не найден, темп ");
    Serial.println(temperature, 1);
  }
  
  delay(1000);
}
```
Полный скрипт для Windows Server 2016
cmd
net user operator1 P@ssw0rd /add
net user operator2 P@ssw0rd /add
mkdir C:\Data
icacls C:\Data /reset /t /q
icacls C:\Data /inheritance:r
icacls C:\Data /grant operator1:F
icacls C:\Data /grant operator2:RX
icacls C:\Data
text

Вот. Копируй этот текст целиком и вставляй в README.md на GitHub
