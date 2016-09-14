# WebRTC handbook.

## Введение

WebRTC это набор спецификаций, описывающих трансляцию аудио\видео и сообщений (data streams) в режиме реального времени. Основные действующие лица:

* WebRTC браузер, он же WebRTC User Agent (WebRTC UA) - это приложение, которое умеет WebRTC на уровне протокола и реализует правильное Javascript API
* WebTRC не браузер, известный так же как WebRTC приложение (WebRTC device\WebRTC native application) - это приложение, которое умеет WebRTC на уровне протокола и не предоставляет никакого Javascript API
* WebRTC endpoint - это лубой софт, который умеет WebRTC на уровне протокола
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

                      Рис 1: Browser RTC Trapezoid. Источник: [Overview: Real Time Protocols for Browser-based Applications](https://tools.ietf.org/html/draft-ietf-rtcweb-overview)
```

На данной схеме изображено взаимодействие двух браузеров, но их можно заменить любым WebRTC-совместимым клиентом.

Протокол опиcывает два уровня взаимодействия:

* сигнальный
* медиа

Спецификация не накладывает никаких ограничений на реализацию сигнального уровня, дается только детальное описание сигнальных датаграмм (SDP), но каким образом они будут доставлены от клиента клиенту остается на откуп разработчикам. Медиа уровень служит для передачи данных\аудио\видео преимущественно по udp. Задачей сигнального уровня является наладка и открытие медиа канала, а так же управление этим каналом. Допустим вполне нормальный кейс, когда открывается медиа канал только data streams и после переписки один из клиентов звонит другому, а тот в свою очередь шарит ему свой экран. Так вот для реализации подобного сценария совершенно не нужно создавать новую сессию, а воспользоваться уже существующей, добавив новые медиа потоки. Подобные изменения осуществляются как раз на сигнальном уровне, когда информация о новых медиа-потоках синхронизируется между клиентами посредством сигнального уровня и синхронно применяется.

## Сигнальный уровень.
SDP, ICE ...

## Медиа уровень.
 RTP, audio\video codecs, data streams ...

## Безопасность.

