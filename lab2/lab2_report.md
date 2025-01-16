University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2024/2025
Group: K4110
Author: Lysenko Aleksandra Vladimirovna
Lab: Lab2
Date of create: 16.01.2025
Date of finished: 16.01.2025

## Лабораторная работа №2 "Развертывание веб сервиса в Minikube, доступ к веб интерфейсу сервиса. Мониторинг сервиса."
### Описание

В данной лабораторной работе вы познакомитесь с развертыванием полноценного веб сервиса с несколькими репликами.

### Цель работы

Ознакомиться с типами "контроллеров" развертывания контейнеров, ознакомится с сетевыми сервисами и развернуть свое веб приложение.

### Правила по оформлению

Правила по оформлению отчета по лабораторной работе вы можете изучить по [ссылке](../reportdesign.md)


### Ход работы

- Вам необходимо создать `deployment` с 2 репликами контейнера [ifilyaninitmo/itdt-contained-frontend:master](https://hub.docker.com/repository/docker/ifilyaninitmo/itdt-contained-frontend) и передать переменные в эти реплики: `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME`.

```bash
docker pull  ifilyaninitmo/itdt-contained-frontend:master
```
![docker_pull_image.png](pictures%2Fdocker_pull_image.png)
```bash
minikube start
```
![minicube_start.png](pictures%2Fminicube_start.png)
```bash
kubectl apply -f lab2.yaml
```
![kubectl apply.png](pictures%2Fkubectl%20apply.png)
Проверим, что поды были созданы
```bash
kubectl get pods
```
![kubectl get pods.png](pictures%2Fkubectl%20get%20pods.png)

- Создать сервис через который у вас будет доступ на эти "поды". Выбор типа сервиса остается на ваше усмотрение.

Запускаем сервис
```bash
minikube kubectl -- expose deployment/lab2-deployment
```
![deployment.png](pictures%2Fdeployment.png)

Проверим, что сервис запустился
```bash
kubectl get services
```
![kubectl get services.png](pictures%2Fkubectl%20get%20services.png)

- Запустить в `minikube` режим проброса портов и подключитесь к вашим контейнерам через веб браузер.
```bash
minikube kubectl -- port-forward service/lab2-deployment 8100:8080
```


- Проверьте на странице в веб браузере переменные `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME` и `Container name`. Изменяются ли они? Если да то почему?

- Проверьте логи контейнеров, приложите логи в отчёт.