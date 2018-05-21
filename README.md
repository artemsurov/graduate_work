# Краткая жизненная история по теме "Приключения Уя и робота Краулера из Гитлаба в Кубирнетисе"

1. [Рождение Уя и появление Краулера в Докере](#dockerfile_ui_crawler)
   1. [Краулер из Докера](#docker-crowler)
   2. [Уй из Докера](#docker-ui)
      - [Необходимые компоненты](#requirements-ui)
      - [Запуск веб-интерфейса. Переменные окружения](#vars-ui)
      - [Пример для запуска](#run-ui)
2. [Как Уй и Краулер в Кубирнетис ходили](#ui-crawler-kubernetes)
   1. [Уй отправляется в путь](#ui-kubernetes)
      - [Плоть Уя](#ui-kubernetes-deployment)
      - [Дох Уя](#ui-kubernetes-service)
   2. [Краулер догоняет Уя](#crawler-kubernetes)
      - [Плоть Краулера](#crawler-kubernetes-deployment)

   3.[Знакомство с мистером Кроликом](#rabbit-kubernetes)
      - [Плоть Кролика](#rabbit-kubernetes-deployment)
      - [Дух Кролика](#rabbit-kubernetes-service)
   4.[Танцы с Монго-Бонгой](#mongo-kubernetes)
      - [Плоть Монго-Бонги](#mongo-kubernetes-deployment)
      - [Дух Монго-Бонги](#mongo-kubernetes-service)
   5. [Бешеный Тиль](#tiller-kubernetes)
      - [День рождения Бешеного Тиля](#tiller-kubernetes-make)
      - [Запуск Бешеного Тиля на Улицу Кубирнетиса](#tiller-kubernetes-run)
      - [Что с Тилем?](#tiller-kubernetes-test)
3. [Как друзья в Топ-Чарт попали](#charts)

   1. [Уй попадает в Чарт](#charts-ui)

      - [Как тимплейт съел Уя](#charts-ui-deployment)
      - [Дох Уя переселился в Чарт?](#charts-ui-service)
      - [Что же значили те буковки, Уй?](#charts-ui-values)

   2. [Краулер идёт по следу Уя в Чарт](#charts-crawler)

      - [У чарта Краулера есть свой чарт](#charts-crawler-chart)
      - [Краулер раскрывает карты](#charts-crawler-values)
      - [Что же значили те буковки?](#charts-ui-values)
      - [От кого зависит жизнь Краулера?](#charts-crawler-requirements)

   3.[Мечтают ли боты побывать в Чарте?](#charts-bot)

      - [Великие тайны робота](#charts-bot-values)
      - [Мечты сбываются](#charts-bot-chart)
      - [Что у бота под капотом?](#charts-bot-deployment)
      - [Бот спешит на помощь?](#charts-bot-helper)
   4. [Поляна чарта под микроскопом](#charts-structure)
   5. [Кому нужна помощь функций?](#charts-helpers)
   6. [Что внутри помощников?](#charts-helpers-structure)

## Dockerfile UI и Crawler <a name="dockerfile_ui_crawler"></a>


### Dockerfile Crawler <a name="docker-crowler"></a>

Всё начинается снизу вверх, - от малого к большому. Вот и в нашей
истории родился маленький бот Краулер в небольшом местечке под названием
Докер


graduate_work/src/search_engine_crawler/Dockerfile


```docker
FROM python:3.6.4-jessie

WORKDIR /app
COPY . /app

RUN pip install -r /app/requirements.txt

ENV EXCLUDE_URLS '.*github.com'

ENTRYPOINT ["python", "-u", "/app/crawler/crawler.py"]
CMD ["https://vitkhab.github.io/search_engine_test_site/"]
```

### Dockerfile UI. Search Engine UI <a name="docker-ui"></a>

Рядом с ним и в ту же пору проживал мальчик Уй, и путь к Краулеру
пролегал именно через его дом.

graduate_work/src/search_engine_ui/Dockerfile

```docker
FROM python:3.6.4-jessie

WORKDIR /app
COPY . /app

RUN pip install -r /app/requirements.txt

ENV MONGO mongo
ENV FLASK_APP ui.py

WORKDIR /app/ui
ENTRYPOINT ["gunicorn", "ui:app", "-b","0.0.0.0"]
```

Веб-интерфейс поиска слов и фраз на проиндексированных
[ботом](https://github.com/express42/search_engine_crawler) сайтах.

Веб-интерфейс минимален, предоставляет пользователю строку для запроса и
результаты. Поиск происходит только по индексированным сайтам. Результат
содержит только те страницы, на которых были найдены все слова из
запроса. Рядом с каждой записью результата отображается оценка
полезности ссылки (чем больше, тем лучше).

#### Необходимые компоненты <a name="requirements-ui"></a>

Для запуска веб-интерфейса нужно установить дополнительные компоненты
python командой

```
pip install -r requirements.txt
```

#### Запуск веб-интерфейса. Переменные окружения <a name="vars-ui"></a>

* `MONGO` - адрес `mongodb`-хоста
* `dok` - порт для подключения к `mongodb`-хосту

Для работы веб-интерфейса нужен запущенный сервис `mongodb`

#### Пример для запуска <a name="run-ui"></a>

В общем случае веб-интерфейс можно запустить с помощью команд

```
cd ui
FLASK_APP=ui.py gunicorn ui:app -b 0.0.0.0
```

Для проверки работы веб-интефейса надо зайти по адресу
`http://HOST_IP:8000/`, где `HOST_IP` - адрес хоста на котором запущен
веб-интерфейс.

## Разворачиваем Crawler, UI, MongoDB и RabbitMQ в Kubernetes <a name="ui-crawler-kubernetes"></a>
Недолго и тесно друзьям жилось в докере, и решили они отправиться в Столицу Контейнерии - в Кубирнетис



### Разворачиваем UI в Kubernetes <a name="ui-kubernetes"></a>

#### Плоть Уя <a name="ui-kubernetes-deployment"></a>

```yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: ui
  labels:
    app: crawler
    component: ui
spec:
  replicas: 1
  selector:
    matchLabels:
      app: crawler
      component: ui
  template:
    metadata:
      name: ui-pod
      labels:
        app: crawler
        component: ui
    spec:
      containers:
      - image: asurov/crawler-ui
        name: ui
```


#### Дох Уя <a name="ui-kubernetes-service"></a>

Сервис UI мы опубликовали наружу с помощью NodePort

```kubernetes
apiVersion: v1
kind: Service
metadata:
  name: ui
  labels:
    app: crawler
    component: ui
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8000
  selector:
    app: crawler
    component: ui
```

### Разворачиваем Crawler в Kubernetes <a name="crawler-kubernetes"></a>
#### Crawler deployment <a name="crawler-kubernetes-deployment"></a>
```kubernetes
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: crawler
  labels:
    app: crawler
    component: crawler-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: crawler
      component: crawler-app
  template:
    metadata:
      name: crawler-pod
      labels:
        app: crawler
        component: crawler-app
    spec:
      containers:
      - image: asurov/crawler       # Поднимаем контейнер им Николая Артёмовича Краулера
        name: crawler
```
### Разворачиваем RabbitMQ в Kubernetes <a name="rabbit-kubernetes"></a>
#### RabbitMQ deployment <a name="rabbit-kubernetes-deployment"></a>

```yml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: rabbit
  labels:
    app: crawler
    component: rabbit
spec:
  replicas: 1
  selector:
    matchLabels:
      app: crawler
      component: rabbit
  template:
    metadata:
      name: rabbit-pod
      labels:
        app: crawler
        component: rabbit
    spec:
      containers:
      - image: rabbitmq:3           # Поднимаем RabbitMQ 3 версии
        name: rabbit
```

#### RabbitMQ service <a name="rabbit-kubernetes-service"></a>

```yml
apiVersion: v1
kind: Service
metadata:
  name: rabbit
  labels:
    app: crawler
    component: rabbit-svc
spec:
  ports:
  - name: epmd              # Порты взяты из приложенного соответствующего Charta
    port: 5672
  - name: amqp
    port: 4369
  - name: dist
    port: 5671
  - name: stats
    port: 25672
  selector:
    app: crawler
    component: rabbit
```

### Разворачиваем MongoDB в Kubernetes <a name="mongo-kubernetes"></a>
#### MongoDB deployment <a name="mongo-kubernetes-deployment"></a>

Подсистема PersistentVolume предоставляет API для пользователей и администраторов, которые абстрагируют информацию о том, как хранилище предоставляется из того, как оно потребляется.
Поднимем базу данных MongoDB и примонтируем к ней PersistentVolume размером 10Гб. Опишем в одном файле БД и PV:

```yml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: mongo
  labels:
    app: crawler
    component: mongo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: crawler
      component: mongo
  template:
    metadata:
      name: mongo
      labels:
        app: crawler
        component: mongo
    spec:
      containers:
      - image: mongo:3.2                    # Создаём контейнер с Монгой
        name: mongo
        volumeMounts:                       # Маунтим PersistentVolume
        - name: mongo-gce-pd-storage
          mountPath: /data/db
      volumes:
      - name: mongo-gce-pd-storage          # Указываем, какой именно сторадж маунтить
        persistentVolumeClaim:
          claimName: mongo-pvc-dynamic
---
kind: PersistentVolumeClaim                 # Создаём PersistentVolume
apiVersion: v1
metadata:
  name: mongo-pvc-dynamic
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi                         # Размер
```
#### MongoDB service <a name="rabbit-kubernetes-service"></a>

```yml
apiVersion: v1
kind: Service
metadata:
  name: mongodb
  labels:
    app: crawler
    component: mongo
spec:
  ports:
  - port: 27017             # Открываем порты, по которым будем работать с Монгой
    protocol: TCP
    targetPort: 27017
  selector:
    app: crawler
    component: mongo
```

## Установим серверную часть Helm’а - Tiller <a name="tiller-kubernetes"></a>
```markdown
Tiller - это аддон Kubernetes, т.е. Pod, который общается с API Kubernetes.
Для этого понадобится ему выдать ServiceAccount и назначить роли RBAC, необходимые для работы.
```

### Создаём файл tiller.yml и помещаем в него манифест <a name="tiller-kubernetes-make"></a>

```yamlex
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

```bash
kubectl apply -f tiller.yml
```

### Теперь запустим tiller-сервер <a name="tiller-kubernetes-run"></a>

```bash
helm init --service-account tiller
```


### Проверим <a name="tiller-kubernetes-test"></a>

```bash
kubectl get pods -n kube-system --selector app=helm
```
> output

```markdown
NAME                             READY     STATUS    RESTARTS   AGE
tiller-deploy-7f6c5fcc87-jfbtx   1/1       Running   0          12d
```




## Charts <a name="charts"></a>

```markdown
Chart - это пакет в Helm.

Создаём директорию charts в папке kubernetes со следующей структурой директорий:
```
```bash
mkdir bot crawler-app gitlab-omnibus ui
```

### Начнем разработку Chart’а для компоненты ui приложения <a name="charts-ui"></a>

ui/Chart.yaml helm предпочитает .yaml

```yaml
name: ui
version: 1.0.0
description: UI interface for crawler app
maintainers:
  - name: Artem Surov
    email: artemka@surov.com
appVersion: 1.0
```
```markdown
Основным содержимым Chart’ов являются шаблоны манифестов Kubernetes.
```
### Установка Chart'a <a name="charts-install"></a>

```bash
helm install --name test-ui-1 ui/
```




```markdown
name: {{ .Release.Name }}-{{ .Chart.Name }}

Здесь мы используем встроенные переменные

.Release - группа переменных с информацией о релизе (конкретном запуске Chart’а в k8s)

.Chart - группа переменных с информацией о Chart’е (содержимое файла Chart.yaml)

Также еще есть группы переменных:

.Template - информация о текущем шаблоне ( .Name и .BasePath)

.Capabilities - информация о Kubernetes (версия, версии API)
 
.Files.Get - получить содержимое файла
```

```bash
kubectl get ingress
```
>output

```markdown
NAME      HOSTS     ADDRESS         PORTS     AGE
ui        *         35.201.91.172   80, 443   6h
```
> По IP-адресам можно попасть на разные релизы ui- приложений (необходимо подождать несколько минут).


### Кастомизируем установку своими переменными (образ и порт). <a name="charts-ui-deployment"></a>

ui/templates/deployment.yaml

```yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: {{ template "ui.fullname" . }}
  labels:
    app: crawler
    component: ui
    release: {{ .Release.Name }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: crawler
      component: ui
      release: {{ .Release.Name }}
  template:
    metadata:
      name: ui-pod
      labels:
        app: crawler
        component: ui
        release: {{ .Release.Name }}
    spec:
      containers:
      - image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}" # Ссылка на наш репозиторий и сам имедж 
        name: ui
        env: 
        - name: MONGO
          value: {{ .Values.mongoHost | default (printf "%s-mongodb" .Release.Name) }}
```

### Сервис веб-приложения в чарте <a name="charts-ui-service"></a>

ui/templates/service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: reddit
    component: ui
    release: {{ .Release.Name }}
spec:
  type: NodePort
  ports:
  - port: {{ .Values.service.externalPort }}        # Параметризируем порт внешний
    protocol: TCP
    targetPort: {{ .Values.service.internalPort }}  # И внутренний
  selector:
    app: reddit
    component: ui
    release: {{ .Release.Name }}
```

ui/templates/ingress.yaml

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  annotations:
    kubernetes.io/ingress.class: "gce"
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: {{ .Release.Name }}-{{ .Chart.Name }}
          servicePort: {{ .Values.service.externalPort }}
```

### Собственно, определяем значения переменных: <a name="charts-ui-values"></a>

ui/templates/values.yaml

```yaml
service:
  internalPort: 9292
  externalPort: 9292

image:
  repository: asurov/crawler
  tag: latest

MONGO:

RMQ_HOST:
```

```markdown
Поскольку адрес БД может меняться в зависимости от условий запуска:
• бд отдельно от кластера
• бд запущено в отдельном релизе
• ...
, то создадим удобный шаблон для задания адреса БД.
env:
- name: POST_DATABASE_HOST
value: {{ .Values.databaseHost }}

Будем задавать бд через переменную databaseHost. Иногда лучше использовать подобный формат переменных
вместо структур database.host, так как тогда прийдется определять структуру database, иначе helm выдаст ошибку.
Используем функцию default. Если databaseHost не будет определена или ее значение будет пустым, то используется
вывод функции printf (которая просто формирует строку <имя- релиза>-mongodb)

value: {{ .Values.databaseHost | default (printf "%s-mongodb" .Release.Name) }}
                                                       |
                                                       release-name-mongodb
В итоге должно получиться
                                                       
env:
- name: POST_DATABASE_HOST
  value: {{ .Values.databaseHost | default (printf "%s-mongodb" .Release.Name) }}
             

Теперь, если databaseHost не задано, то будет использован адрес базы, поднятой внутри релиза
```

## Шаблонизируем сервис crawler: <a name="charts-crawler"></a>


### Helm chart crawler <a name="charts-crawler-chart"></a>
crawler-app/Chart.yaml

```yaml
apiVersion: v1
appVersion: "1.0"
description: A Helm chart for Kubernetes
name: crawler-app
version: 0.1.0
```

### Crawler values vars <a name="charts-crawler-values"></a>
crawler-app/values.yaml

```yaml
bot:
  image:
    repository: asurov/crawler
    tag: latest
ui:
  image:
    repository: asurov/crawler-ui
    tag: latest
rabbitmq:
  rabbitmq:
      username: guest
      password: guest
mongodb:
  usePassword: false
```

### Указываем зависимости, которые необходимо подгрузить для работы приложения, - другие соседние сервисы:

kubernetes/charts/crawler-app/requirements.yaml <a name="charts-crawler-requirements"></a>

```yml
dependencies:
  - name: ui
    version: 1.0.0
    repository: file://../ui
  - name: bot
    version: 1.0.0
    repository: file://../bot
  - name: rabbitmq
    version: 0.8.1
    repository: https://kubernetes-charts.storage.googleapis.com
  - name: mongodb
    version: 2.0.4
    repository: https://kubernetes-charts.storage.googleapis.com
```
Когда запускаем Хельм, создаются архивы тех приложений, которые описаны в requirements.yaml, в папке kubernetes/charts/crawler-app/charts

## Шаблонизируем сервис Робота <a name="charts-bot"></a>

### Bot values var <a name="charts-bot-values"></a>

bot/values.yaml

```yaml
image:
  repository: asurov/crawler
  tag: latest

MONGO:

RMQ_HOST:
```
### Bot Chart <a name="charts-bot-chart"></a>
bot/Chart.yaml

```yaml
name: bot
version: 1.0.0
description: Bot for grab information by sites
maintainers:
  - name: Artem Surov
    email: artemka@surov.com
appVersion: 1.0
```

### Bot Chart Deployment <a name="charts-bot-deployment"></a>

bot/templates/deployment.yaml

```yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: {{ template "bot.fullname" . }}
  labels:
    app: crawler
    component: bot
    release: {{ .Release.Name }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: crawler
      component: bot
      release: {{ .Release.Name }}
  template:
    metadata:
      name: crawler-pod
      labels:
        app: crawler
        component: bot
        release: {{ .Release.Name }}
    spec:
      containers:
      - image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        name: crawler
        env: 
        - name: MONGO
          value: {{ .Values.mongoHost | default (printf "%s-mongodb" .Release.Name) }}
        - name: RMQ_HOST
          value: {{ .Values.rmqHost | default (printf "%s-rabbitmq" .Release.Name) }}
```

### Bot Chart helper <a name="charts-bot-helper"></a>

bot/templates/_helpers.tpl

```yaml
{{- define "bot.fullname" -}}
{{- printf "%s-%s" .Release.Name .Chart.Name }}
{{- end -}}
```



##### Итоговая структура выглядит вот так: <a name="charts-structure"></a>

```markdown
├── bot
│   ├── Chart.yaml
│   ├── templates
│   │   ├── deployment.yaml
│   │   └── _helpers.tpl
│   └── values.yaml
├── crawler-app
│   ├── Chart.yaml
│   ├── requirements.yaml
│   └── values.yaml
├── gitlab-omnibus
│   ├── CHANGELOG.md
│   ├── charts
│   │   └── gitlab-runner
│   │       ├── Chart.yaml
│   │       ├── README.md
│   │       ├── templates
│   │       │   ├── _cache_s3.tpl
│   │       │   ├── configmap.yaml
│   │       │   ├── deployment.yaml
│   │       │   ├── _helpers.tpl
│   │       │   ├── NOTES.txt
│   │       │   ├── role-binding.yaml
│   │       │   ├── role.yaml
│   │       │   ├── secrets.yaml
│   │       │   └── service-account.yaml
│   │       └── values.yaml
│   ├── Chart.yaml
│   ├── README.md
│   ├── requirements.yaml
│   ├── templates
│   │   ├── fast-storage
│   │   │   └── storage.yaml
│   │   ├── gitlab
│   │   │   ├── gitlab-config-storage.yaml
│   │   │   ├── gitlab-deployment.yaml
│   │   │   ├── gitlab-storage.yaml
│   │   │   ├── gitlab-svc.yaml
│   │   │   ├── postgresql-configmap.yaml
│   │   │   ├── postgresql-deployment.yaml
│   │   │   ├── postgresql-storage.yaml
│   │   │   ├── postgresql-svc.yaml
│   │   │   ├── redis-deployment.yaml
│   │   │   ├── redis-storage.yaml
│   │   │   └── redis-svc.yaml
│   │   ├── gitlab-config.yaml
│   │   ├── _helpers.tpl
│   │   ├── ingress
│   │   │   ├── gitlab-ingress.yaml
│   │   │   └── gitlab-pages-ingress.yaml
│   │   ├── load-balancer
│   │   │   ├── lego
│   │   │   │   ├── 00-namespace.yaml
│   │   │   │   ├── configmap.yaml
│   │   │   │   └── deployment.yaml
│   │   │   └── nginx
│   │   │       ├── 00-namespace.yaml
│   │   │       ├── configmap.yaml
│   │   │       ├── daemonset.yaml
│   │   │       ├── default-deployment.yaml
│   │   │       ├── default-service.yaml
│   │   │       ├── service.yaml
│   │   │       └── tcp-configmap.yaml
│   │   └── NOTES.txt
│   └── values.yaml
└── ui
    ├── Chart.yaml
    ├── templates
    │   ├── deployment.yaml
    │   ├── _helpers.tpl
    │   └── service.yaml
    └── values.yaml


```

```markdown
Также стоит отметить функционал helm по использованию helper’ов и функции templates.
Helper - это написанная нами функция. В функция описывается, как правило, сложная логика.

Шаблоны этих функций распологаются в файле _helpers.tpl
```
## Пример функции comment.fullname <a name="charts-helpers"></a>

_helpers.tpl

```markdown
{{- define "comment.fullname" -}}
{{- printf "%s-%s" .Release.Name .Chart.Name }}
{{- end -}}
```
которая в результате выдаст то же, что и:
{{ .Release.Name }}-{{ .Chart.Name }}






```markdown
с помощью template вызывается функция comment.fullname, описанная ранее в файле _helpers.tpl
```

### Структура ипортирующей функции template <a name="charts-helpers-structure"></a>

```markdown
                {{ template "comment.fullname" . }}
                    |               |          |
                    
         Функция template         Название    область видимости
                                функции для     для импорта
                                  импорта  
                             
“.”- вся область видимости всех переменных (можно передать .Chart , тогда .Values не будут доступны внутри функции)                             
```
