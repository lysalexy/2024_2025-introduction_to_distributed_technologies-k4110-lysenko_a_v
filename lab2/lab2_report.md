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
kubectl apply -f service_lab2.yaml
```
![deployment.png](pictures%2Fdeployment.png)

Проверим, что сервис запустился
```bash
kubectl get services
```
![kubectl get services.png](pictures%2Fkubectl%20get%20services.png)

Можем посмотреть описание сервиса
![describe_service.png](pictures%2Fdescribe_service.png)

Можем посмотреть url сервиса
![service_url.png](pictures%2Fservice_url.png)


!!! Точкой стыковки сервиса и подов являются поля service.spec.selector.app и deployment.spec.template.metadata.labels.app, так что их значения должны совпадать

- Запустить в `minikube` режим проброса портов и подключитесь к вашим контейнерам через веб браузер.
```bash
minikube kubectl -- port-forward service/lab2 3000:3000
```
![port-forward.png](pictures%2Fport-forward.png)

- Проверьте на странице в веб браузере переменные `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME` и `Container name`. Изменяются ли они? Если да то почему?
![frontend_view.png](pictures%2Ffrontend_view.png)
  Services and pods are matched matching service's spec.selector.app to pod's app label.
- Проверьте логи контейнеров, приложите логи в отчёт.
```bash
>kubectl logs lab2-deployment-7d5b675df8-8ftkw
```
![pod_log.png](pictures%2Fpod_log.png)
```bash
>kubectl describe pod lab2-deployment-7d5b675df8-8ftkw
```
![describe_pod_1.png](pictures%2Fdescribe_pod_1.png)
![describe_pod_2.png](pictures%2Fdescribe_pod_2.png)