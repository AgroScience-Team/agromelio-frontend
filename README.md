# DigitalTwin Frontend

## Главная страница

![home_page](https://github.com/AgroScience-Team/frontend/blob/new_master/docs/home_page.jpg)

## Карта

![map_page](https://github.com/AgroScience-Team/frontend/blob/new_master/docs/map_page.jpg)

## Настройки профиля

![settings_page](https://github.com/AgroScience-Team/frontend/blob/new_master/docs/settings_page.jpg)

## Как развернуть приложение для разработки?

1. Установить [docker](https://docs.docker.com/install/overview/) и [docker-compose](https://docs.docker.com/compose/install/)
   
2. На ubuntu лучше настроить [использование докера без sudo](https://askubuntu.com/questions/477551/how-can-i-use-docker-without-sudo)

3. Загрузить проект из репозитория

```sh
git clone git@github.com:AgroScience-Team/agromelio-frontend.git
``` 
4. Для сборки сервисов, описанных в конфигурационных файлах:
```sh
docker-compose build
```
5. Запуск собранных сервисов из конфигурационного файла:
```sh
docker-compose up -d
```
6. Перейти на сайт
http://localhost:9000/#/   
 
7. Чтобы завершить работу приложения в командной строке: 
```sh
docker-compose down  
``` 
