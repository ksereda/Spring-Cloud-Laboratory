# Spring-Cloud-Laboratory

Spring Cloud, Netflix OSS, Cloud Foundry

___

### Вступление

`Cloud Foundry` - система с открытым исходным кодом.

Определение из вики:

    Pivotal CF — платформа для облачных систем, которая выступает в качестве некоего слоя абстракции для виртуальной среды (как виртуализация для виртуализации). 
    Суть в том, чтобы создать унифицированную площадку, на которой можно запускать любые приложения без привязки к конкретному облаку или гипервизору. 
    То есть строительным блоком служит не виртуальная машина, а контейнер приложения.

Платформа Cloud Foundry нацелена на увеличение скорости развертывания приожений и управления ими.
Она оптимизирована для непрерывного развертывания веб-приложений и сервисов.

Существует несколько реализаций Cloud Foundry, каждая из которых основана на открытом коде. 
В нашем примере мы будем запускать и развертывать приложение в Pivotal Web Services (http://run.pivotal.io).
Эта платформа предлагает большой выбор всех функций, предоставляемых компанией Pivotal в системе Pivotal Cloud Foundry.
Она базируется на AWS.
В двух словах что это такое. 
Pivotal Cloud Foundry (PCF) – это готовое коммерческое решение для предприятий, которые:
- нуждаются в платформе, работающей в гибридной среде или в среде с несколькими облаками;
- хотят перейти от использования монолитных архитектур к облачным схемам;
- хотят запускать цифровые сервисы следующего поколения с первого дня использования;
- уже используют Java или Spring для развертывания микросервисов.

В 2014 году был создан некоммерческий фонд Cloud Foundry Foundation (CFF), под руководством организации Linux Foundation, что позволило создать нейтральную площадку для совместного развития, управления и продвижения проекта.

В настоящее время в объединение CFF входит более шестидесяти организаций – лидеров мирового рынка телекоммуникаций и инженерных систем: Cisco, Dell EMC, Hewlett Packard Enterprise, IBM, Pivotal, SAP, VMware, Intel и многих других.

Архитектура Pivotal CF:
Pivotal относится к платформенным облакам, которые базируются на виртуальных машинах. После загрузки .OVA-файла и разворачивания из него VM для vSphere в пакете уже содержатся необходимые для Cloud Foundry сервисы.

С инструментами VMWare вы можете ознакомиться в интеренете.

Архитектурно Pivotal CF строится из так называемых «микросервисов», каждый из которых выполняет ограниченную роль:
- базы данных
- big data
- система обмена сообщениями и т.д.

Сами программы размещаются в изолированных контейнерах со всем необходимым для работы.
Пользователю не придется разбираться с установкой, добавлением необходимых компонентов и драйверов. 
Вместо этого он импортирует новый контейнер в облако и получит готовое решение.

___

### Деплой в облако

Давайте начнем.

1) Зарегистрируйтесь на сайте (www.pivotal.io)
2) Перейдите в личный кабинет
3) Убедитесь что у вас установлен CLI (если нету, тогда установите)
Я использую ОС Ubuntu -> для этого откройте терминал и введите

    
    cf login
   
Затем введите логин и пароль от pivotal

После успешной аутентификации вы увидите Org - вашу организацию, которую вы указывали в личном кабинете и Space - среду.
Т.е. в Cloud Foundry может быть множество организаций, к которым данный юзер имеет доступ.
Также в рамках данной организации может быть несколько сред (например development, staging, integration и т.д.)

Давайте попробуем развернуть приложение 2 способами:

- Вручную (при помощи CLI):

В уже открытой нами консоли пишем

    cf create-service cleardb spark mysql-db-prod
    
где cleardb - это "аналог" сервиса p-mysql (p-mysql недоступен на рынке. При использовании команды
cf create-service p-mysql 100mb mysql-db-prod
вы скорее всего получите ошибку типа:
Service offering 'p-mysql' not found.
Компания pivotal предлагает логическое объяснение:
Service is not available from marketplace. If you are trying with Pivotal Web Services, check with cleardb service offerings.

Более подробно про сервис `mysql` можно прочитать в документации:
https://docs.pivotal.io/p-mysql/2-4/index.html

где spark - это план обслуживания
где mysql-db-prod - придуманное имя для базы

Мы предоставили базу данных из сервиса MySQL по конкретному плану.
База есть, но пока она не привязана к приложению к ней нельзя обратиться.

Добавляя приложение к Cloud Foundry вы предоставляете его в виде `.jar` или `контейнера`.
Затем платформа пытается определить к чему оно относиться (Java, Python, и т.д.)
Файл .jar передается пакету `buildpack` (эти пакеты существуют для всех языков и платформ).
`Java buildpack` определит, что приложение представляет собой выполняемый метод `main`.
Будет задействована самая последняя версия OpenJDK.
Вся эта конфигурация упакована в Linux-контейнер, который затем диспетчер Cloud Foundry развернет у себя в кластере.
Очень важно понимать, что в одной среде Cloud Foundry может выполняться очень много (речь идет о тысячах) контейнеров.

Затем мы должны задать пакет `buildpack` или поместить компонент приложения (типа target/myapp.jar) без его запуска!

    cf push -p target/myapp.jar my-application --random-route --no-start
    
где my-application - имя приложения

Далее привязываем базу к приложению

    cf bind-service my-application mysql-db-prod
    
где my-application - имя приложения
где mysql-db-prod - придуманное имя для базы

Далее запускаем наше приложение

    cf start my-application
    
Но для начала нам надо сделать это самое приложение =)
Вы можете попрактиковаться на любом уже написанном.


- При помощи файла манифеста
Но вместо того, чтобы каждый раз прописывать эти страшные команды мы можем просто напросто оформить конфигурацию приложения в файле-манифесте Cloud Foundry - `manifest.yml`.


    applications:
        name: my-application
        buildpack: https://github.com/ksereda/my-application.git
        instances: 1
        random-route: true
        path: targer/myapp.jar
        services:
            mysql-db-prod
        
        env:
            DEBUG: "true"
            SPRING_PROFILE_ACTIVE: cloud

Для запуска (в консоли):

    cf push -f manifest.yml

#### Пару слов про Java клиент Cloud Foundry

`Java клиент Cloud Foundry` построен на основе проекта `Pivotal Reactor 3.0`, который в сво очередь лежит в основе `Spring 5` -> т.е. Java клиент построен на принципах `reactive` системы.

Все что было написано выше также можно сделать при помощи кода (при помощи Java клиента Cloud Foundry) - `ReactorCloudFoundryClient.`
Вам также потребуется описать:
- `ReactorDropperClient` - клиент для Dropper (это подсистема Cloud Foundry) которая основана на веб сокетах.
- `ReactorUaaClient` - клиент для UAA (это подсистема для авторизации и аутентификации в Cloud Foundry)
- `DefaultCloudFoundryOperations` - представляет операции клиента низкоуровненвой подсистемы.
- `ConnectionContext` - описывает сам экземпляр Cloud Foundry.
- `PasswordGrantTokenProvider` - описывает аутентификацию.

Мы также описываем `ServicesDeployer` - он предоставляет экземпляр базы данных (например MySQL). Он также удаляет экземпляры и их привязку если они уже существуют.
Затем описываем `ApplicationDeployer` - он предоставляет само приложение. Оно привязывается к сервису, который мы описали в `ServicesDeployer`.

Т.е. по сути последние 2 предложения - это и есть вся работа, которую мы проделывали чуть выше с помощью `CLI` и команды `cf`.

Об этом можно подготовить отдельную статью.

Спасибо за прочтение.
