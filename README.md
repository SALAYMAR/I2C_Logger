# I2C_Logger

Система для логирования данных с I2C-устройств на Windows Server 2016 с разграничением доступа.

## Настройка Windows Server 2016

Все команды выполняются в командной строке (cmd) от имени Администратора.

### 1. Создание пользователей

```cmd
net user operator1 P@ssw0rd /add
net user operator2 P@ssw0rd /add
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

text
C:\Data
    NT AUTHORITY\SYSTEM:(F)
    BUILTIN\Administrators:(F)
    WIN-...\operator1:(F)
    WIN-...\operator2:(RX)
Скетч для Arduino
Базовый пример кода
cpp
#include <Wire.h>

#define DEVICE_ADDR 0x68

void setup() {
  Serial.begin(9600);
  Wire.begin();
}

void loop() {
  Wire.requestFrom(DEVICE_ADDR, 6);
  while (Wire.available()) {
    Serial.print(Wire.read());
    Serial.print(" ");
  }
  Serial.println();
  delay(1000);
}
Загрузка на GitHub
1. Создайте репозиторий на GitHub
Название: I2C_Logger

Скопируйте ссылку: https://github.com/ВАШ_ЛОГИН/I2C_Logger.git

2. Выполните команды в cmd
cmd
cd C:\путь_к_вашей_папке
git init
git add .
git commit -m "Первый коммит"
git remote add origin https://github.com/ВАШ_ЛОГИН/I2C_Logger.git
git push -u origin master
3. Обновление изменений
cmd
git add .
git commit -m "Описание изменений"
git push
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
