# Курсовой проект
Клиент-серверное сетевое хранилице.

## Оглавление
1. [Требования](#требования)
   - [Требования для MVP](#требования-для-mvp)
     - [Функциональные требования для MVP](#функциональные-требования-для-mvp)
     - [Нефункциональные требования для MVP](#нефункциональные-требования-для-mvp)
     - [Технические ограничения](#технические-ограничения)
   - [Опциональные требования](#опциональные-требования)
     - [Опциональные функциональные требования](#опциональные-функциональные-требования)
     - [Опциональные нефункциональные требования](#опциональные-нефункциональные-требования)
2. [Вопросы](#вопросы)
3. [Реализация](#реализация)
   - [MVP](#mvp)
     - [Архитектура](#архитектура)

## Требования
### Требования для MVP
#### Функциональные требования для MVP
1. Подключение клиента к серверу
2. Выполняемые операции:
   - Отправка файлов на сервер
   - Скачивание файлов
   - Получение списка файлов на сервере
   - Удаление файлов на сервере
   - Удаление файлов на клиенте
3. Клиентская часть имеет графический интерфейс (толстый клиент) или имеет консольное управление
4. Все будет работать на сокетах
5. Все файлы хранятся в каталоге с именем пользователя. Реализация вложенных каталогов не нужна.

#### Нефункциональные требования для MVP
1. Аутентификация
2. Авторизация и ролевой доступ
   - Клиенты выполняют действия только над своими файлами

#### Технические ограничения
1. Взаимодействия между клиентом и сервером на сокетах
2. При выборе графического интерфейса для клиента он должен быть реализован на JavaFX

### Опциональные требования
#### Опциональные функциональные требования
1. Докачка файлов в случае остановки передачи или сбоя
2. Sharing файлов. Пользователи должны иметь возможность передачи прав на свои файлы другим пользователям:
   - На удаление на сервере
   - На скачивание
   - На получение списка
   - На запись новых файлов на сервер
3. Проверка целостности данных

#### Опциональные нефункциональные требования
1. Шифрование

## Вопросы
1. Как вы собираетесь передавать файлы? (поток байтов/свой протокол?, сериализация)
2. Надо ли передавать что-то кроме файлов? Как передавать по одному? каналу и файлы, и сообщения/команды?
3. Нужна ли база данных?

## Реализация

Планируется микросервисная архитектура. Микросервисная архитектура не подходит для решения задач подобного плана (объем и тип решения), т.к. требует временных вложений больше, чем сама задача. Но она выбрана в учебных целях.

В MVP будет реализован минимальный функционал с ограниченными или отсутствующими инраструктурными решениями или сервисами. При наличии времени, функциональнал будет расширяться. Шаги расширения описаны после раздела MVP.

*Архитектура не является финальной и будет уточняться по мере необходимости.*

### MVP
#### Архитектура
##### Модули системы

Мы имеем один домен, один bounded context.
Функционал сервера решено разделить на 4 микросервиса:
1. FileFetcher. Ответственнен за передачу клиенту файла и списка файлов на сервере.
2. FileCreator. Ответственен за создание файлов на сервере.
3. FileDeletor. Ответственен за удаление файлов с сервера.
4. AuthService. Ответственен за аутентификацию клиентов.

На реальной задаче микросервисы 1-3 были бы объединены в единый, здесь же разделение обусловлено целями обучения.

Файлы будут храниться в HDFS. HDFS отлично подходит для задач управления файлами. Альтерантивы не рассматривались в виду отсутствия времени.

Персональные данные будут храниться в специализированном ПО. В данный момент идет выбор между HashiCorp Vault и KeyCloak. Выбор зависит от способа аутентификации, которая так же еще не определена.

![PlantUML model](http://www.plantuml.com/plantuml/png/5Oqx4eCm30Nxd28vW56VYgi8Is1dX63q8PFR1rTltXQximTQv2wFLzSgIPmcpnPn-Xl0y3jjFLekwMprq62DgHux-C96FszymGSlB6iNBWsLfEi6AsEaCPo2EkI5Ecw6Umds03ggmqrXlEaSyPcLxlq3)

##### Технологический стек MVP:
1. Java 1.8
2. JUnit
3. Jacoco
4. Maven5.
5. Slf4j, logback
6. Netty
6. Sonarqube
7. HDFS
8. HashiCorp Vault \ KeyCloak

##### Интеграция клиентской и серверной части

Взаимодействие между клиентом и серверов будет осуществляться по TCP соединению. Реализация планируется средствами Netty и NIO. Для обеспечения stateless реализации сервера каждый клиент будет держать одно соединение к серверной части. Тут есть риски. Т.к. передавать необходимо и большие файлы и управлящие команды с метаданными, то реализация единичного потока может потребовать парсинга всех входящих обращений для анализа, является ли оно командой (например, старт\стоп передачи файла) или частью файла. Более того, текстовое представление команд может потребовать использования BASE64 кодирования для записи потока байт файла в текстовое сообщение, что увеличит нагрузку на 30%. С другой стороны, поддержание двух соединений между клиентом и сервером предполагает хранение состоянии на серверной части, что ухудшит масштабируемость. Необходим дополнительный ресерч инструментов реализации - NIO\Netty, чтобы понять, возможна ли реализация в один поток.

##### Реализация клиентской части

Клиентская часть будет управляться через консольный ввод.

##### Релизация аутентификации

Клиентское приложение будет аутентифицироваться через AuthService. Т.к. архитектура предполагает несколько микросервисов, для исключения необходимости аутентификации при каждом обращении требуется реализация SSO. В качестве решения отлично подойдет использование JWT.
JWT предполагает обеспечение защиты токена от кражи при передаче и хранении. Этот функционал не фходит в MVP.

##### Ограничения и политика решения
Разрабатываемое приложение максимально близко соответствует [12 факторам.](https://12factor.net)

1. Каждый микросервис имеет свой репозиторий.
2. Зависимости управляются сборщиками
3. Конфигурация на уровне environment (не входит в MVP)
4. HDFS и SSO менеджер считаются подключенными ресурсами
5. Приложение предполагает автоматизированную сборку и тестирование (не входит в MVP)
6. Все процессы stateless между транзакциями
7. Порты будут экспортированы при помощи докера (не входит в MVP)
8. Отсутствие многопоточности. Обработка паралельной нагрузки предполагается за счет масштабирования процессов, не потоков.
9. Отсутствие хранимого состояния предполагает утилизируемость всех сервисов.
10. Журнилирование будет обрабатываться как потокок событий и не решаться на уровне приложения (не входит в MVP)

Решение будет покрыто тестами не менее, чем на 60%.

MVP не будет обеспечивать service discovery и балансировку запросов. Сетевые адреса будут вынесены в environment variables.

MVP в отношении инфраструктурных сервисов:
- будет обеспечивать журналирование в файлы на серверах
- не будет обеспечивать аудит
- не будет обеспечивать мониторинг
- не будет обеспечивать отказоусточивать
- будет разворачиваться "as is" в качестве процессов-демонов (за исключение клиентского приложения). Использование контейнеров и средств управления контейнерами в MVP не предусмотрено.
