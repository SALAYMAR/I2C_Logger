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
### Фото в сборе и данных
<img width="3024" height="4032" alt="79D65678-33A7-4107-8DC9-F59490E3B72F" src="https://github.com/user-attachments/assets/94a825ba-bfd5-4444-ae19-03ba95883662" />

<img width="3024" height="4032" alt="FA6390D8-ABC7-449D-B0F0-463EF4C0FE16" src="https://github.com/user-attachments/assets/b624526e-05d9-4987-b940-7c2e9deddd45" />

<img width="3024" height="4032" alt="142F8363-2728-43AB-8A94-59CCA48CC5D0" src="https://github.com/user-attachments/assets/47a8d75f-3007-4535-b221-d69f88388e7b" />

