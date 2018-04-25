# graduate_workх
## Простой запуск приложения в Docker контейнерах
1. Cобераем образ c тэгом `"docker build . -t asurov/name"`
2. Создаем сеть - пользовательский bridge`"docker network create --driver mynet"`
3. Запускаем контейнеры `"docker run -dit  --network kurs --name nm asurov/cntname"`, не забудем пробросить порты для ui приложения `"-p 80:8000"`

## Описываем контейнеры в kubernete объектах 