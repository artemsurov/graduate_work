# graduate_workх
## Простой запуск приложения в Docker контейнерах
1. Cобираем образ c тэгом `"docker build . -t asurov/name"`
2. Создаем сеть - пользовательский bridge`"docker network create --driver mynet"`
3. Запускаем контейнеры `"docker run -dit  --network kurs --name nm asurov/cntname"`, не забудем пробросить порты для ui приложения `"-p 80:8000"`, а также правильно проименовать сервисы mongo и rabitmq, чтобы приложения нашли их.
4. Проверяем в браузере
## Описываем контейнеры в kubernete объектах 
    

При ининциализации Tiller-a имет смысл не забывать дать ему собственный ServiceAccount.
ServiceAccount описан в tiller.yaml, загружаем в kubernetes:
kubectl apply -f tiller.yml
Инициализируем tiller:
helm init --service-account tiller

Для установки gitlaba нужно выполнить следующие команды:
Отключить RBAC для упрощения работы.
helm install --name gitlab . -f values.yaml
Узнать ip адрес:
kubectl get service -n nginx-ingress nginx
echo "35.234.78.177 gitlab-gitlab staging production” | sudo tee -a /etc/hosts
