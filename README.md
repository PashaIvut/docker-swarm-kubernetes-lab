# ЛР: Деплой и управление жизненным циклом приложения в Kubernetes
## Среда выполнения - Killercoda.
## Часть 1. Проверка доступа к кластеру и namespace

### Задание 1. Проверка кластера
1. Получим информацию о кластере.  
<img width="954" height="147" alt="image" src="https://github.com/user-attachments/assets/aef62163-2bf8-4c52-b769-94e4c00899aa" />   

Разбор вывода:
- 172.30.1.2 — это внутренний IP нашего главного узла.

Кластер успешно развёрнут и управляющие компоненты работают. 

==============================================================  
2. Выведем список нод.  
<img width="495" height="111" alt="image" src="https://github.com/user-attachments/assets/7fd70679-a8f8-44f7-825a-8d56d17750c8" />  

У нас две ноды в кластере:
- controlplane — главная нода. Содержит компоненты управления кластером.
- node01 — рабочая нода. Здесь будут запускаться поды.

==============================================================  
Контрольные вопросы:  
1) Node – это отдельная машина (физическая или виртуальная) в кластере, на которой могут запускаться поды.
Cluster – совокупность всех узлов и управляющих компонентов, которые работают как единая система.  
2) В моём кластере control plane находится на ноде controlplane, а worker — на ноде node01. Это видно в выводе **kubectl get nodes** по колонке ROLES. Все прозрачно.

==============================================================  

### Задание 2. Namespace
1) Cоздадим **namespace**.
<img width="488" height="44" alt="image" src="https://github.com/user-attachments/assets/4e2cbd99-5e5d-4282-88df-e5c1836ef777" />

2) Проверим, что он создался.
<img width="325" height="198" alt="image" src="https://github.com/user-attachments/assets/54d2b233-ab4f-461c-9c06-6f2563cdf954" />

3) Сделаем его namespace по умолчанию.  
<img width="775" height="52" alt="image" src="https://github.com/user-attachments/assets/e8924e50-e65b-4c76-84d4-15826f8b8e39" />  


Контрольные вопросы:  
- Namespace нужен для логического разделения ресурсов кластера между разными командами, окружениями. Это помогает избежать конфликтов, позволяет управлять доступом.
- Если работать в default, можно случайно повлиять на другие процессы или "украсть" их ресурсы. Также в реальных проектах default часто используют для системных компонентов и работать там не принято.

============================================================== 
## Часть 2. Deployment и реплики (через YAML)
### Задание 3. Создание Deployment web  
Создадим файл deployment-web.yaml по указанной структуре.  
<img width="504" height="463" alt="image" src="https://github.com/user-attachments/assets/cbce86d2-84d4-46d8-bf3e-a18c6d23ed86" />


Проверим, что он создаллся.  
<img width="564" height="49" alt="image" src="https://github.com/user-attachments/assets/5f0b5571-4944-49fc-be89-e83571faff1b" />  

Применим манифест.  
<img width="530" height="45" alt="image" src="https://github.com/user-attachments/assets/bc3cd327-dbbd-42c6-9829-90a83f0216a7" />  
При применении манифеста можно отбросить флаг -n с названием namespace, так как мы сделали наш napespace основным в предыдущем задании.

Проверим, что поды запущены с помощью команды **kubectl get deployments,pods**, которая запрашивает одновременно информацию о двух типах объектов — Deployment и Pod в текущем namespace.  

<img width="561" height="181" alt="image" src="https://github.com/user-attachments/assets/2ea36753-b3a7-4891-8bf0-9ba02e5c48cf" />  

Кратко о выводе:
- Всё работает — 3 пода, все в статусе Running  
- Масштабирование выполнено — реплик ровно 3, как указано в манифесте  
- ReplicaSet активен — одинаковый хеш у всех подов  


Контрольные вопросы:
- Deployment, ReplicaSet и Pod отличаются уровнем ответственности (по сути, формируется иерархия). Pod представляет собой самую маленькую единицу, внутри которой запускаются контейнеры с приложением. ReplicaSet — это контроллер, который следит за тем, чтобы всегда работало заданное количество подов. Deployment управляет ReplicaSet и обеспечивает обновления, масштабирование и откаты приложения.
- Желаемое состояние хранится в control plane. Его поддерживают контроллеры — они постоянно сравнивают текущее состояние кластера с желаемым, и если обнаруживают расхождение, то выполняют действия для его достижения, например, создают или удаляют поды.

============================================================== 

## Часть 3. Масштабирование и наблюдение
### Задание 4. Масштабирование  
Увеличим до 5-ти реплик.  
<img width="574" height="42" alt="image" src="https://github.com/user-attachments/assets/e200f8b1-a992-4e60-b005-9a93ece11e74" />  

Проверим, что количество подов увеличилось (так и есть).  
<img width="505" height="156" alt="image" src="https://github.com/user-attachments/assets/f42b654c-0b43-4e90-bee0-278e9c15fa55" />


Уменьшим до 2-х.
<img width="1104" height="137" alt="image" src="https://github.com/user-attachments/assets/c8ab2330-49ad-44f0-9ba4-63a8ce3e0905" />  
Количество уменьшилось.  


Я предполагал, что при уменьшении количества реплик Kubernetes сначала удалит более новые поды, так как старые поды потенциально содержат больше накопленных данных или кэша, и их потеря могла бы быть критичной. Но нет. После нескольких попыток увеличения и уменьшения реплик никакой закономерности я не увидел.  

Контрольные вопросы:
- Kubernetes может удалить любой под при scale down, потому что поды взаимозаменяемы — они не имеют уникальной идентичности и не хранят состояние, поэтому для системы главное чтобы общее количество соответствовало заданному. 
- В выводе kubectl get pods -o wide колонка READY показывает 1/1 для всех подов — контейнер nginx работает и готов принимать трафик. Readiness probe проверяет готовность пода: если не проходит, READY будет 0/1, и под исключается из балансировки. Liveness probe проверяет жизнеспособность контейнера: если не проходит, Kubernetes перезапускает контейнер. В моём Deployment пробы не настроены, поэтому Kubernetes считает под готовым сразу после запуска.

============================================================== 

## Часть 4. Service и доступность
### Задание 5. Публикация Service

Выбираю тип NodePort, так как работаю в Killercoda.  
Создадим service-web.yaml.   
<img width="542" height="308" alt="image" src="https://github.com/user-attachments/assets/d4493900-b27f-40f9-bfdd-cd075d08cff2" />  

Применим манифест.  
<img width="511" height="45" alt="image" src="https://github.com/user-attachments/assets/cdb3b992-b00e-4f90-a622-b5bc799079bd" />  

Проверим, что сервис создался.  
<img width="692" height="92" alt="image" src="https://github.com/user-attachments/assets/98b4b561-af2e-427d-b963-81d58593c4b8" />  

Проверим эндпоинты.  
<img width="786" height="120" alt="image" src="https://github.com/user-attachments/assets/4a824edd-07bd-4c04-9edf-641ea0d22bfa" />  
Все хорошо, selector совпадает с labels подов.  

Проверим доступность приложения.
Посмотрим номер NodePort.
<img width="717" height="71" alt="image" src="https://github.com/user-attachments/assets/a8af2018-fd4a-4273-be82-e79018e0832d" />  
**80:32260/TCP** - наш порт.

Для проверки откроем меню справа, выберем меню traffic/ports.
Видим две ноды, введем наш порт 32260 и нажмем Аccess.
<img width="1456" height="804" alt="image" src="https://github.com/user-attachments/assets/b0567da4-ff16-4563-9668-a41ad55f6b0d" />  

Все работает! Ура! Успех!!! ;)
<img width="1327" height="524" alt="image" src="https://github.com/user-attachments/assets/4618d140-83c8-45e1-9a87-600cfe6efeae" />  

Контрольные вопросы:
- Service имеет постоянный IP и имя, которые не меняются. Он автоматически отслеживает поды по labels и обновляет список endpoints, поэтому трафик всегда идет на живые поды, даже если их IP меняются.
- Если кратко: ClusterIP — доступен только внутри кластера. NodePort — открывает порт на каждой ноде для доступа извне.

==============================================================  

## Часть 5. Rolling update и rollback
### Задание 6. Обновление версии (rolling update)

Проверим текущую версию nginx.   
<img width="605" height="45" alt="image" src="https://github.com/user-attachments/assets/6f2cfeaf-67a8-443e-bf76-bafbd77bf1bc" />  



Обновление прошло успешно.  
<img width="652" height="41" alt="image" src="https://github.com/user-attachments/assets/bbd2bd0c-6d57-4e9a-bbe8-6a00b1c69b05" />  

Видим, как исчезают старые поды и появляются новые.  
<img width="862" height="227" alt="image" src="https://github.com/user-attachments/assets/370f1383-9c42-4c3e-b9fd-b02e2521e354" />  
Успешное завершение rollout - "successfully rolled out"

Версия действиетльно новая.  
<img width="611" height="51" alt="image" src="https://github.com/user-attachments/assets/a2a55a24-b706-482f-b32c-3e80ffb5f76d" />  

Смотрим историю ревизий.  
<img width="552" height="111" alt="image" src="https://github.com/user-attachments/assets/1539039f-d032-4126-b096-d03726873cac" />  
Ревизия 1 — версия 1.26  
Ревизия 2 — версия 1.27  

Контрольные вопросы:
- При rollout обновляется шаблон подов в Deployment. Deployment создаёт новый ReplicaSet с новым образом, а старый остаётся. Затем постепенно заменяет старые поды новыми, увеличивая количество подов в новом ReplicaSet и уменьшая в старом. 
- Kubernetes хранит историю ревизий, чтобы можно было выполнить откат на предыдущую стабильную версию в случае проблем с новой.

### Задание 7. Откат (rollback)  
Выполним откат на предыдущую версию. При этом используем только **undo**, потому что откат автоматически происходит к предыдущей ревизии.  
<img width="521" height="39" alt="image" src="https://github.com/user-attachments/assets/9a3141a6-94d7-4cbd-8ac9-e389e2c64d5e" />  

Откат прошел успешно.  
<img width="542" height="52" alt="image" src="https://github.com/user-attachments/assets/98b0d0f1-e768-4b7f-ab5f-6790cb39775f" />  

Доказательство, что вернулись на прежнюю версию.  
<img width="600" height="48" alt="image" src="https://github.com/user-attachments/assets/987483a7-9989-481e-a80c-00ca725a7e11" />  

Теперь ревизия 3 - это наша вернувшаяся версия.  
<img width="554" height="113" alt="image" src="https://github.com/user-attachments/assets/188fe896-b0d7-4d82-a350-f40691676262" />  

Контрольные вопросы:
- Предыдущей ревизией считается состояние Deployment до последнего обновления. В истории ревизий это номер, который был перед текущим.
- Rollback критически важен, когда новое обновление содержит критическую ошибку. Важно быстро откатиться к стабильной версии.

==============================================================  

## Часть 6. Canary / A–B через две версии и разные реплики
### Задание 8. Проектирование labels

**Схема:**  
<img width="487" height="289" alt="image" src="https://github.com/user-attachments/assets/66fa3aef-d58c-4e5f-9b12-28feaf9fa37b" />


Общий label (app: web-app) — чтобы Service находил все поды. Разные labels (track: stable/canary) — чтобы различать версии и управлять ими.


Для начала увеличим stable до 4-х реплик, перед этим добавив label track: stable в Deployment web.  
<img width="592" height="45" alt="image" src="https://github.com/user-attachments/assets/ed492d77-f6e0-404e-9052-13809acb8dd1" />  

Проверим stable-поды.  
<img width="890" height="122" alt="image" src="https://github.com/user-attachments/assets/321f55fd-2c80-4dfd-81a2-740f0ae53b89" />  

Создадим canary deployment с другой версией nginx.  
<img width="241" height="481" alt="image" src="https://github.com/user-attachments/assets/9bf6d185-fb8a-44fd-aaba-342fe19762a7" />  

Проверим, что имеется 4-е stable-пода и 1 canary.  
<img width="981" height="142" alt="image" src="https://github.com/user-attachments/assets/00a7677a-86e3-416c-a7fe-728340b7898b" />  
Так и есть! Ура! Победа! ;)  

Проверим эндпоинты.  
<img width="696" height="86" alt="image" src="https://github.com/user-attachments/assets/351c11fc-4455-4989-8a0f-2044404930c3" />  
Все хорошо.

### Задание 10. Проверка A–B поведения
Узнаем имя canary-пода.  
<img width="531" height="63" alt="image" src="https://github.com/user-attachments/assets/31780b0f-d3a9-40a0-92a3-bf4bffd9c9d4" />  

Начнем просматривать его логи. Отправляем 20 запросов.
<img width="793" height="344" alt="image" src="https://github.com/user-attachments/assets/2769023e-3b01-4a85-a236-a1e15f486e0f" />  

В среднем в canary под попадало от 3-х до 4-х запросов (было предприянто несколького попыток отправки запросов).  
<img width="750" height="163" alt="image" src="https://github.com/user-attachments/assets/17b45c0f-917b-415d-9bca-d57a86a902f9" />

Ответ для секции "На подумать":
Я выбрал вариант фиксация попадания запросов на разные pod'ы через логи, потому что:
- Простота реализации — не требует каких-либо настроек
- Наглядность — в логах canary-пода сразу видно, какие запросы на него попали
- Точность — можно точно посчитать количество запросов на canary и сравнить с общим числом

Контрольные вопросы:  
- Kubernetes просто кидает запросы случайным образом по всем доступным подам (возможно имеет место какая-то балансировка на основе случайного выбора). Где подов больше — туда чаще и попадает. У меня 4 stable и 1 canary, поэтому canary получает примерно каждый пятый запрос.
- Labels — для поиска и группировки. По ним Service находит поды. Annotations — что-то типа заметок для людей, система на них не смотрит.

==============================================================  

## Часть 7. Диагностика  
### Задание 11. Ошибка selector (искусственно)

Посмотрим текущий селектор.  
<img width="595" height="47" alt="image" src="https://github.com/user-attachments/assets/896e13f6-6510-4550-8806-cb7b46bc4393" />  

Добавим ошибку в selector.  
<img width="233" height="40" alt="image" src="https://github.com/user-attachments/assets/138b76be-563d-4fd5-ade6-c15272ab5c36" />  

Эндпоинты теперь пустые. Сервис не нашел поды.  
<img width="693" height="87" alt="image" src="https://github.com/user-attachments/assets/1f133df5-2f57-4aee-8c3f-19bf6c852cf8" />  

Найдем причину. Просмотрим labels у подов.  
<img width="950" height="147" alt="image" src="https://github.com/user-attachments/assets/0b41e622-1418-45d5-b834-81cc59575bb8" />   

Посмотрим на селектор.  
<img width="568" height="46" alt="image" src="https://github.com/user-attachments/assets/76c98663-1972-4053-ad28-719503e659d6" />   

У подов app=web-app, а сервис ищет app=web-app-wrong — нестыковочка)))  

Откатимся к старому селектору с **app: web-app**.  
Все починилось, эндпоинты видны.  
<img width="691" height="422" alt="image" src="https://github.com/user-attachments/assets/88599fad-3c28-414c-a2de-15d759afb978" />  

Контрольные вопросы:
- Service есть, но его selector не совпадает с labels подов. В результате у сервиса нет endpoints, и трафику просто некуда идти. 
- Посмотреть kubectl get endpoints. Если в колонке ENDPOINTS пусто, значит selector не нашёл поды.


































 

