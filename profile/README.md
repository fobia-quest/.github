<div align="center">

# 🗝️ FOBIA Escape Room Electronics
### Централизованная экосистема аппаратных модулей и тайников квеста

![Platform](https://img.shields.io/badge/Platform-ESP32-blue?logo=espressif)
![Framework](https://img.shields.io/badge/Framework-Arduino%20%2F%20PlatformIO-orange?logo=platformio)
![Network](https://img.shields.io/badge/Network-ESP--MESH-green)
![Data](https://img.shields.io/badge/Format-JSON%20v6-lightgrey)

---

</div>

## 📌 О проекте

Репозиторий объединяет микроконтроллерные модули, электронные загадки и централизованный пульт управления для квест-комнаты **«FOBIA»**.

Все устройства объединены в отказоустойчивую самоорганизующуюся ячеистую сеть без внешних роутеров. Если отдельный тайник экранирован стенами подвала, сигнал ретранслируется цепочкой через соседние узлы к пульту оператора.

---

## 🌐 Параметры сети ESP-MESH

| Параметр | Значение | Описание |
| :--- | :--- | :--- |
| **Библиотека** | `painlessMesh` (v1.5.0+) | Базовый стек mesh-сети на 2.4 ГГц |
| **Сериализация** | `ArduinoJson` (v6.x) | Формирование и парсинг пакетов |
| **SSID (Prefix)** | `FOBIA_QUEST_NET` | Идентификатор радиосети |
| **Password** | `FobiaSecret2026` | Ключ шифрования сети |
| **Port** | `5555` | TCP-порт сокетов сети |
| **UART Speed** | `115200` бод | Скорость вывода отладки |

---

## 🧩 Модули квеста

| Репозиторий | Роль | Описание и периферия |
| :--- | :--- | :--- |
| [**fobia-admin-hub**](https://github.com/fobia-quest/fobia-admin-hub) | **Root Node** | Пульт администратора: физические кнопки управления комнатой, мониторинг состояния узлов, аварийный сброс |
| [**Monster_Eyes_Lock**](https://github.com/fobia-quest/Monster_Eyes_Lock) | **Node** | Тайник «Глаза Монстра»: 2 геркона (`GPIO 13`, `14`), плавный ШИМ глаз (`GPIO 18`, `19`), силовой замок 12 В (`GPIO 23`) |
| [**esp32-quest-phone**](https://github.com/fobia-quest/esp32-quest-phone) | **Node** | Сюжетный ретро-телефон: DFPlayer Mini (аппаратный UART), геркон трубки, наборный диск, прием звонков от оператора |
| [**fobia-device-sdk**](https://github.com/fobia-quest/fobia-device-sdk) | **SDK** | Библиотека быстрой интеграции новых загадок в сеть за 5 строк кода |

---

## 📡 Протокол обмена данными (JSON)

### 1. Периодическая телеметрия узлов (каждые 3 сек)
```json
{
  "device": "Monster_Eyes_Lock",
  "door_open": false,
  "reed1": true,
  "reed2": true,
  "uptime_sec": 142
}

{
  "device": "esp32-quest-phone",
  "hook_off": false,
  "state": "ON_HOOK",
  "uptime_sec": 142
}

### 2. Команды управления с пульта оператора

Принудительное открытие тайника:
```json
{"target":"Monster_Eyes_Lock","cmd":"FORCE_OPEN"}
```

Принудительное закрытие замка:
```json
{"target":"Monster_Eyes_Lock","cmd":"FORCE_CLOSE"}
```

Входящий звонок на телефон игрокам (по умолчанию трек квеста, либо указанный трек):
```json
{"target":"esp32-quest-phone","cmd":"TRIGGER_RING","track":5}
```

Глобальный сброс сценария комнаты (или конкретного устройства):
```json
{"target":"ALL","cmd":"RESET"}
```

---

## 🛠️ Разработка нового устройства через SDK

Подключение нового реквизита сводится к использованию библиотеки **`fobia-device-sdk`**:

```cpp
#include <Arduino.h>
#include "FobiaDevice.h"

FobiaDevice puzzle("New_Puzzle");

void setup() {
  Serial.begin(115200);

  // Кастомные поля для телеметрии
  puzzle.onTelemetry([](JsonObject& data) {
    data["solved"] = false;
  });

  // Обработчик входящих команд
  puzzle.onCommand([](const String& cmd, JsonObject& payload) {
    if (cmd == "FORCE_OPEN") {
      // Логика аварийного открытия
    }
  });

  puzzle.begin();
}

void loop() {
  puzzle.update(); // Обязательный вызов в loop без delay!
}
```
