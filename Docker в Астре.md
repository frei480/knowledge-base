# Docker

[Docker](https://docs.docker.com/get-started/docker-overview/) — это одновременно платформа и технология для контейнеризации. Она позволяет создавать контейнеры и управлять ими для развертывания и доставки кода на целевую систему.

Приложение, развернутое с помощью Docker, будет работать одинаково в любой системе, где он установлен. Для этого требуется создавать контейнер, в котором будет содержатся весь код приложения с зависимостями и среда выполнения. При запуске Docker изолирует приложения друг от друга и от хостовой системы, что обеспечивает высокий уровень безопасности и предотвращает конфликты зависимостей.

Основные пакеты

|Пакет|Версия|Репозиторий|Описание|
|---|---|---|---|
|docker.io|25.0.5.astra2+ci3|installation|Среда выполнения|

Дополнительные пакеты

| Пакет             | Версия            | Репозиторий  | Описание                                                                |
| ----------------- | ----------------- | ------------ | ----------------------------------------------------------------------- |
| docker-compose    | 1.29.2-3+b1       | installation | Инструмент для определения и управления многоконтейнерными приложениями |
| docker-compose-v2 | 25.0.5.astra2+ci3 | installation | Инструмент для определения и управления многоконтейнерными приложениями |
| docker-buildx     | 25.0.5.astra2+ci3 | installation | Cборка Docker-образов                                                   |

## Установка

Docker в Astra Linux Special Edition 1.8 поддерживает два режима работы:

- привилегированный режим - выполнение службы контейнеризации docker с правами суперпользователя;
    
- непривилегированный режим (_рекомендуемый_) - выполнение службы контейнеризации docker в пользовательском пространстве имён.

    При использовании этого режима:
    - служба контейнеризации работает как суперпользователь только с точки зрения приложения в контейнере;
    - служба контейнеризации и контейнеры не получают прав суперпользователя в хостовой ОС;

Подробнее об особенностях и режимах работы можно посмотреть в статье [“Установка и администрирование Docker в Astra Linux Special Edition”](https://wiki.astralinux.ru/pages/viewpage.action?pageId=158601444).

### Привилегированный режим[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id2 "Link to this heading")

Для установки _Docker_ следует перейти в интерфейс командной строки и выполнить:

#### Пункт 1[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id3 "Link to this heading")

- установить пакет:
    

sudo apt install docker.io

#### Пункт 2[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id4 "Link to this heading")

Примечание

После установки можно добавить пользователя в группу docker, что позволит работать с Docker без использования **sudo**.

- для включения пользователя в группу docker выполнить команду:
    

sudo usermod -aG docker <имя_пользователя>

- текущего пользователя можно включить в группу командой:
    

sudo usermod -aG docker $USER

Совет

Для применения действия необходимо выйти из текущей сессии пользователя и зайти повторно.

#### Пункт 3[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id5 "Link to this heading")

- для просмотра информации о версии используется команда:
    

docker --version

#### Пункт 4[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id6 "Link to this heading")

- для проверки работы запустим образ _hello-world_ (минимальный образ, предназначенный для подтверждения корректности установки Docker) командой:
    

docker run hello-world

При успешной работе Docker будет выведено подтверждающее сообщение или строка с информацией о типе ошибки.

Примечание

Для выполнения команды выше требуется подключения к интернету для загрузки тестового образа.

#### Пункт 5[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id7 "Link to this heading")

- общие сведения об установленных образах и запущенных процессах Docker можно посмотреть командой:
    

docker info

### Непривилегированный (rootless) режим[](https://docs.astralinux.ru/latest/guide/virtual/docker/#rootless "Link to this heading")

Важно

Данный режим является рекомендованным к применению. Режим поддерживается в обновлениях Astra Linux, содержащих Docker версии 20.10 и выше. Режим не поддерживается при использовании _hardened_ ядра ОС.

Для использования Docker в rootless режиме следует:

#### Пункт 1[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id8 "Link to this heading")

- установить пакет _rootless-helper-astra_:
    

sudo apt install rootless-helper-astra

Примечание

При выполнении команды выше если ранее не был установлен пакет _docker.io_, он установится автоматически.

#### Пункт 2[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id9 "Link to this heading")

- включить пользовательские службы Docker для пользователей, которые будут использовать контейнеры Docker в rootless режиме:
    

sudo systemctl start rootless-docker@<имя_пользователя>@<метка_безопасности>

Примечание

Где <метка_безопасности> - метка безопасности, с которой должна быть запущена служба, например, нулевая метка **0:0:0:0**.

Метка безопасности при этом не может превышать максимальную метку безопасности пользователя.

Если метка не указана явно, то будет использована метка безопасности текущей пользовательской сессии.

#### Пункт 3[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id10 "Link to this heading")

- при необходимости, разрешить автоматический запуск этих служб:
    

sudo systemctl enable rootless-docker@<имя_пользователя>@<метка_безопасности>

Примечание

Дальнейшее использование Docker пользователями производится с помощью команды _rootlessenv_.

При запуске без параметров, эта команда предоставит пользователю командную оболочку, в которой пользователь сможет выполнять **команды Docker от своего имени**.

При запуске с параметрами, команда _rootlessenv_ попытается интерпретировать указанные параметры как стандартные команды Docker, и **выполнить их в пользовательском окружении**.

**Все данные при этом будут сохраняться в домашнем каталоге пользователя**.

#### Пункт 4[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id11 "Link to this heading")

- для просмотра информации о версии используется команда:
    

rootlessenv docker --version

#### Пункт 5[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id12 "Link to this heading")

- для проверки работы запустим образ _hello-world_ (минимальный образ, предназначенный для подтверждения корректности установки Docker) командой:
    

rootlessenv docker run hello-world

При успешной работе Docker будет выведено подтверждающее сообщение или строка с информацией о типе ошибки.

Примечание

Для выполнения команды выше требуется подключения к интернету для загрузки тестового образа.

#### Пункт 6[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id13 "Link to this heading")

- общие сведения об установленных образах и запущенных процессах Docker можно посмотреть командой:
    

rootlessenv docker info

## Основные команды[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id14 "Link to this heading")

Дополнительная информация по основным командам доступна на странице

- [Основные команды](https://docs.astralinux.ru/latest/guide/virtual/docker/manage/#guide-virtual-docker-manage).
    

## Docker Образы Для разработки[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id15 "Link to this heading")

Для разработки предложены следующие типы докер образов ALSE (подробную информацию см. [по ссылке](https://registry.astralinux.ru/latest/descriptions/local/containers/)):

- Standard – базовая ОС Astra Linux Special Edition и стандартные утилиты из базовой системы.
    
- Multi-service (init) – базовая ОС Astra Linux Special Edition с системой инициализации systemd.
    
- Service – базовая ОС Astra Linux Special Edition с установленным сетевым сервисом.
    
- Dev – базовая ОС Astra Linux Special Edition и окружение для одного из языков программирования.
    

Поддерживаются следующие языки программирования (версии языков соответствуют версиям из ОС):

- C++;
    
- Erlang;
    
- Go;
    
- NodeJS;
    
- OpenJDK;
    
- Perl;
    
- PHP;
    
- Python;
    
- Ruby;
    
- Rust.
    

Поддерживаются следующие сетевые сервисы:

- Apache2;
    
- HaProxy;
    
- MariaDB;
    
- Memcached;
    
- MySQL;
    
- PostgreSQL;
    
- RabbitMQ;
    
- Redis.
    

Образы доступны для скачивания [по ссылке](https://registry.astralinux.ru/browse/library/) в разделе **astra**.

### Привилегированный режим[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id16 "Link to this heading")

[Ограничения работы Docker в привилегированном режиме](https://docs.astralinux.ru/latest/guide/virtual/docker/restrictions/#guide-virtual-docker-restrictions).

#### Пункт 1[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id17 "Link to this heading")

- для загрузки базового образа Astra Linux Special Edition 1.8 необходимо ввести команду:
    

docker pull registry.astralinux.ru/library/astra/ubi18:latest

#### Пункт 2[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id18 "Link to this heading")

- для запуска базового образа Astra Linux Special Edition 1.8 c доступом к командной оболочке внутри контейнера необходимо выполнить команду:
    

docker run -it registry.astralinux.ru/library/astra/ubi18:latest /bin/bash

#### Пункт 3[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id19 "Link to this heading")

- для загрузки образа с Astra Linux Special Edition 1.8 и окружением _python_ нужно выполнить команду:
    

docker pull registry.astralinux.ru/library/astra/ubi18-python311:latest

#### Пункт 4[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id20 "Link to this heading")

- для запуска образа с _python_ окружением, необходимо выполнить команду:
    

docker run -it registry.astralinux.ru/library/astra/ubi18-python311:latest

### Непривилегированный (rootless) режим[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id21 "Link to this heading")

#### Пункт 1[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id22 "Link to this heading")

- для загрузки базового образа Astra Linux Special Edition 1.8 необходимо ввести команду:
    

rootlessenv docker pull registry.astralinux.ru/library/astra/ubi18:latest

#### Пункт 2[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id23 "Link to this heading")

- для запуска базового образа Astra Linux Special Edition 1.8 c доступом к командной оболочке внутри контейнера необходимо выполнить команду:
    

docker run -it registry.astralinux.ru/library/astra/ubi18:latest /bin/bash

#### Пункт 3[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id24 "Link to this heading")

- для загрузки образа с Astra Linux Special Edition 1.8 и окружением _python_ нужно выполнить команду:
    

rootlessenv docker pull registry.astralinux.ru/library/astra/ubi18-python311:latest

#### Пункт 4[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id25 "Link to this heading")

- для запуска образа с _python_ окружением, необходимо выполнить команду:
    

rootlessenv docker run -it registry.astralinux.ru/library/astra/ubi18-python311:latest

## Создание Собственного образа[](https://docs.astralinux.ru/latest/guide/virtual/docker/#id26 "Link to this heading")

Докер образы представляют собой исполняемый пакет, содержащий все необходимое для запуска приложения: код, среду выполнения, библиотеки, переменные окружения и файлы конфигурации. Использование собственных образов приложений имеет ряд преимуществ при разработке. Ниже предложены два метода создания образов:

1. Создание образа из chroot-окружения (подробнее см. в статье [Создание собственного образа Astra Linux для использования в Docker](https://wiki.astralinux.ru/pages/viewpage.action?pageId=158601444#id-%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0%D0%B8%D0%B0%D0%B4%D0%BC%D0%B8%D0%BD%D0%B8%D1%81%D1%82%D1%80%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5Docker%D0%B2AstraLinuxSpecialEdition-%D0%A1%D0%BE%D0%B7%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%D0%BE%D0%B1%D1%80%D0%B0%D0%B7%D0%B0%D0%B8%D0%B7chroot-%D0%BE%D0%BA%D1%80%D1%83%D0%B6%D0%B5%D0%BD%D0%B8%D1%8F).

2. Создание и модификация образа с помощью докерфайла (подробнее см. в статье [Установка и администрирование Docker в ALSE](https://wiki.astralinux.ru/pages/viewpage.action?pageId=158601444#id-%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0%D0%B8%D0%B0%D0%B4%D0%BC%D0%B8%D0%BD%D0%B8%D1%81%D1%82%D1%80%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5Docker%D0%B2AstraLinuxSpecialEdition-%D0%A1%D0%BE%D0%B7%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%D0%B8%D0%BC%D0%BE%D0%B4%D0%B8%D1%84%D0%B8%D0%BA%D0%B0%D1%86%D0%B8%D1%8F%D0%BE%D0%B1%D1%80%D0%B0%D0%B7%D0%B0%D1%81%D0%BF%D0%BE%D0%BC%D0%BE%D1%89%D1%8C%D1%8E%D0%B4%D0%BE%D0%BA%D0%B5%D1%80%D1%84%D0%B0%D0%B9%D0%BB%D0%B0).

## Docker-compose[](https://docs.astralinux.ru/latest/guide/virtual/docker/#docker-compose "Link to this heading")

Устарел, но сохраняется для обратной совместимости.

Описание работы с docker-compose описано на странице:

- [Docker-compose](https://docs.astralinux.ru/latest/guide/virtual/docker/compose/#guide-virtual-docker-compose).
    

## Docker-compose-v2[](https://docs.astralinux.ru/latest/guide/virtual/docker/#docker-compose-v2 "Link to this heading")

Современная, более быстрая и функциональная версия docker-compose.

Описание работы с docker-compose-v2 описано на странице:

- [Docker-compose-v2](https://docs.astralinux.ru/latest/guide/virtual/docker/compose2/#guide-virtual-docker-compose2).