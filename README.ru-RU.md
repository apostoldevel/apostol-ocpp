[![en](https://img.shields.io/badge/lang-en-green.svg)](https://github.com/apostoldevel/ocpp-cs/blob/master/README.md)

![image](https://user-images.githubusercontent.com/91010417/230783150-ea57f6c4-ba8a-43cf-a033-ef5ffa3a19ff.png)

# OCPP Central System

**OCPP Central System** - Центральная система и эмулятор зарядных станций, исходные коды на C++.

Реализовано на базе [Apostol](https://github.com/ufocomp/apostol).

Описание
-
**OCPP Central System** — это и готовое решение, которое вы легко можете интегрировать в ваш проект, а так же набор инструментов для создания приложений работающий по протоколу OCPP. 

Данное решение можно использовать для:
- Разработки или интеграции центральной системы зарядных станций;
- Эмулятора зарядных станций;
- Разработки прошивки зарядных станций.

OCPP
-
Open Charge Point Protocol [OCPP](http://ocppforum.net) — это протокол связи между зарядными станциями ("зарядными точками") и единой управляющей системой ("центральной системой").

**OCPP Central System** поддерживает все команды для версий протокола OCPP (1.5 и 1.6).

Версия 1.5 использует SOAP поверх HTTP в качестве RPC/транспортного протокола. Версия 1.6 использует SOAP и JSON поверх протокола WebSocket.

API
-
Мы используем OpenAPI для взаимодействия с `Центральной системой (CS)`. Вы можете напрямую открыть Swagger UI по [http://cs.ocpp-css.com/docs](http://cs.ocpp-css.com/docs).

Кроме того, вы можете использовать любой клиент OpenAPI, чтобы импортировать файл [api.yaml](https://github.com/apostoldevel/ocpp-cs/blob/master/www/docs/api.yaml) из нашего репозитория.

Демонстрация
-
Вы можете подключить свою зарядную станцию к демонстрационной версии `Центральной системы`.

---
Адреса подключения:
- WebSocket: ws://ws.ocpp-css.com/ocpp
- SOAP: http://soap.ocpp-css.com/Ocpp
---

Для управления зарядной станцией используйте веб-оболочку по адресу:
- [http://cs.ocpp-css.com](http://cs.ocpp-css.com)

Авторизация:
```
username: demo
password: demo
```

RFID-карта:
```
idTag: demo
```

Сборка и установка
-
Самый простой способ установки `Центральной системы` - в виде контейнера.

### Docker hub

Вы можете получить готовый образ на docker hub:
```shell
docker pull apostoldevel/cs
```
Запустить контейнер:
```shell
docker run -p 9220:9220 --network host --env WEBHOOK_URL=http://localhost:8080/api/v1/ocpp --rm --name cs apostoldevel/cs
```

Сборка образа контейнера
-
Образ контейнера можно собрать самостоятельно с настройками под доменное имя или IP-адрес вашего сервера.

Клонируйте репозиторий:
```shell
git clone https://github.com/apostoldevel/ocpp-cs.git && cd ocpp-cs
```

Выполните настройки в соответствии с Вашими требованиями:

- Отредактируйте файл `./docker/default.conf`, обратите особое внимание разделу `[webhook]`;
- Отредактируйте файл `./docker/www/config.js`, укажите доменное имя или IP-адрес сервера;
- Отредактируйте файл `./docker/conf/sites/default.json`, добавьте IP-адрес вашего сервера:

  Например, IP-адрес вашего сервера `192.168.1.100` или DNS-имя `cs.example.com`.
  ```json
  {
    "hosts": ["cs.example.com", "cs.example.com:9220", "192.168.1.100:9220", "localhost:9220"]
  }
  ```

Создайте и запустите контейнер одной командой:
```shell
docker compose up
```

Web приложение
-
После запуска контейнера `Центральная система` будет доступна по адресу http://localhost:9220 в вашем браузере.

Swagger UI также будет доступен по адресу http://localhost:9220/docs/ в вашем браузере.

###### Запуск из контейнера не потребует авторизации.

Интеграция
-

Существует несколько способов интеграции `Центральной системы` с Вашим проектом. Самый простой способ через `Webhook endpoint`.

В файле настроек `Центральной системы`, а именно `./docker/default.conf` при сборке контейнера или `/etс/cs/cs.conf` внутри контейнера, имеется раздел `[webhook]`.

```text
## Webhook configuration
[webhook]
## default: false
enable=false

## Webhook endpoint URL
url=http://localhost:8080/api/v1/ocpp

## Authorization schema value: Off | Basic | Bearer
authorization=Basic
## Username for basic schema
username=ocpp
## Password for basic schema
password=ocpp
## Token for Bearer schema
token=
```

В этом разделе Вы можете указать `endpoint URL` на который `Центральная система` будет направлять пакеты поступающие от зарядных станций. 

В частности это десять команд из раздела **4. Operations Initiated by Charge Point** спецификации OCPP v1.6.:

- 4.1. Authorize
- 4.2. Boot Notification
- 4.3. Data Transfer
- 4.4. Diagnostics Status Notification
- 4.5. Firmware Status Notification
- 4.6. Heartbeat
- 4.7. Meter Values
- 4.8. Start Transaction
- 4.9. Status Notification
- 4.10. Stop Transaction

Дополнительно можно настроить параметры авторизации на стороне Вашего сервера, который будет принимать запросы от `Центральной системы`.    

Данные от `Центральной системы` будут в следующем JSON-формате:
```json lines
{
  "identity": "string",
  "uniqueId": "string",
  "action": "string",
  "payload": "JSON Object",
  "account": "string"
}
```
Где:
  - `identity`: Обязательный. Идентификатор зарядной станции;
  - `uniqueId`: Обязательный. Идентификатор пакета данных (запроса);
  - `action`: Обязательный. Имя действия;
  - `payload`: Обязательный. Полезная нагрузка - данные от зарядной станции;
  - `account`: Необязательный. Идентификатор учетной записи пользователя в Вашей системе.

###### Благодаря `account` можно связать зарядную станцию с пользователем систему, если бизнес логика проекта это предусматривает. Обычно в настройках зарядной станции указывается адрес для подключения к центральной системе в формате `ws://webServices/ocpp/EM-A0000001`, если к идентификатору зарядной станции `EM-A0000001` добавить дополнительное значение, например: `/EM-A0000001/AC0001`, то `AC0001` это и будет идентификатор учётной записи пользователя.     

`Центральная система` будет ожидать ответ от Вашей информационной системы в таком же JSON-формате. Значения полей (`identity`, `uniqueId`, `action`) должны быть заполнены значениями из входящего запроса, но в `payload` должны находится данные ответа на `action` в формате спецификации протокола OCPP. Данные из `payload` будут отправлены зарядной станции в качестве ответа на поступивший от неё запрос.  

Пример запроса:
```json lines
{
  "identity": "EM-A0000001",
  "uniqueId": "25cf07c9ae20a0566d1043587b5790a6",
  "action": "BootNotification",
  "payload": {
    "firmwareVersion": "1.0.0.1",
    "chargePointModel": "CP_EM",
    "chargePointVendor": "Apostol",
    "chargePointSerialNumber": "202206040001"
  },
  "account": "AC0001"
}
```

Пример ответа:
```json lines
{
  "identity": "EM-A0000001",
  "uniqueId": "25cf07c9ae20a0566d1043587b5790a6",
  "action": "BootNotification",
  "payload": {
    "status": "Accepted",
    "interval": 600,
    "currentTime": "2024-10-22T23:08:58.205Z"
  }
}
```
### PostgreSQL
Ещё один способ интеграции `Центральной системы` - через прямое подключение к базе данных PostgreSQL.

Для интеграции с вашей системой через базу данных PostgreSQL, вам нужно будет создать схему `ocpp` и несколько функций в базе данных:
  - ocpp.Parse;
  - ocpp.ParseXML;
  - ocpp.ChargePointList;
  - ocpp.TransactionList;
  - ocpp.ReservationList;
  - ocpp.JSONToSOAP;
  - ocpp.SOAPToJSON.

##### Параметры функций могут быть предоставлены разработчикам по запросу в нашу службу поддержки. 

При коммуникации с зарядными станциями `Центральная система` будет вызывать эти функции и передавать данные от зарядных станций в JSON-формате непосредственно в базу данных. Разбор данных и реализация бизнес-логики будет выполняться в PostgreSQL на языке программирования PL/pgSQL.

#### Внимание: Версия из репозитория настроена на интеграцию с базой данных.

Чтобы собрать `Центральную систему` без интеграции с базой данных, измените следующие настройки в файле [CMakeLists.txt](https://github.com/apostoldevel/ocpp-cs/blob/master/CMakeLists.txt):
```
WITH_AUTHORIZATION OFF
WITH_POSTGRESQL OFF
```

Эмулятор зарядных станций
-

`Центральная система` может создавать эмуляторы зарядных станций. Очень полезно в процессе разработки.

Настройки для эмуляторов находятся в папке `/etc/cs/cp` (при сборке контейнера в `./docker/conf/cp`). Внутри `cp` находятся папки с настройками эмуляторов в виде файла `configuration.json` который содержит в себе конфигурацию зарядной станции-эмулятора.  

Включить режим эмуляции можно в файле настроек `Центральной системы` - `/etc/cs/cs.conf` (при сборке контейнера в `./docker/conf/default.conf`):
```text
## Process: Charging point emulator
[process/ChargePoint]
## default: false
enable=true
```

Если в настройках отключить `master` процесс, то приложение будет работать только в режиме эмулятора зарядных станций (`Центральная система` будет отключена). 

```text
## Create master process
## Master process run processes:
## - worker (if count not equal 0)
## - helper (if value equal true)
## default: true
master=false
```

Сборка из исходных кодов
-
Вы можете самостоятельно собрать приложение из исходных кодов.

### Подготовка к сборке

Для сборки потребуется установка следующих пакетов:

1. Компилятор C++;
1. [CMake](https://cmake.org);
1. Библиотека [libpq-dev](https://www.postgresql.org/download/) (библиотеки и заголовки для разработки фронтенда на языке C);
1. Библиотека [postgresql-server-dev-all](https://www.postgresql.org/download/) (библиотеки и заголовки для разработки бэкенда на языке C).

Чтобы установить компилятор C++ и необходимые библиотеки в Debian/Ubuntu, выполните:
```
sudo apt-get install build-essential libssl-dev libcurl4-openssl-dev make cmake gcc g++
```

Чтобы установить PostgreSQL, используйте инструкции по [этой](https://www.postgresql.org/download/) ссылке.

###### Подробное описание установки C++, CMake, IDE и других компонентов, необходимых для сборки проекта, не включено в это руководство.

Клонируйте репозиторий:
```shell
git clone https://github.com/apostoldevel/ocpp-cs.git && cd ocpp-cs
```

Выполните настройки в соответствии с Вашими требованиями:

- Отредактируйте файл `./conf/default.conf`, обратите особое внимание разделу `[webhook]`;
- Отредактируйте файл `./www/config.js`, укажите доменное имя или IP-адрес сервера;
- Отредактируйте файл `./conf/sites/default.json`, добавьте IP-адрес вашего сервера:

  Например, IP-адрес вашего сервера `192.168.1.100` или DNS-имя `cs.example.com`.
  ```json
  {
    "hosts": ["cs.example.com", "cs.example.com:9220", "192.168.1.100:9220", "localhost:9220"]
  }
  ```
- Отредактируйте файл `CMakeLists.txt`, отключите режим работы с базой данных и авторизацию:
  ```
  WITH_AUTHORIZATION OFF
  WITH_POSTGRESQL OFF
  ```
  
###### Конфигурация CMake:
```
/// Установить как root.
/// Отключить для локальной установки.
/// По умолчанию: ON 
INSTALL_AS_ROOT = {ON | OFF}

/// Сборка с авторизацией OAuth 2.0 для промышленной версии.
/// Отключить для режима эмулятора.
/// По умолчанию: ON
WITH_AUTHORIZATION = {ON | OFF}

/// Сборка с поддержкой PostgreSQL для промышленной версии.
/// Отключить для режима эмулятора.
/// По умолчанию: ON
WITH_POSTGRESQL = {ON | OFF}
```

Сборка и установка
-

Находясь в папке в исходными кодами `ocpp-cs` выполните следующие команды:

Настройка:
```shell
./configure
```

Сборка с установкой:
```shell
sudo ./deploy --install
```
или
```shell
cd cmake-build-release
make
sudo make install
```

По умолчанию, **cs** будет установлен в:
```
/usr/sbin
```

Файл конфигурации и необходимые файлы для работы, в зависимости от варианта установки, будут расположены в:
```
/etc/cs
или
~/cs
```

Запуск
-
###### Если `INSTALL_AS_ROOT` установлен в `ON`.

`cs` — это системная служба Linux (демон).

Для управления `cs` используйте стандартные команды управления службами.

Чтобы запустить, выполните:
```shell
sudo systemctl start cs
```

Чтобы проверить статус, выполните:
```shell
sudo systemctl status cs
```

Результат должен быть **примерно** таким:
```
● cs.service - OCPP Central System
     Loaded: loaded (/etc/systemd/system/cs.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2024-09-25 20:22:15 MSK; 3 weeks 6 days ago
    Process: 1195974 ExecStartPre=/usr/bin/rm -f /run/cs.pid (code=exited, status=0/SUCCESS)
    Process: 1195975 ExecStartPre=/usr/sbin/cs -t (code=exited, status=0/SUCCESS)
    Process: 1195976 ExecStart=/usr/sbin/cs (code=exited, status=0/SUCCESS)
   Main PID: 1195977 (cs)
      Tasks: 3 (limit: 2347)
     Memory: 7.9M
        CPU: 35min 23.394s
     CGroup: /system.slice/cs.service
             ├─1195977 cs: master process /usr/sbin/cs
             ├─1195978 cs: worker process ("ocpp central system service")
             └─1195979 cs: charging point emulator process
```

Управление
-

Управлять `cs` можно с помощью сигналов.
Номер главного процесса по умолчанию записывается в файл `/run/cs.pid`.
Изменить имя этого файла можно при конфигурации сборки или же в `cs.conf` секция `[daemon]` ключ `pid`.

Главный процесс поддерживает следующие сигналы:

|Сигнал   |Действие          |
|---------|------------------|
|TERM, INT|быстрое завершение|
|QUIT     |плавное завершение|
|HUP	  |изменение конфигурации, запуск новых рабочих процессов с новой конфигурацией, плавное завершение старых рабочих процессов|
|WINCH    |плавное завершение рабочих процессов|	

Управлять рабочими процессами по отдельности не нужно. Тем не менее они тоже поддерживают некоторые сигналы:

|Сигнал   |Действие          |
|---------|------------------|
|TERM, INT|быстрое завершение|
|QUIT	  |плавное завершение|
