## GitLab CI

#### Gitlab будем ставить с помощью Helm Chart’а из пакета Omnibus.

##### 1) Добавим репозиторий Gitlab

```bash
helm repo add gitlab https://charts.gitlab.io
```

##### 2) Мы будем менять конфигурацию Gitlab, поэтому скачаем Chart

```bash
helm fetch gitlab/gitlab-omnibus --untar

cd gitlab-omnibus

helm install --name gitlab . -f values.yaml
```

Выпадает ошибка

> Error: Get
> http://localhost:8080/api/v1/namespaces/kube-system/configmaps?labelSelector=OWNER%!D(MISSING)TILLER:
> dial tcp 127.0.0.1:8080: connect: connection refused

##### изменяем automountServiceAccountToken to true

```bash
kubectl --namespace=kube-system edit deployment/tiller-deploy
```

> Error: Chart incompatible with Tiller v2.9.0-rc3

##### Удалим проверку tiller в Charts/gitlab-omnibus/Chart.yaml

```yaml
apiVersion: v1
description: GitLab Omnibus all-in-one bundle
home: https://about.gitlab.com
icon: https://gitlab.com/gitlab-com/gitlab-artwork/raw/master/logo/logo-square.png
keywords:
- git
- ci
- cd
- deploy
- issue tracker
- code review
- wiki
maintainers:
- email: support@gitlab.com
  name: GitLab Inc.
- name: Mark Pundsack
- name: Jason Plum
- name: DJ Mountney
- name: Joshua Lambert
name: gitlab-omnibus
sources:
- http://docs.gitlab.com/ce/install/kubernetes/
- https://gitlab.com/charts/charts.gitlab.io
version: 0.1.37

```

```bash
helm install --name gitlab . -f values.yaml
```

> output

```markdown
NAME:   gitlab
LAST DEPLOYED: Fri Apr 27 15:23:08 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1beta1/Ingress
NAME           AGE
gitlab-gitlab  2s

==> v1/ConfigMap
gitlab-gitlab-runner             2s
gitlab-gitlab-config             2s
gitlab-gitlab-postgresql-initdb  2s
kube-lego                        2s
nginx                            2s
tcp-ports                        2s

==> v1/StorageClass
gitlab-gitlab-fast  2s

==> v1/PersistentVolumeClaim
gitlab-gitlab-config-storage      2s
gitlab-gitlab-registry-storage    2s
gitlab-gitlab-storage             2s
gitlab-gitlab-postgresql-storage  2s
gitlab-gitlab-redis-storage       2s

==> v1/Service
gitlab-gitlab             2s
gitlab-gitlab-postgresql  2s
gitlab-gitlab-redis       2s
default-http-backend      2s
nginx                     2s

==> v1beta1/DaemonSet
nginx  2s

==> v1/Namespace
kube-lego      2s
nginx-ingress  2s

==> v1/Secret
gitlab-gitlab-runner   2s
gitlab-gitlab-secrets  2s

==> v1beta1/Deployment
gitlab-gitlab-runner      2s
gitlab-gitlab             2s
gitlab-gitlab-postgresql  2s
gitlab-gitlab-redis       2s
kube-lego                 2s
default-http-backend      2s


NOTES:

  It may take several minutes for GitLab to reconfigure.
    You can watch the status by running `kubectl get deployment -w gitlab-gitlab --namespace default
  You did not specify a baseIP so one will be assigned for you.
  It may take a few minutes for the LoadBalancer IP to be available.
  Watch the status with: 'kubectl get svc -w --namespace nginx-ingress nginx', then:

  export SERVICE_IP=$(kubectl get svc --namespace nginx-ingress nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

  Then make sure to configure DNS with something like:
    *.example.com       300 IN A $SERVICE_IP

```


```bash
kubectl get service
```

> output

```markdown
NAME                                  TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                                 AGE
kubernetes                            ClusterIP      10.11.240.1     <none>          443/TCP                                 13d
lovely-sloth-mongodb                  ClusterIP      10.11.250.94    <none>          27017/TCP                               23h
lovely-sloth-rabbitmq                 ClusterIP      10.11.246.238   <none>          4369/TCP,5672/TCP,25672/TCP,15672/TCP   23h
lovely-sloth-ui                       NodePort       10.11.244.11    <none>          8000:32297/TCP                          23h
nginx-nginx-ingress-controller        LoadBalancer   10.11.246.124   35.188.78.251   80:32243/TCP,443:32226/TCP              10m
nginx-nginx-ingress-default-backend   ClusterIP      10.11.240.86    <none>          80/TCP                                  10m
```

````bash
echo "35.188.78.251 gitlab-gitlab staging production” >> /etc/hosts
````

```bash
kubectl get pods
```

>output

```markdown
NAME                                                  READY     STATUS    RESTARTS   AGE
lovely-sloth-bot-77789b759-pqm8x                      1/1       Running   55         23h
lovely-sloth-mongodb-668889f664-zthfc                 1/1       Running   0          23h
lovely-sloth-rabbitmq-0                               1/1       Running   0          7h
lovely-sloth-ui-c4449cfb7-5x7p7                       1/1       Running   0          23h
nginx-nginx-ingress-controller-ddb9c85dc-kkm2k        1/1       Running   0          11m
```

```markdown
Идем по адресу http://gitlab-gitlab
1) Ставим собственный пароль.
2) Логинимся под пользователем root и новым паролем otusgitlab
```

### Запустим проект

##### Создаем группу В качестве имени свой Docker ID. http://gitlab-gitlab/docker-id В настройках группы выберем пункт CI/CD

##### Добавляем 2 переменные - Эти учетные данные будут использованы при сборке и релизе docker-образов с помощью Gitlab CI

CI_REGISTRY_USER - логин в dockerhub CI_REGISTRY_PASSWORD - пароль от Docker Hub

##### В группе создадим новый проекты:
    - se-crawler -Search Engine Crawler - Поисковый бот для сбора текстовой информации с веб-страниц и ссылок..
    - Se-deploy где у нас лежит код инфраструктуры
    - se-ui - Search Engine UI Веб-интерфейс поиска слов и фраз на проиндексированных ботом сайтах.


### Локально у себя создаем директорию Gitlab_ci со следующей структурой директорий.

### Перенесём исходные коды сервиса

````markdown
.
├── CHANGELOG.md
├── charts
│   └── gitlab-runner
│       ├── Chart.yaml
│       ├── README.md
│       ├── templates
│       │   ├── _cache_s3.tpl
│       │   ├── configmap.yaml
│       │   ├── deployment.yaml
│       │   ├── _helpers.tpl
│       │   ├── NOTES.txt
│       │   ├── role-binding.yaml
│       │   ├── role.yaml
│       │   ├── secrets.yaml
│       │   └── service-account.yaml
│       └── values.yaml
├── Chart.yaml
├── README.md
├── requirements.yaml
├── templates
│   ├── fast-storage
│   │   └── storage.yaml
│   ├── gitlab
│   │   ├── gitlab-config-storage.yaml
│   │   ├── gitlab-deployment.yaml
│   │   ├── gitlab-storage.yaml
│   │   ├── gitlab-svc.yaml
│   │   ├── postgresql-configmap.yaml
│   │   ├── postgresql-deployment.yaml
│   │   ├── postgresql-storage.yaml
│   │   ├── postgresql-svc.yaml
│   │   ├── redis-deployment.yaml
│   │   ├── redis-storage.yaml
│   │   └── redis-svc.yaml
│   ├── gitlab-config.yaml
│   ├── _helpers.tpl
│   ├── ingress
│   │   ├── gitlab-ingress.yaml
│   │   └── gitlab-pages-ingress.yaml
│   ├── load-balancer
│   │   ├── lego
│   │   │   ├── 00-namespace.yaml
│   │   │   ├── configmap.yaml
│   │   │   └── deployment.yaml
│   │   └── nginx
│   │       ├── 00-namespace.yaml
│   │       ├── configmap.yaml
│   │       ├── daemonset.yaml
│   │       ├── default-deployment.yaml
│   │       ├── default-service.yaml
│   │       ├── service.yaml
│   │       └── tcp-configmap.yaml
│   └── NOTES.txt
└── values.yaml
````




#### Перенесли содержимое директории сharts  graduation/se-deploy


### Настроим CI

```markdown
Job для выполнения каждой задачи запускается в отдельномKubernetes POD-е.

Требуемые операции вызываются в блоках script:

script:
- setup_docker
- build


Описание самих операций производится в виде bash-функций в блоке .auto_devops:


.auto_devops: &auto_devops |
function setup_docker() {
...
}
function release() {
...
}
function build() {
...
}

```

#### Создаём файл .gitlab-ci.yml с содержимым

```yaml
image: alpine:latest

stages:
  - test            # Запускается контейнер с тестами из задания после успеха запускается 
  - build           # Создаёт контейнер из докера с гитом и запускает скрипт с билдом приложений 
  - review          # Создаёт инфраструктуру для ревью на стенде разработки, запускающую приложение в k8s по коммиту в feature-бранчи (не master).
  - cleanup         # Удаляет инфраструктуру, которая была описана в review
  - production      # Выкатывает на наш кластер cluster-1 инфрастуктуру с запущенными приложениями 

test:
  stage: test
  image: python:3.6.4-jessie
  script:
    - pip install -r requirements.txt -r requirements-test.txt # устанавливаем программы указанные в зависимостях
    - python -m unittest discover -s tests/ 
    - coverage run -m unittest discover -s tests/ 
    - coverage report --include crawler/crawler.py
  only:
    - branches

build:
  stage: build
  image: docker:git
  services:
    - docker:dind
  script:
    - setup_docker
    - build
  variables:
    DOCKER_DRIVER: overlay2
  only:
    - branches

review:
  stage: review
  script:
    - install_dependencies
    - ensure_namespace
    - install_tiller
    - deploy
  variables:
    KUBE_NAMESPACE: review
    host: $CI_PROJECT_PATH_SLUG-$CI_COMMIT_REF_SLUG
  environment:
    name: review/$CI_PROJECT_PATH/$CI_COMMIT_REF_NAME
    url: http://$CI_PROJECT_PATH_SLUG-$CI_COMMIT_REF_SLUG
    on_stop: stop_review
  only:
    refs:
      - branches
    kubernetes: active

stop_review:
  stage: cleanup
  variables:
    GIT_STRATEGY: none
  script:
    - install_dependencies
    - delete
  environment:
    name: review/$CI_PROJECT_PATH/$CI_COMMIT_REF_NAME
    action: stop
  when: manual
  allow_failure: true
  only:
    refs:
      - branches
    kubernetes: active

production:
  stage: production
  script:
    - install_dependencies
    - ensure_namespace
    - install_tiller
    - deploy
  variables:
    KUBE_NAMESPACE: production
    CLUSTER_NAME: cluster-1	
    SERVICE_ACCOUNT: /etc/deploy/sa.json
    ZONE: us-central1-a
    PROJECT: vpn-project-202516
  environment:
    name: production
    url: http://production
  when: manual
  only:
    refs:
      - master
    kubernetes: active

.auto_devops: &auto_devops |
  [[ "$TRACE" ]] && set -x
  export CI_REGISTRY="index.docker.io"
  export CI_APPLICATION_REPOSITORY=$CI_REGISTRY/$CI_REGISTRY_USER/$CI_PROJECT_NAME
  export CI_APPLICATION_TAG=$CI_COMMIT_REF_SLUG
  export CI_CONTAINER_NAME=ci_job_build_${CI_JOB_ID}
  export TILLER_NAMESPACE="kube-system"
  
# Загружает Chart из репозитория se-deploy и делает релиз в неймспейсе review с образом приложения, собранным на стадии build.
  function deploy() {
    track="${1-stable}"
    name="$CI_ENVIRONMENT_SLUG"

    if [[ "$track" != "stable" ]]; then
      name="$name-$track"
    fi

    if [[ "$KUBE_NAMESPACE" == "production" ]]; then
      curl https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-201.0.0-linux-x86_64.tar.gz | tar zx
      mkdir -p /etc/deploy
      echo "Create dir /etc/deploy"
      base64 --version
      echo ${service_account} | base64 -di > ${SERVICE_ACCOUNT}
      sh ./google-cloud-sdk/bin/gcloud auth activate-service-account --key-file ${SERVICE_ACCOUNT}
      sh ./google-cloud-sdk/bin/gcloud container clusters get-credentials ${CLUSTER_NAME} --zone ${ZONE} --project ${PROJECT}
    fi

    echo "Clone deploy repository..."
    git clone http://gitlab-gitlab/graduation/se-deploy.git
    
    echo "Download helm dependencies..."
    helm dep update se-deploy/crawler-app

    echo "Deploy helm release $name to $KUBE_NAMESPACE"
    helm upgrade --install \
      --wait \
      --set $CI_PROJECT_NAME.image.tag=$CI_APPLICATION_TAG \
      --namespace="$KUBE_NAMESPACE" \
      --version="$CI_PIPELINE_ID-$CI_JOB_ID" \
      "$name" \
      se-deploy/crawler-app/
  }

  function install_dependencies() {

    apk add -U openssl curl tar gzip bash ca-certificates git coreutils python2
    wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://raw.githubusercontent.com/sgerrand/alpine-pkg-glibc/master/sgerrand.rsa.pub
    wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.23-r3/glibc-2.23-r3.apk
    apk add glibc-2.23-r3.apk
    rm glibc-2.23-r3.apk

    curl https://storage.googleapis.com/pub/gsutil.tar.gz | tar -xz -C $HOME
    export PATH=${PATH}:$HOME/gsutil

    curl https://kubernetes-helm.storage.googleapis.com/helm-v2.8.2-linux-amd64.tar.gz | tar zx
    mv linux-amd64/helm /usr/bin/
    helm version --client

    curl  -o /usr/bin/sync-repo.sh https://raw.githubusercontent.com/kubernetes/helm/master/scripts/sync-repo.sh
    chmod a+x /usr/bin/sync-repo.sh

    curl -L -o /usr/bin/kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    chmod +x /usr/bin/kubectl
    kubectl version --client
  }

  function setup_docker() {
    if ! docker info &>/dev/null; then
      if [ -z "$DOCKER_HOST" -a "$KUBERNETES_PORT" ]; then
        export DOCKER_HOST='tcp://localhost:2375'
      fi
    fi
  }

  function ensure_namespace() {
    kubectl describe namespace "$KUBE_NAMESPACE" || kubectl create namespace "$KUBE_NAMESPACE"
  }


  function build() {

    echo "Building Dockerfile-based application..."
    echo `git show --format="%h" HEAD | head -1` > build_info.txt
    echo `git rev-parse --abbrev-ref HEAD` >> build_info.txt
    docker build -t "$CI_APPLICATION_REPOSITORY:$CI_APPLICATION_TAG" .

    if [[ -n "$CI_REGISTRY_USER" ]]; then
      echo "Logging to GitLab Container Registry with CI credentials..."
      docker login -u "$CI_REGISTRY_USER" -p "$CI_REGISTRY_PASSWORD"
      echo ""
    fi

    echo "Pushing to GitLab Container Registry..."
    docker push "$CI_APPLICATION_REPOSITORY:$CI_APPLICATION_TAG"
    echo ""
  }

  function install_tiller() {
    echo "Checking Tiller..."
    helm init --upgrade
    kubectl rollout status -n "$TILLER_NAMESPACE" -w "deployment/tiller-deploy"
    if ! helm version --debug; then
      echo "Failed to init Tiller."
      return 1
    fi
    echo ""
  }

# Функция удаления окружения. Созданные для таких целей окружения временны, их требуется “убивать”,  когда они больше не нужны.
  function delete() {
    track="${1-stable}"
    name="$CI_ENVIRONMENT_SLUG"
    helm delete "$name" --purge || true
  }

before_script:
  - *auto_devops
```





##### Можем увидеть какие релизы запущены

```bash
helm ls
```

>output

```markdown
Error: incompatible versions client[v2.9.0-rc3] server[v2.7.2]
```

```bash
helm init --upgrade
```

```markdown
NAME            REVISION        UPDATED                         STATUS          CHART                   NAMESPACE
lovely-sloth    7               Sat May 19 18:33:37 2018        DEPLOYED        crawler-app-0.1.0       default  
nginx           1               Sun May 20 17:07:04 2018        DEPLOYED        nginx-ingress-0.19.2    default  
```


