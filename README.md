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
      - [День рождения Бешеного Тиля](#tiller-kubernetes-make)
      - [Запуск Бешеного Тиля на Улицу Кубирнетиса](#tiller-kubernetes-run)
      - [Что с Тилем?](#tiller-kubernetes-test)

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
      - image: asurov/crawler
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
      - image: rabbitmq:3
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
  - name: epmd
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
      - image: mongo:3.2
        name: mongo
        volumeMounts:
        - name: mongo-gce-pd-storage
          mountPath: /data/db
      volumes:
      - name: mongo-gce-pd-storage
        persistentVolumeClaim:
          claimName: mongo-pvc-dynamic
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mongo-pvc-dynamic
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
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
  - port: 27017
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

### Начнем разработку Chart’а для компоненты ui приложения

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

#### Templates

```markdown
Основным содержимым Chart’ов являются шаблоны манифестов Kubernetes.
```
### Установка Chart'a

```bash
helm install --name test-ui-1 ui/
```
> output

```markdown
NAME:   test-ui-1
LAST DEPLOYED: Sun Apr 15 16:12:41 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1beta1/Deployment
NAME  AGE
ui    1s

==> v1beta1/Ingress
ui  1s

==> v1/Service
ui  1s
```

##### 3) Посмотрим, что получилось

```bash
helm ls
```

```markdown
NAME            REVISION        UPDATED                         STATUS          CHART           NAMESPACE
test-ui-1       1               Sun Apr 15 16:12:41 2018        DEPLOYED        ui-1.0.0        default  
```

##### Теперь сделаем так, чтобы можно было использовать 1 Chart для запуска нескольких экземпляров (релизов). Шаблонизируем его.

ui/templates/service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}   # Нам нужно уникальное имя запущенного ресурса
  labels:
    app: reddit
    component: ui
    release: {{ .Release.Name }}                # Помечаем, что сервис из конкретного релиза
spec:
  type: NodePort
  ports:
  - port: 9292
    protocol: TCP
    targetPort: 9292
  selector:
    app: reddit
    component: ui
    release: {{ .Release.Name }}                # Выбираем поды только из этого релиза
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

##### Шаблонизируем подобным образом остальные сущности

ui/templates/deployment.yaml

```markdown
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: reddit
    component: ui
    release: {{ .Release.Name }}
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:                                     # важно, чтобы selector deployment’а нашел только нужные POD’ы.
    matchLabels:
      app: reddit
      component: ui
      release: {{ .Release.Name }}
  template:
    metadata:
      name: ui-pod
      labels:
        app: reddit
        component: ui
        release: {{ .Release.Name }}
    spec:
      containers:
      - image: asomir/ui
        name: ui
        ports:
        - containerPort: 9292
          name: ui
          protocol: TCP
        env:
        - name: ENV
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
```

ui/templates/ingress.yaml

```markdown
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
      - path: /*
        backend:
          serviceName: {{ .Release.Name }}-{{ .Chart.Name }}
          servicePort: 9292
```

##### Установим несколько релизов ui

```bash
helm install ui --name ui-1
helm install ui --name ui-2 
helm install ui --name ui-3
```
ui-1 ui-2 ui-3 - Имя релиза

> output

```markdown
NAME:   ui-3
LAST DEPLOYED: Sun Apr 15 16:25:47 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Service
NAME     AGE
ui-3-ui  0s

==> v1beta1/Deployment
ui-3-ui  0s

==> v1beta1/Ingress
ui-3-ui  0s
```
##### Проверяем ингрессы

```bash
kubectl get ingress
```
>output

```markdown
NAME      HOSTS     ADDRESS          PORTS     AGE
ui-1-ui   *         35.186.251.110   80        2m
ui-2-ui   *         35.190.51.208    80        2m
ui-3-ui   *         35.190.44.156    80        2m
```
> По IP-адресам можно попасть на разные релизы ui- приложений (необходимо подождать несколько минут).

Мы уже сделали возможность запуска нескольких версий
приложений из одного пакета манифестов, используя лишь
встроенные переменные.

##### Кастомизируем установку своими переменными (образ и порт).

ui/templates/deployment.yaml

```yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: reddit
    component: ui
    release: {{ .Release.Name }}
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: reddit
      component: ui
      release: {{ .Release.Name }}
  template:
    metadata:
      name: ui
      labels:
        app: reddit
        component: ui
        release: {{ .Release.Name }}                                        
    spec:
      containers:
      - image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"  # Ссылка на наш репозиторий и сам имедж 
        name: ui
        ports:
        - containerPort: {{ .Values.service.internalPort }}              # Параметризируем порт
          name: ui
          protocol: TCP
        env:
        - name: ENV
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
```
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

##### Собственно, определяем значения переменных:

```yaml
service:
  internalPort: 9292
  externalPort: 9292

image:
  repository: asomir/ui
  tag: latest
```

##### Запускаем наши образы:

```bash
helm upgrade ui-1 ui/
helm install ui-2 ui/
helm install ui-3 ui/
```
##### Мы собрали Chart для развертывания ui-компоненты приложения. Он должен иметь следующую структуру:

```markdown
└── ui
    ├── Chart.yaml
    └── templates
        ├── deployment.yaml
        ├── ingress.yaml
        └── service.yaml
```

##### Осталось собрать пакеты для остальных компонент

post/templates/service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: reddit
    component: post
    release: {{ .Release.Name }}
spec:
  type: ClusterIP
  ports:
  - port: {{ .Values.service.externalPort }}
    protocol: TCP
    targetPort: {{ .Values.service.internalPort }}
  selector:
    app: reddit
    component: post
    release: {{ .Release.Name }}
```

post/templates/deployment.yaml

```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: reddit
    component: post
    release: {{ .Release.Name }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reddit
      component: post
      release: {{ .Release.Name }}
  template:
    metadata:
      name: post
      labels:
        app: reddit
        component: post
        release: {{ .Release.Name }}
    spec:
      containers:
      - image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        name: post
        ports:
        - containerPort: {{ .Values.service.internalPort }}
          name: post
          protocol: TCP
        env:
        - name: POST_DATABASE_HOST
          value: postdb
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
post/values.yaml

```yaml
service:
  internalPort: 5000
  externalPort: 5000

image:
  repository: asomir/post
  tag: latest

databaseHost:
```
##### Шаблонизируем сервис comment:

comment/templates/deployment.yaml

```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: reddit
    component: comment
    release: {{ .Release.Name }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reddit
      component: comment
      release: {{ .Release.Name }}
  template:
    metadata:
      name: comment
      labels:
        app: reddit
        component: comment
        release: {{ .Release.Name }}
    spec:
      containers:
      - image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        name: comment
        ports:
        - containerPort: {{ .Values.service.internalPort }}
          name: comment
          protocol: TCP
        env:
        - name: COMMENT_DATABASE_HOST
          value: {{ .Values.databaseHost | default (printf "%s-mongodb" .Release.Name) }}
```
comment/templates/service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: reddit
    component: comment
    release: {{ .Release.Name }}
spec:
  type: ClusterIP
  ports:
  - port: {{ .Values.service.externalPort }}
    protocol: TCP
    targetPort: {{ .Values.service.internalPort }}
  selector:
    app: reddit
    component: comment
    release: {{ .Release.Name }}
```

comment/values.yaml

```yaml
service:
  internalPort: 9292
  externalPort: 9292

image:
  repository: asomir/comment
  tag: latest

databaseHost:
```

##### Итоговая структура выглядит вот так:

```markdown
├── comment
│   ├── Chart.yaml
│   ├── templates
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── values.yaml
├── post
│   ├── Chart.yaml
│   ├── templates
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── values.yaml
├── reddit
└── ui
    ├── Chart.yaml
    ├── templates
    │   ├── deployment.yaml
    │   ├── ingress.yaml
    │   └── service.yaml
    └── values.yaml

```

```markdown
Также стоит отметить функционал helm по использованию helper’ов и функции templates.
Helper - это написанная нами функция. В функция описывается, как правило, сложная логика.

Шаблоны этих функций распологаются в файле _helpers.tpl
```
##### Пример функции comment.fullname :

charts/comment/templates/_helpers.tpl

```markdown
{{- define "comment.fullname" -}}
{{- printf "%s-%s" .Release.Name .Chart.Name }}
{{- end -}}
```
которая в результате выдаст то же, что и:
{{ .Release.Name }}-{{ .Chart.Name }}


##### И заменим в соответствующие строчки в файле, чтобы использовать helper

charts/comment/templates/service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ template "comment.fullname" . }}  # было name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: reddit
    component: comment
    release: {{ .Release.Name }}
spec:
  type: ClusterIP
  ports:
  - port: {{ .Values.service.externalPort }}
    protocol: TCP
    targetPort: {{ .Values.service.internalPort }}
  selector:
    app: reddit
    component: comment
    release: {{ .Release.Name }}
```

```markdown
с помощью template вызывается функция comment.fullname, описанная ранее в файле _helpers.tpl
```

##### Структура ипортирующей функции template

```markdown
                {{ template "comment.fullname" . }}
                    |               |          |
                    
         Функция template         Название    область видимости
                                функции для     для импорта
                                  импорта  
                             
“.”- вся область видимости всех переменных (можно передать .Chart , тогда .Values не будут доступны внутри функции)                             
```