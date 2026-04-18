# I2C_Logger

Система для логирования данных с I2C-устройств на Windows Server 2016 с разграничением доступа.

## Настройка Windows Server 2016

Все команды выполняются в командной строке (cmd) от имени Администратора.

### 1. Создание пользователей

```cmd
net user operator1 P@ssw0rd /add
net user operator2 P@ssw0rd /add
```
### 2. Создание папки Data
```
mkdir C:\Data
```
### 3. Настройка NTFS-прав
```
icacls C:\Data /reset /t /q
icacls C:\Data /inheritance:r
icacls C:\Data /grant operator1:F
icacls C:\Data /grant operator2:RX
```
### 4. Проверка прав
```
icacls C:\Data
Ожидаемый результат:
C:\Data
    NT AUTHORITY\SYSTEM:(F)
    BUILTIN\Administrators:(F)
    WIN-...\operator1:(F)
    WIN-...\operator2:(RX)
```
<img width="974" height="802" alt="image" src="https://github.com/user-attachments/assets/157d5878-bf45-4960-b16a-e42db3460b77" />

### Скетч для Задания c Arduino 

## Алгоритм работы I2C-логгера на Arduino

### Что происходит

Arduino подключается к датчику по проводам SDA (данные) и SCL (тактовая частота). Arduino — главный, датчик — подчиненный. Arduino спрашивает — датчик отвечает.

### Алгоритм по шагам

**1. При запуске (один раз)**

- Открывается Serial-порт для отправки данных на компьютер
- Запускается I2C-шина (Wire.begin)
- Arduino становится ведущим устройством

**2. В основном цикле (бесконечно)**

- **Задержка** — ждет заданный интервал (например, 1 секунду)
- **Запрос** — Arduino обращается к датчику по его I2C-адресу и просит данные
- **Чтение** — Arduino принимает сырые байты от датчика
- **Преобразование** — сырые значения переводятся в понятные величины (температура, влажность и т.д.)
- **Вывод** — данные отправляются в Serial-порт на компьютер
- **Повтор** — всё начинается заново

### Базовый пример кода
```
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <OneWire.h>
#include <DallasTemperature.h>

#define ONE_WIRE_BUS 2

LiquidCrystal_I2C lcd(0x27, 16, 2);

OneWire oneWire(ONE_WIRE_BUS);
DallasTemperature sensors(&oneWire);

unsigned long previousMillis = 0;
unsigned long currentSeconds = 0;

void setup() {
  Serial.begin(9600);
  
  lcd.init();
  lcd.backlight();
  lcd.clear();
  
  sensors.begin();
  
  delay(1000);
}

void loop() {
  unsigned long currentMillis = millis();
  
  if (currentMillis - previousMillis >= 1000) {
    previousMillis = currentMillis;
    currentSeconds++;
    
    sensors.requestTemperatures();
    float temperatureC = sensors.getTempCByIndex(0);
    
    unsigned long hh = (currentSeconds % 86400) / 3600;
    unsigned long mm = (currentSeconds % 3600) / 60;
    unsigned long ss = currentSeconds % 60;
    
    Serial.print(hh);
    Serial.print(" ");
    Serial.print(mm);
    Serial.print(" ");
    Serial.print(ss);
    Serial.print(" темпер ");
    Serial.print(temperatureC, 1);
    Serial.println("C");
    
    lcd.setCursor(0, 0);
    lcd.print("Time: ");
    if (hh < 10) lcd.print("0");
    lcd.print(hh);
    lcd.print(":");
    if (mm < 10) lcd.print("0");
    lcd.print(mm);
    lcd.print(":");
    if (ss < 10) lcd.print("0");
    lcd.print(ss);
    lcd.print("  ");
    
    lcd.setCursor(0, 1);
    lcd.print("Temp: ");
    lcd.print(temperatureC, 1);
    lcd.print(" C");
    lcd.print("   ");
  }
}
```
### Фото в сборе и данных
<img width="3024" height="4032" alt="79D65678-33A7-4107-8DC9-F59490E3B72F" src="https://github.com/user-attachments/assets/94a825ba-bfd5-4444-ae19-03ba95883662" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/bd72b8c8-673a-4add-89ba-b5e4a60b8f1f" />


