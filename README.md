# WebRTC handbook.

## Введение

WebRTC это набор спецификаций, описывающих трансляцию аудио\видео и сообщений (data streams) в режиме реального времени. Каждый раз обращаться к WebRTC как к набору спецификаций не слишком удобно, поэтому дальше в тексте я либо буду явно писать WebRTC либо просто протокол, в зависимости от контекста.

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
Рис 1: Browser RTC Trapezoid (не берусь это переводить). Источник: [Overview: Real Time Protocols for Browser-based Applications](https://tools.ietf.org/html/draft-ietf-rtcweb-overview)

На данной схеме изображено взаимодействие двух браузеров, но их можно заменить любым WebRTC-совместимым клиентом.

Протокол опиcывает два уровня взаимодействия:

* сигнальный
* медиа

Спецификация не накладывает никаких ограничений на реализацию сигнального уровня, дается только детальное описание сигнальных датаграмм (SDP) и их последовательность, но каким образом они будут доставлены от клиента клиенту остается на откуп разработчикам. Медиа уровень служит для передачи данных\аудио\видео преимущественно по udp. Задачей сигнального уровня является наладка и открытие медиа канала, а так же управление этим каналом. Допустим вполне нормальный кейс, когда открывается медиа канал только data streams и после переписки один из клиентов звонит другому, а тот в свою очередь шарит ему свой экран. Так вот для реализации подобного сценария совершенно не нужно создавать новую сессию, а воспользоваться уже существующей, добавив новые медиа потоки. Подобные изменения осуществляются как раз на сигнальном уровне, когда информация о новых медиа-потоках синхронизируется между клиентами и применяется.

Описание протокола разбито на две части:

* описание протокола (IETF спецификации)
* описание Javascript API (W3C спецификации)

Обмен потоковыми данными в реальном времени через интернет существует уже достаточно давно, но т.к. раньше это решалось дорогим железом и проприетарным софтом, не так много компаний могли этим заниматься и цены на этом рынке были крайне высоки. Поэтому и возникла необходимость в разработке открытого стандарта, позволяющего любому желающему реализовать приложение с помощью которого можно обмениваться потоковыми данными, что делает технологию доступней, как бизнесу, так и потребителям. Плюс для каждой из необходимых платформ достаточно один раз написать WebRTC-клиeнта и любое приложение на этой платформе может им пользоваться без изменений, здесь, правда, нельзя забывать, что для установления WebRTC сессии необходим сигнальный севрер, однако, если у вас под рукой есть приложение реализующее Offer\Answer модель взаимодействия посредством SDP датаграмм (SIP-сервер, например) - вы легко можете его использовать в качестве сингального сервера.

В целом предназначение WebRTC упростить разработку приложений, работающих с потоковыми данными, особенно при клиент-серверном взаимодействии, а это онлайн-кинотеатры, мессенджеры с возможностью видео звонков, приложения для организации конференц-связи, IP-телефония и т.д. Протокол имеет peet-to-peer природу, где каждая из сторон имеет равные права и нет разницы, что за приложение на другой стороне соединения: другой браузер, медиа сервер, медиа гейтвей или мобильный клиент. Это означает, что при звонке от одного клиента к другому им нужен только сигнальный сервер для обмена SDP датаграммами, однако, групповые звонки, [судя по спецификации](https://tools.ietf.org/html/draft-ietf-rtcweb-overview-15#page-8) без медиа-сервера не осуществишь, т.к. кто-то должен собирать все видео-потоки, объединять их в один и раздавать всем клиентам.


3.  Architecture and Functionality groups

   The model of real-time support for browser-based applications does
   not assume that the browser will contain all the functions that need
   to be performed in order to have a function such as a telephone or a
   video conferencing unit; the vision is that the browser will have the
   functions that are needed for a Web application, working in
   conjunction with its backend servers, to implement these functions.

3. Архитектурные и функциональные группы

Модель взаимодействия в реальном времени для браузерных приложений не подразумевает, что браузер будет обладать всей функциональностью, необходимой для реализации телефонии или видео-конференций; цель в том чтобы сделать возможной реализацию подобного функционала с помощью совместной работы web-приложения и бекенда. Этим так же объясняется peer-to-peer природа протокола. Однако, функциональность, которой обладает WebRTC совместимый браузер, позволяет двум браузерам обмениваться потоковыми данными напрямую, с минимальной поддержкой со стороны бекенда (только сигнальный уровень).

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

<<-RFC

   The functionality groups that are needed in the browser can be
   specified, more or less from the bottom up, as:

   o  Data transport: TCP, UDP and the means to securely set up
      connections between entities, as well as the functions for
      deciding when to send data: Congestion management, bandwidth
      estimation and so on.

   o  Data framing: RTP and other data formats that serve as containers,
      and their functions for data confidentiality and integrity.

   o  Data formats: Codec specifications, format specifications and
      functionality specifications for the data passed between systems.
      Audio and video codecs, as well as formats for data and document
      sharing, belong in this category.  In order to make use of data
      formats, a way to describe them, a session description, is needed.

   o  Connection management: Setting up connections, agreeing on data
      formats, changing data formats during the duration of a call; SIP
      and Jingle/XMPP belong in this category.

   o  Presentation and control: What needs to happen in order to ensure
      that interactions behave in a non-surprising manner.  This can
      include floor control, screen layout, voice activated image
      switching and other such functions - where part of the system
      require the cooperation between parties.  XCON and Cisco/
      Tandberg's TIP were some attempts at specifying this kind of
      functionality; many applications have been built without
      standardized interfaces to these functions.

   o  Local system support functions: These are things that need not be
      specified uniformly, because each participant may choose to do
      these in a way of the participant's choosing, without affecting
      the bits on the wire in a way that others have to be cognizant of.
      Examples in this category include echo cancellation (some forms of
      it), local authentication and authorization mechanisms, OS access
      control and the ability to do local recording of conversations.

  One can think of the three first groups as forming a "media transport
   infrastructure", and of the three last groups as forming a "media
   service".  In many contexts, it makes sense to use a common
   specification for the media transport infrastructure, which can be
   embedded in browsers and accessed using standard interfaces, and "let
   a thousand flowers bloom" in the "media service" layer; to achieve
   interoperable services, however, at least the first five of the six
   groups need to be specified.

4.  Data transport

   Data transport refers to the sending and receiving of data over the
   network interfaces, the choice of network-layer addresses at each end
   of the communication, and the interaction with any intermediate
   entities that handle the data, but do not modify it (such as TURN
   relays).

   It includes necessary functions for congestion control: When not to
   send data.

   WebRTC endpoints MUST implement the transport protocols described in
   [I-D.ietf-rtcweb-transports].

5.  Data framing and securing

   The format for media transport is RTP [RFC3550].  Implementation of
   SRTP [RFC3711] is REQUIRED for all implementations.

   The detailed considerations for usage of functions from RTP and SRTP
   are given in [I-D.ietf-rtcweb-rtp-usage].  The security
   considerations for the WebRTC use case are in
   [I-D.ietf-rtcweb-security], and the resulting security functions are
   described in [I-D.ietf-rtcweb-security-arch].
   Considerations for the transfer of data that is not in RTP format is
   described in [I-D.ietf-rtcweb-data-channel], and a supporting
   protocol for establishing individual data channels is described in
   [I-D.ietf-rtcweb-data-protocol].  WebRTC endpoints MUST implement
   these two specifications.

   WebRTC endpoints MUST implement [I-D.ietf-rtcweb-rtp-usage],
   [I-D.ietf-rtcweb-security], [I-D.ietf-rtcweb-security-arch], and the
   requirements they include.

6.  Data formats

   The intent of this specification is to allow each communications
   event to use the data formats that are best suited for that
   particular instance, where a format is supported by both sides of the
   connection.  However, a minimum standard is greatly helpful in order
   to ensure that communication can be achieved.  This document
   specifies a minimum baseline that will be supported by all
   implementations of this specification, and leaves further codecs to
   be included at the will of the implementor.

   WebRTC endpoints that support audio and/or video MUST implement the
   codecs and profiles required in [I-D.ietf-rtcweb-audio] and
   [I-D.ietf-rtcweb-video].

7.  Connection management

   The methods, mechanisms and requirements for setting up, negotiating
   and tearing down connections is a large subject, and one where it is
   desirable to have both interoperability and freedom to innovate.

   The following principles apply:

   1.  The WebRTC media negotiations will be capable of representing the
       same SDP offer/answer semantics that are used in SIP [RFC3264],
       in such a way that it is possible to build a signaling gateway
       between SIP and the WebRTC media negotiation.

   2.  It will be possible to gateway between legacy SIP devices that
       support ICE and appropriate RTP / SDP mechanisms, codecs and
       security mechanisms without using a media gateway.  A signaling
       gateway to convert between the signaling on the web side to the
       SIP signaling may be needed.

   3.  When a new codec is specified, and the SDP for the new codec is
       specified in the MMUSIC WG, no other standardization should be
       required for it to be possible to use that in the web browsers.
       Adding new codecs which might have new SDP parameters should not
       change the APIs between the browser and Javascript application.
       As soon as the browsers support the new codecs, old applications
       written before the codecs were specified should automatically be
       able to use the new codecs where appropriate with no changes to
       the JS applications.

   The particular choices made for WebRTC, and their implications for
   the API offered by a browser implementing WebRTC, are described in
   [I-D.ietf-rtcweb-jsep].

   WebRTC browsers MUST implement [I-D.ietf-rtcweb-jsep].

   WebRTC endpoints MUST implement the functions described in that
   document that relate to the network layer (for example Bundle, RTCP-
   mux and Trickle ICE), but do not need to support the API
   functionality described there.

8.  Presentation and control

   The most important part of control is the user's control over the
   browser's interaction with input/output devices and communications
   channels.  It is important that the user have some way of figuring
   out where his audio, video or texting is being sent, for what
   purported reason, and what guarantees are made by the parties that
   form part of this control channel.  This is largely a local function
   between the browser, the underlying operating system and the user
   interface; this is specified in the peer connection API
   [W3C.WD-webrtc-20120209], and the media capture API
   [W3C.WD-mediacapture-streams-20120628].

   WebRTC browsers MUST implement these two specifications.

9.  Local system support functions

   These are characterized by the fact that the quality of these
   functions strongly influence the user experience, but the exact
   algorithm does not need coordination.  In some cases (for instance
   echo cancellation, as described below), the overall system definition
   may need to specify that the overall system needs to have some
   characteristics for which these facilities are useful, without
   requiring them to be implemented a certain way.

   Local functions include echo cancellation, volume control, camera
   management including focus, zoom, pan/tilt controls (if available),
   and more.

   Certain parts of the system SHOULD conform to certain properties, for
   instance:
   o  Echo cancellation should be good enough to achieve the suppression
      of acoustical feedback loops below a perceptually noticeable
      level.

   o  Privacy concerns MUST be satisfied; for instance, if remote
      control of camera is offered, the APIs should be available to let
      the local participant figure out who's controlling the camera, and
      possibly decide to revoke the permission for camera usage.

   o  Automatic gain control, if present, should normalize a speaking
      voice into a reasonable dB range.

   The requirements on WebRTC systems with regard to audio processing
   are found in [I-D.ietf-rtcweb-audio]; the proposed API for control of
   local devices are found in [W3C.WD-mediacapture-streams-20120628].

   WebRTC endpoints MUST implement the processing functions in
   [I-D.ietf-rtcweb-audio].  (Together with the requirement inSection 6,
   this means that WebRTC endpoints MUST implement the whole document.)

10.  IANA Considerations

   This document makes no request of IANA.

   Note to RFC Editor: this section may be removed on publication as an
   RFC.

11.  Security Considerations

   Security of the web-enabled real time communications comes in several
   pieces:

   o  Security of the components: The browsers, and other servers
      involved.  The most target-rich environment here is probably the
      browser; the aim here should be that the introduction of these
      components introduces no additional vulnerability.

   o  Security of the communication channels: It should be easy for a
      participant to reassure himself of the security of his
      communication - by verifying the crypto parameters of the links he
      himself participates in, and to get reassurances from the other
      parties to the communication that they promise that appropriate
      measures are taken.

   o  Security of the partners' identity: verifying that the
      participants are who they say they are (when positive
      identification is appropriate), or that their identity cannot be
      uncovered (when anonymity is a goal of the application).
   The security analysis, and the requirements derived from that
   analysis, is contained in [I-D.ietf-rtcweb-security].

   It is also important to read the security sections of
   [W3C.WD-mediacapture-streams-20120628] and [W3C.WD-webrtc-20120209].

<<-RFC


###todo:
+ [overview](https://tools.ietf.org/html/draft-ietf-rtcweb-overview-15) <- I'm here
* актуализировать ссылки
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
