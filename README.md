# WebRTC handbook.

## Введение

WebRTC это набор спецификаций, описывающих трансляцию аудио\видео и сообщений (data streams) в режиме реального времени. Каждый раз обращаться к WebRTC как к набору спецификаций не слишком удобно, поэтому дальше в тексте я либо буду явно писать WebRTC, спецфикация, или просто протокол, в зависимости от контекста.

Основные действующие лица:

* WebRTC браузер, он же WebRTC User Agent (WebRTC UA) - это приложение, которое умеет WebRTC на уровне протокола и реализует правильное Javascript API
* WebTRC не браузер, известный так же как WebRTC приложение (WebRTC device\WebRTC native application) - это приложение, которое умеет WebRTC на уровне протокола и не предоставляет никакого Javascript API
* WebRTC endpoint - это любой софт, который умеет WebRTC на уровне протокола
* WebRTC-compatible endpoint - это софт, который умеет WebRTC на уровне протокола, но не полностью, т.е. реализует только часть спецификации
* WebRTC gateway - это софт, который транслирует WebRTC трафик WebRTC несовместимым клиентам

Базовая схема взаимодействия:

```

                +-----------+             +-----------+
                |   Web     |             |   Web     |
                |           |  Signaling  |           |
                |           |-------------|           |
                |  Server   |   path      |  Server   |
                |           |             |           |
                +-----------+             +-----------+
                     /                           \
                    /                             \ Application-defined
                   /                               \ over
                  /                                 \ HTTP/Websockets
                 /  Application-defined over         \
                /   HTTP/Websockets                   \
               /                                       \
         +-----------+                           +-----------+
         |JS/HTML/CSS|                           |JS/HTML/CSS|
         +-----------+                           +-----------+
         +-----------+                           +-----------+
         |           |                           |           |
         |           |                           |           |
         |  Browser  | ------------------------- |  Browser  |
         |           |          Media path       |           |
         |           |                           |           |
         +-----------+                           +-----------+

                      
```
Рис 1: Browser RTC Trapezoid (не берусь это переводить). [Источник](https://tools.ietf.org/html/draft-ietf-rtcweb-overview)

На данной схеме изображено взаимодействие двух браузеров, но их можно заменить любым WebRTC-совместимым клиентом.

Протокол опиcывает два уровня взаимодействия:

* сигнальный (signaling path)
* медиа (media path)

Спецификация не накладывает никаких ограничений на реализацию транспорта сигнального уровня, дается только детальное описание сигнальных датаграмм (SDP) и их последовательность, но каким образом они будут доставлены от клиента клиенту остается на откуп разработчикам. Медиа уровень служит для передачи данных\аудио\видео преимущественно по udp. Задачей сигнального уровня является наладка и открытие медиа канала(-ов), а так же управление ими. Допустим вполне нормальный кейс, когда открывается медиа канал (только data streams), после переписки один из клиентов звонит другому, а тот в свою очередь шарит ему свой экран. Так вот для реализации подобного сценария совершенно не нужно создавать новую сессию, а воспользоваться уже существующей, добавив новые медиа потоки. Подобные изменения осуществляются как раз на сигнальном уровне, когда информация о новых медиа-потоках синхронизируется между клиентами и применяется.

Описание протокола разбито на две части:

* описание протокола (IETF спецификации)
* описание Javascript API (W3C спецификации)

Здесь будет рассмотрена только IETF часть спецификации.

Обмен потоковыми данными в реальном времени через интернет существует уже достаточно давно, но т.к. раньше это решалось дорогим железом и проприетарным софтом, не так много компаний могли этим заниматься и цены на этом рынке были крайне высоки. Поэтому и возникла необходимость в разработке открытого стандарта, позволяющего любому желающему реализовать приложение с помощью которого можно обмениваться потоковыми данными, что делает технологию доступней, как бизнесу, так и потребителям. Плюс для каждой из необходимых платформ достаточно один раз написать WebRTC-клиeнта и любое приложение на этой платформе может им пользоваться без изменений, здесь, правда, нельзя забывать, что для установления WebRTC сессии необходим сигнальный севрер, однако, если у вас под рукой есть приложение реализующее Offer\Answer модель взаимодействия посредством SDP датаграмм (SIP-сервер, например) - вы легко можете его использовать в качестве сингального сервера.

В целом предназначение WebRTC упростить и унифицировать разработку приложений, работающих с потоковыми данными, особенно при клиент-серверном взаимодействии, а это онлайн-кинотеатры, мессенджеры с возможностью видео звонков, приложения для организации конференц-связи, IP-телефония и т.д. Протокол имеет peet-to-peer природу, где каждая из сторон имеет равные права и нет разницы, что за приложение на другой стороне соединения: другой браузер, медиа сервер, медиа гейтвей или мобильный клиент. Это означает, что при звонке от одного клиента к другому им нужен только сигнальный сервер для обмена SDP датаграммами, однако, групповые звонки, без медиа-сервера не осуществишь, т.к. кто-то должен собирать все медиа-потоки и проксировать клиентам. [Цитата из спецификации](https://tools.ietf.org/html/draft-ietf-rtcweb-overview-15#page-8):

```
The model of real-time support for browser-based applications does
not assume that the browser will contain all the functions that need
to be performed in order to have a function such as a telephone or a
video conferencing unit; the vision is that the browser will have the
functions that are needed for a Web application, working in
conjunction with its backend servers, to implement these functions.

Модель взаимодействия в реальном времени для браузерных приложений не подразумевает,
что браузер будет обладать всей функциональностью, необходимой для реализации телефонии
или видео-конференций; цель в том чтобы сделать возможной реализацию подобного
функционала с помощью совместной работы web-приложения и бекенда.

```

Этим так же объясняется peer-to-peer природа протокола. Однако, даже та функциональность, которой наделен WebRTC совместимый браузер, позволяет двум браузерам обмениваться потоковыми данными напрямую, с минимальной поддержкой со стороны бекенда (только сигнальный уровень).

```

                        +------------------------+  On-the-wire
                        |                        |  Protocols
                        |      Servers           |--------->
                        |                        |
                        |                        |
                        +------------------------+
                                    ^
                                    |
                                    |
                                    | HTTP/
                                    | Websockets
                                    |
                                    |
                      +----------------------------+
                      |    Javascript/HTML/CSS     |
                      +----------------------------+
                   Other  ^                 ^RTC
                   APIs   |                 |APIs
                      +---|-----------------|------+
                      |   |                 |      |
                      |                 +---------+|
                      |                 | Browser ||  On-the-wire
                      | Browser         | RTC     ||  Protocols
                      |                 | Function|----------->
                      |                 |         ||
                      |                 |         ||
                      |                 +---------+|
                      +---------------------|------+
                                            |
                                            V
                                       Native OS Services


```
Рис 2: WebRTC совместимый браузер.

Браузер является WebRTC совместимым если поддерживает:

* Транспорт данных (data transport) - TCP, UDP и средства для безопасного установления соединений между участниками обмена, а так же функциональность, позволяющая решить когда отправлять данные: контроль переполнения\загруженности, оценка пропускной способности и т.д.
* Кадрирование данных (data framing) - RTP и другие форматы, использующиеся в качестве контейнеров, а так же поддержка их функций по обеспечению конфиденциальности и целостности.
* Фоматы данных - описанный спецификацией набор audio\video кодеков, а так же форматы для передачи данных и шаринга документов. А так же поддерживать механизм, позволяющий описать поддерживаемые форматы другому клиенту при установлении сессии.
* Управление соединением - установка соединения, соглашение по форматам данных, изменение форматов данных для установленного соединения.
* Представление и управление (presentation and control) - механизмы обеспечивающие предсказуемость и наглядность работы с потоковыми данными для клиента, т.е. браузер должен предоставить возможность web-приложению таким образом организовать работу с потоковыми данными, чтобы это было прозрачно для пользователя, чтобы он в любой момент времени понимал, куда\откуда и какие потоковые данные передаются, а так же предоставить возможность контролировать этот процесс, т.е. пользователь может отреагировать на звонок (принять\отклонить) при этом он должен иметь всю необходимую информацию для этого (кто звонит, например), а так же сам инициировать звонок. Так же пользователь, а значит и web-приложение должен иметь возможность самому решить какие потоковые данные передавать, а какие нет.
* Функции поддержки локальных систем (local system support functions) - механизмы локальной аутентификации и авторизации, доступ к системным вызовам ОС, эхо-подавление и возможность локальной записи медиаданных.

Первые три пункта описывают инфраструктуру медиа-транспорта. Остальные три пункта описывают медиа-сервис. Как минимум первые пять пунктов браузер должен поддерживать, чтобы быть WebRTC совместимым. 

## Транспорт данных.

Под транспортом данных понимается отправка\получение данных через сетевые интерфейсы на обоих концах соединения, а так же взаимодействие с промежуточными сервисами, ретранслирующими данные без внесения изменений, например TURN серверы. А так же включает функционал по контролю перегрузки/блокирования канала, для прекращения передачи данных. WebTRC клиенты должны поддерживать работу с транспортными протоколами описанными [здесь](https://tools.ietf.org/html/draft-ietf-rtcweb-transports-15).

## Кадрирование данных и безопасность.

Транспортом для медиа-данных является [RTP](https://tools.ietf.org/html/rfc3550) протокол. Однако, поддержка защищенного аналога [SRTP](https://tools.ietf.org/html/rfc3711) ОБЯЗАТЕЛЬНА для всех WebRTC клиентов.

Более подробно работа с RTP\SRTP в рамках WebRTC описана [здесь](https://tools.ietf.org/html/draft-ietf-rtcweb-rtp-usage-26). Требования по безопасности WebRTC соединений описаны [здесь](https://tools.ietf.org/html/draft-ietf-rtcweb-security-08), описание механизмов, реализующих эти требования, приведены [здесь](https://tools.ietf.org/html/draft-ietf-rtcweb-security-arch-12). Каждый WebRTC клиент (endpoint) должен поддерживать эти две спецификации.

## Форматы данных.

Целью данной спецификации является возможность клиентам договориться по форматам для каждого из медиа-потоков (видео\аудио), согласовав форматы из списка поддерживаемых на каждом из клиентов. При этом, чтобы такая договоренность в целом была достижима, определены минимальные требования по поддержке [аудио-кодеков](https://tools.ietf.org/html/rfc7874) и [видео-кодеков](https://tools.ietf.org/html/rfc7742). При этом разработчики могут добавить поддержку любых дополнительных форматов и кодеков помимо минимума, описанного в спецификации.

## Управление соединением.

Методы, механизмы и требования по установке, поддержке и завершению соединений это обширная область, которая спроектирована таким образом, чтобы обеспечить, как совместимость, так и свободу для инноваций.

Основные принципы:

* Процесс согласования WebRTC соединений должен реализовывать ту же модель SDP запросов\ответов как в [SIP](https://tools.ietf.org/html/rfc3264), в связи с этим SIP сервер легко использовать в качестве сигнального сервера для WebRTC.
* Должна быть возможна поддержка старых (legacy) SIP устройств, которые поддерживают ICE, RTP и SDP механизмы, кодеки и механизмы безопасности без использования медиа-гейтвея. Возможно дополнительно понадобиться наличие сигнального сервиса, для согласования сингальных схем между WEB и SIP клиентами.
* Когда новый кодек определен и SDP для него описан рабочей группой MMUSIC (MMUSIC WG) - никакой дополнительной стандартизации не нужно для использования его в браузерах. Т.е. добавление новых кодеков со своими специфичными SDP параметрами никак не меняет API между браузером и web-приложением. Как только браузер начинает поддерживать новый кодек, старые приложение, написанные до этого момента, должны автоматически получить возможность использовать этот кодек, когда потребуется без изменений в JS коде.

Подробнее о браузероном API и управлении соединениями написано [здесь](https://tools.ietf.org/html/draft-ietf-rtcweb-jsep-15). Все WebRTC браузеры должны реализовывать эту спецификацию. WebRTC ендпоинты должны так же реализовывать механизмы описанные в данной спецификации (Bundle, RTCP-mux, Trickle ICE), кроме Javascript API.

## Представление и управление

Самой важной частью управления является пользовательское управление взаимодействием браузера с IO устройствами и каналами коммуникации. Очень важно, чтобы пользователь понимал куда и кому его аудио\видео\текст транслируются и по какой причине. Данные механизмы описаны в [peer connection API](https://www.w3.org/TR/webrtc) и в [media capture API](https://www.w3.org/TR/mediacapture-streams). Реализация этих спецификаций обязательна для WebRTC браузеров.

## Функции поддержки локальных систем.

Это функции напрямую влияющие на пользовательское взаимодействие с приложением, алгоритмы реализации которых не нуждаются в координации между клиентами. К локальным функциям относятся эхо-подавление, управление громкостью, управление камерой (настройка фокуса, например), различные видео-фильтры и т.д.

Отдельные части системы могут иметь сходные свойства, например:

* Эхо-подавление
* Настройки приватности, например, если разрешен удаленный доступ к камере, то приложение должно уведомить клиента кто использует его камеру и предоставить возможность отменить данное разрешение
* Автоматическая нормализация аудио-потоков

Требования для WebRTC систем по работе с аудио описаны [здесь](https://tools.ietf.org/html/rfc7874). Рекомендуемое API контроля локальных устройств - [здесь](https://www.w3.org/TR/mediacapture-streams/).

## Соображения о безопасности.

Безопасность web-приложений работающих с потоковыми данными в реальном времени можно разделить на несколько частей:

* Безопасность компонент - браузеры, мобильные клиенты и все вовлеченные в процесс сервисы.
* Безопасность коммуникационных каналов.
* Безопасность идентификации - когда каждый из участников является тем за кого себя выдает.

Данная тема раскрывается в следующих спецификациях:

* [security considerations](https://tools.ietf.org/html/draft-ietf-rtcweb-security)
* [security architecture](https://tools.ietf.org/html/draft-ietf-rtcweb-security-arch)
* [peer connection API](https://www.w3.org/TR/webrtc)
* [media capture API](https://www.w3.org/TR/mediacapture-streams)

###todo:
✓ [overview](https://tools.ietf.org/html/draft-ietf-rtcweb-overview-15)
* актуализировать ссылки  <- I'm here
+ [use cases](https://tools.ietf.org/html/rfc7478)
+ [jsep](https://tools.ietf.org/html/draft-ietf-rtcweb-jsep)
- [SDP + ICE](https://tools.ietf.org/html/draft-ietf-rtcweb-sdp-02)
- [RTP](https://tools.ietf.org/html/draft-ietf-rtcweb-rtp-usage)
- [SRTP](https://tools.ietf.org/html/rfc7201)
+ [security considerations](https://tools.ietf.org/html/draft-ietf-rtcweb-security)
- [security architecture](https://tools.ietf.org/html/draft-ietf-rtcweb-security-arch)
- [DSCP, Qos](https://tools.ietf.org/html/draft-ietf-tsvwg-rtcweb-qos)
- [data channels](https://tools.ietf.org/html/draft-ietf-rtcweb-data-channel)
- [data channels establishment](https://tools.ietf.org/html/draft-ietf-rtcweb-data-protocol)
+ [audio codecs](https://tools.ietf.org/html/rfc7874)
- [video codecs](https://tools.ietf.org/html/rfc7742)
