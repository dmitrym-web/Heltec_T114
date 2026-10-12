# Heltec T114 — подключение к удалённому MeshMonitor

## 1. Общая схема

    T114 → USB или BLE → Компьютер → TCP или MQTT → MeshMonitor

| # | T114 → Компьютер | Компьютер → MeshMonitor | Что на компьютере |
|:---:|---|---|---|
| 1 | USB | TCP | Serial Bridge |
| 2 | USB | MQTT | Serial Bridge + MQTT Proxy |
| 3 | BLE | TCP | BLE Bridge |
| 4 | BLE | MQTT | BLE Bridge + MQTT Proxy |

---

## 2. Характеристики Heltec T114

| Параметр | Значение |
|---|---|
| Микроконтроллер | Nordic nRF52840 (ARM Cortex-M4F, 64 МГц) |
| ОЗУ | 256 КБ |
| Флеш-память | 1 МБ |
| LoRa-чип | Semtech SX1262 |
| Беспроводная связь | Bluetooth 5.0 (BLE) |
| LoRa-диапазоны | 470–510 МГц / 863–928 МГц |
| Мощность передачи (TX) | 21 ± 1 дБм |
| Чувствительность (RX) | до −135 дБм (SF12/125 кГц) |
| Питание | USB-C (5 В), Li-Ion (3.7 В), солнечная панель |
| Энергопотребление (сон) | 11 мкА (заявлено), 7.5–8 мА (реально) |
| Энергопотребление (актив) | ~9 мА |
| GPS-модуль (доп.) | +8 мА даже при выключенном |
| Размеры | 50.80 × 22.86 мм |
| Рабочая температура | −20…+70 °C |

### Распиновка и интерфейсы

| Интерфейс | Кол-во | Назначение |
|---|:---:|---|
| SPI | 2 | LoRa-чип, дисплей, флеш-память |
| I2C | 2 | Часы реального времени (RTC) и др. |
| UART | 2 | GPS-модуль и др. |
| GPIO | 21 | АЦП (ADC), ШИМ (PWM) |
| GPIO 4 | 1 | АЦП для измерения напряжения батареи |
| GPIO 14 | 1 | Управление адресными RGB (NeoPixel) |
| P1.03 | 1 | Встроенный зелёный светодиод |
| GPS | 2 | GPIO 37/39 для модуля GPS |

---

## 3. Шаг 1 — Прошивка T114 через компьютер

T114 прошивается двумя способами: через веб-флешер (WebSerial) или вручную через UF2-файл. Оба работают с компьютера по USB-кабелю.

**Важно перед прошивкой:** Подключи антенну к T114 перед включением. Передача без антенны может необратимо повредить LoRa-чип.

### Способ 1: Веб-флешер (Chrome / Edge)

| Шаг | Действие |
|---|---|
| 1 | Подключи T114 к компьютеру по USB |
| 2 | Открой `flasher.meshtastic.org` (или `flasher.meshcore.io`) **в Chrome или Edge** |
| 3 | Выбери плату **Heltec T114** |
| 4 | Выбери версию прошивки и вариант (Companion / Repeater / Room Server) |
| 5 | Нажми **Flash** |
| 6 | Дважды нажми **RST** на T114 (быстро) |
| 7 | T114 определится как USB-накопитель. Флешер запишет прошивку |
| 8 | После записи T114 перезагрузится |

### Способ 2: Ручная прошивка UF2-файлом

| Шаг | Действие |
|---|---|
| 1 | Подключи T114 к компьютеру по USB |
| 2 | Дважды нажми **RST** на T114 **быстро, в течение ~0.5 сек** |
| 3 | T114 определится как USB-накопитель с именем **`HT-n5262`** |
| 4 | Скачай **`.uf2`** файл прошивки с `flasher.meshtastic.org` или `flasher.meshcore.io` |
| 5 | Перетащи `.uf2` файл в корень USB-накопителя |
| 6 | Дождись окончания копирования. T114 перезагрузится автоматически |

### Траблшутинг прошивки

| Проблема | Причина | Решение |
|---|---|---|
| **`HT-n5262` не появляется** | Ритм нажатия неверный | Нажимай **очень быстро**, почти как двойной клик мышью. Попробуй несколько раз подряд |
| **`HT-n5262` не появляется** | Кабель только для зарядки | Замени USB-кабель на **data-кабель** (не charge-only) |
| **Устройство не определяется как COM-порт** | Нет драйверов | Установи драйверы для nRF52840 (обычно не требуется, но бывает на Windows) |
| **Прошивка записывается, но устройство не загружается** | Битый файл или прерванная запись | Скачай `.uf2` заново, повтори запись. Не отключай кабель до завершения |
| **Устройство зависло в DFU-режиме** | Bootloader не загрузил приложение | Дважды нажми **RST**, чтобы выйти. Если не помогает — сделай **Erase Flash** (см. ниже) |
| **UF2-файл не копируется** | Bootloader старый | Обнови bootloader через `adafruit-nrfutil` или через **OTAFIX bootloader** |

### Полный сброс (Erase Flash)

| Шаг | Действие |
|---|---|
| 1 | Войди в DFU-режим (двойной **RST**) |
| 2 | В веб-флешере нажми иконку **корзины** рядом с кнопкой Flash |
| 3 | Скачай `flash_erase-nRF32_softdevice_v6.uf2` (правый клик → «Сохранить как») |
| 4 | Перетащи файл в USB-накопитель T114 |
| 5 | T114 перезагрузится с полностью стёртой памятью. Прошей новую прошивку |

### Обновление по воздуху (OTA) — только для Meshtastic

T114 поддерживает **Bluetooth OTA** через приложение **nRF Device Firmware Update** (Android / iOS). Требуется **`.zip`** файл (не `.uf2`).

| Шаг | Действие |
|---|---|
| 1 | Установи **nRF Device Firmware Update** в App Store / Play Store (поиск: `nrf dfu`) |
| 2 | Скачай **ZIP**-версию прошивки с `flasher.meshtastic.org` |
| 3 | В приложении DFU: Settings → включи **Packet receipt notifications** → **Number of Packets = 8** для T114 |
| 4 | Выбери ZIP-файл и устройство T114 в DFU-приложении |
| 5 | Нажми **Upload**. Процесс занимает 2–5 минут |
| 6 | Держи экран телефона включённым, не уводи приложение в фон |

**Рекомендуется:** перед OTA установить **OTAFIX bootloader** через USB. Он делает OTA-обновления надёжнее и позволяет восстановиться после неудачной прошивки.

---

## 4. Шаг 2 — Подготовка компьютера

На компьютере должна быть установлена утилита **Meshtastic Python CLI**. Без неё команд `meshtastic --set ...` в системе нет.

### Установка CLI

| ОС | Команды |
|---|---|
| **Windows** | 1. Установи Python 3.9+ с python.org, галочка **Add Python to PATH**<br>2. `pip3 install --upgrade meshtastic`<br>3. Проверка: `meshtastic --version` |
| **macOS** | `brew install python`<br>`brew install pipx`<br>`pipx ensurepath`<br>`pipx install "meshtastic[cli]"` |
| **Linux** | `sudo apt install python3-pip`<br>`pip3 install --upgrade "meshtastic[cli]"`<br>`sudo usermod -aG dialout $USER`<br>перелогиниться |

### Определение порта T114

| ОС | Как узнать порт |
|---|---|
| **Windows** | Диспетчер устройств → Порты (COM и LPT) → например `COM3` |
| **Linux / macOS** | `ls /dev/tty*` → `/dev/ttyUSB0` или `/dev/ttyACM0` |

Все команды ниже подаются с флагом `--port`, например:

    meshtastic --port COM3 --set serial.enabled true

---

## 5. Шаг 3 — Настройка T114

| Сценарий | Serial-режим | MQTT Client Proxy | Uplink/Downlink |
|---|:---:|:---:|:---:|
| 1 (USB → TCP) | Да | Нет | Нет |
| 2 (USB → MQTT) | Да | Да | Да |
| 3 (BLE → TCP) | Нет | Нет | Нет |
| 4 (BLE → MQTT) | Нет | Да | Да |

**Serial-режим (сценарии 1, 2):**

    meshtastic --port COM3 --set serial.enabled true --set serial.echo false --set serial.mode SIMPLE --set serial.baud BAUD_115200

**MQTT Client Proxy (сценарии 2, 4):**

    meshtastic --port COM3 --set mqtt.enabled true
    meshtastic --port COM3 --set mqtt.proxy_to_client_enabled true

`proxy_to_client_enabled` — обязательно. Без него T114 не будет передавать MQTT-трафик через компьютер.

**Uplink/Downlink (сценарии 2, 4):**

    meshtastic --port COM3 --ch-index 0 --ch-set uplink_enabled true --ch-set downlink_enabled true

---

## 6. Шаг 4 — Настройка компьютера

### Сценарий 1. USB → TCP

    T114 → USB → Serial Bridge (TCP:4403) → MeshMonitor

**T114:** только Serial-режим.
**Компьютер:** один контейнер.

    # docker-compose.yml
    services:
      serial-bridge:
        image: ghcr.io/yeraze/meshtastic-serial-bridge:latest
        container_name: meshtastic-serial-bridge
        devices:
          - /dev/ttyUSB0:/dev/ttyUSB0
        ports:
          - "4403:4403"
        restart: unless-stopped
        environment:
          - SERIAL_DEVICE=/dev/ttyUSB0
          - BAUD_RATE=115200
          - TCP_PORT=4403

**MeshMonitor:** `Dashboard → Sources → Add Source` → тип `meshtastic_tcp` → адрес `serial-bridge` → порт `4403`.

**Когда использовать:** MeshMonitor и компьютер в одной сети.

---

### Сценарий 2. USB → MQTT

    T114 → USB → Serial Bridge (TCP:4403) → MeshMonitor → Virtual Node (4404) ← MQTT Proxy → Embedded Broker

**T114:** Serial-режим + MQTT Client Proxy + Uplink/Downlink.
**Компьютер:** два контейнера.

    # docker-compose.yml
    services:
      serial-bridge:
        image: ghcr.io/yeraze/meshtastic-serial-bridge:latest
        container_name: meshtastic-serial-bridge
        devices:
          - /dev/ttyUSB0:/dev/ttyUSB0
        ports:
          - "4403:4403"
        restart: unless-stopped
        environment:
          - SERIAL_DEVICE=/dev/ttyUSB0
          - BAUD_RATE=115200
          - TCP_PORT=4403

      mqtt-proxy:
        image: ghcr.io/ln4cy/mqtt-proxy:latest
        container_name: mqtt-proxy
        restart: unless-stopped
        depends_on:
          - serial-bridge
        environment:
          - INTERFACE_TYPE=tcp
          - TCP_HOST=serial-bridge
          - TCP_PORT=4403

**MeshMonitor:** включить Virtual Node (порт `4404`), убедиться, что Embedded MQTT Broker добавлен как источник.

**Когда использовать:** MeshMonitor удалён, данные идут через MQTT.

---

### Сценарий 3. BLE → TCP

    T114 → BLE → BLE Bridge (TCP:4403) → MeshMonitor

**T114:** ничего дополнительно.
**Компьютер:** один контейнер.

    # docker-compose.yml
    services:
      ble-bridge:
        image: ghcr.io/yeraze/meshtastic-ble-bridge:latest
        container_name: meshtastic-ble-bridge
        privileged: true
        ports:
          - "4403:4403"
        restart: unless-stopped
        volumes:
          - /var/run/dbus:/var/run/dbus
          - /var/lib/bluetooth:/var/lib/bluetooth:ro
        environment:
          - BLE_ADDRESS=AA:BB:CC:DD:EE:FF

**MeshMonitor:** `Dashboard → Sources → Add Source` → тип `meshtastic_tcp` → адрес `ble-bridge` → порт `4403`.

**Когда использовать:** T114 без кабеля, MeshMonitor в одной сети.

---

### Сценарий 4. BLE → MQTT

    T114 → BLE → BLE Bridge (TCP:4403) → MeshMonitor → Virtual Node (4404) ← MQTT Proxy → Embedded Broker

**T114:** MQTT Client Proxy + Uplink/Downlink.
**Компьютер:** два контейнера.

    # docker-compose.yml
    services:
      ble-bridge:
        image: ghcr.io/yeraze/meshtastic-ble-bridge:latest
        container_name: meshtastic-ble-bridge
        privileged: true
        ports:
          - "4403:4403"
        restart: unless-stopped
        volumes:
          - /var/run/dbus:/var/run/dbus
          - /var/lib/bluetooth:/var/lib/bluetooth:ro
        environment:
          - BLE_ADDRESS=AA:BB:CC:DD:EE:FF

      mqtt-proxy:
        image: ghcr.io/ln4cy/mqtt-proxy:latest
        container_name: mqtt-proxy
        restart: unless-stopped
        depends_on:
          - ble-bridge
        environment:
          - INTERFACE_TYPE=tcp
          - TCP_HOST=ble-bridge
          - TCP_PORT=4403

**MeshMonitor:** включить Virtual Node (порт `4404`), убедиться, что Embedded MQTT Broker добавлен как источник.

**Когда использовать:** MeshMonitor удалён, T114 без кабеля.

---

## 7. Шаг 5 — Запуск и проверка

    docker compose up -d
    docker compose logs -f

| Что проверяем | Команда |
|---|---|
| TCP (сценарии 1, 3) | `nc -zv localhost 4403` |
| MQTT (сценарии 2, 4) | `mosquitto_sub -h localhost -p 1883 -t 'msh/#' -v` |

Если пошли пакеты — данные от T114 доходят до MeshMonitor.

---

## 8. Приложения для работы с T114 (GUI)

| Приложение | Платформа | Ссылка на скачивание | Тип подключения | Краткая настройка |
|:---|:---|:---|:---|:---|
| **Meshtastic Desktop** | Windows / macOS / Linux | [GitHub Releases](https://github.com/meshtastic/Meshtastic-Android/releases) (артефакты `desktop`) | USB Serial, BLE | Скачать `.msi`/`.dmg`/`.AppImage`, запустить, подключить T114 по USB, выбрать порт |
| **MeshMonitor Desktop** | Windows / macOS | [meshmonitor.org/configuration/desktop](https://meshmonitor.org/configuration/desktop) | TCP (через Serial Bridge) | Скачать `.exe`/`.dmg`, установить. Указать IP ноды (не USB). Для USB нужен Serial Bridge |
| **LoRaChat** | Windows / macOS / Linux / Android | [GitHub - 404Nick/LoRaChat-Meshtastic-Desktop-Companion](https://github.com/404Nick/LoRaChat-Meshtastic-Desktop-Companion) | USB Serial (desktop), BLE (Android) | Внизу окна выбрать COM/tty порт, нажать **Listen** |
| **mesh-client** | Windows / macOS / Linux | [GitHub - Colorado-Mesh/mesh-client](https://github.com/Colorado-Mesh/mesh-client) | USB Serial, BLE, TCP, MQTT | Вкладка **Connection**, выбрать тип подключения (Serial для USB), указать порт |
| **GTK Meshtastic Client** | Linux (Debian/Ubuntu/Arch) | Debian: `apt install gtk-meshtastic-client` / Arch AUR: [gtk-meshtastic-client](https://aur.archlinux.org/packages/gtk-meshtastic-client) | Serial, Bluetooth, TCP | Запустить, выбрать порт `/dev/ttyUSB0` или `COM3`, подключиться |
| **Веб-клиент** | Любая (Chrome/Edge) | [client.meshtastic.org](https://client.meshtastic.org) | WebSerial (USB) | Только Chrome/Edge. **New Connection → Serial**, выбрать порт T114 |