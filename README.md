# Курсовая работа
Значения в /etc/hosts
35.234.78.177 gitlab-gitlab staging production
35.188.78.251 crawler-prometheus reddit-grafana reddit-non-prod production reddit-kibana
Используемая версия helm-a
# Check list
- [ ] Автоматизированные процессы создания и управления
платформой:
  - [X] Ресурсы GCP
  - [X] Инфраструктура для CI/CD 
    - Поднят GitLab на отдельном kuber кластере
    - Настроен GitLAb-Ci c авто деплоем в другой "промышленный" кубер кластер (правда настроен он для одной ветке master, оставим и будем говорить, что у нас feature toggle или добавми ветвь )
  - [ ] Инфраструктура для сбора обратной связи (как думаете, что здесь имелось ввиду? Настроенные chart с prometheus?)

- [X] Использование практики IaC (Infrastructure as Code) для управления конфигурацией и инфраструктурой 
  - Настроены Dockerfile и описаны объекты Kubernets в yaml (правда, надо описать нормаьный Service для UI, а то до него снаружи не достучаться, может можно и https добавить), управляется все это с Chart-ов Helm-a
- [X] Настроен процесс CI/CD
- [X] Все, что имеет отношение к проекту хранится в Git (Кроме логинов и  паролей)
- [ ] Настроен процесс сбора обратной связи:
  - [ ] Мониторинг
    - [ ] Сбор метрик
    - [ ] Визуализация
    - [ ] Алерты
  - [ ] Логирование (опционально)
  - [ ] Трейсинг (опционально)
  - [ ] ChatOps (опционально) 
- [ ] Документация:
  - [ ] README по работе с репозиторием
  - [ ] Описание приложения и его архитектуры
  - [ ] How to start?
  - [ ] CHANGELOG с описанием выполненной работы
  - [ ] Если работа в группе, то пункт включает автора изменений

## Простой запуск приложения в Docker контейнерах

1. Cобираем образ c тэгом `"docker build . -t asurov/name"`
2. Создаем сеть - пользовательский bridge`"docker network create --driver mynet"`
3. Запускаем контейнеры `"docker run -dit  --network kurs --name nm asurov/cntname"`, не забудем пробросить порты для `ui` приложения `"-p 80:8000"`, а также правильно проименовать сервисы `mongo` и `rabbitmq`, чтобы приложения нашли их.
4. Проверяем в браузере

## Описываем контейнеры объектами helm и kubernetes
    
При ининциализации Tiller-a имет смысл не забыть дать ему собственный ServiceAccount.
ServiceAccount описан в `tiller.yaml`, загружаем в kubernetes:
```
kubectl apply -f tiller.yml
# Инициализируем tiller:
helm init --service-account tiller
```

Для установки GitLaba нужно выполнить следующие команды:
```
# Отключить RBAC для упрощения работы.
helm install --name gitlab . -f values.yaml
# Узнать ip адрес:
kubectl get service -n nginx-ingress nginx
echo "35.234.78.177 gitlab-gitlab staging production" | sudo tee -a /etc/hosts
```

Репозиторий должен быть публичный иначе CI не сможет выгрузить chart-s для helm и надо будет [настраивать доступ по ssh](https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/ci/ssh_keys/README.md).
Чтобы сохранять контейнеры в docker-hub, необходимо указать дополнительные переменные окружения GitLab-CI
* логин --- `CI_REGISTRY_USER`
* пароль --- `CI_REGISTRY_PASSWORD`

Для деплоя в другой k8s-кластер необходимо получить `service account` GCP в `json` формате, зашифровать его в base64 и сохранить в переменной окружения `service_account` в GitLab-CI.

