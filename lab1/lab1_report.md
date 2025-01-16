University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2024/2025
Group: K4110
Author: Lysenko Aleksandra Vladimirovna
Lab: Lab1
Date of create: 11.01.2025
Date of finished: 16.01.2025

## Лабораторная работа №1 "Установка Docker и Minikube, мой первый манифест."
### Описание
Это первая лабораторная работа в которой вы сможете протестировать Docker, установить Minikube и развернуть свой первый "под".

### Цель работы
Ознакомиться с инструментами Minikube и Docker, развернуть свой первый "под".

### Правила по оформлению

Правила по оформлению отчета по лабораторной работе вы можете изучить по [ссылке](../reportdesign.md)

### Ход работы
Данную лабораторную работу рекомендуется начать с изучения [Документация по Minikube](https://minikube.sigs.k8s.io/docs/), эта статья поможет вам в первоначальном представлении о инструменте Minikube.

Перед выполнением лабораторной работы вам необходимо выполнить следующие задачи:

> Работу можно проводить на устройстве с архитектурой x86/arm64/ARMv7, [оригинальная инструкция](https://minikube.sigs.k8s.io/docs/start/)

- Установить Docker на рабочий компьютер
Docker был установлен на Window c помощью [оригинальной инструкции](https://docs.docker.com/engine/install/)

- Установить Minikube используя [оригинальную инструкцию](https://minikube.sigs.k8s.io/docs/start/)
Minikube был установлен

- После установки вам необходимо развернуть minikube cluster

```bash
minikube start
```
![minikube_start.png](pictures%2Fminikube_start.png)
- После запуска minikube cluster вы сможете взаимодействовать с k8s используя команду:

```bash
minikube kubectl
```
![minikube_kubectl.png](pictures%2Fminikube_kubectl.png)

> Использование `minikube kubectl` необходимо если в вашей системе не был установлен `kubectl` (в инструкции по установке minikube это не предусмотрено), `kubectl` является инструментом по управлению обычного k8s кластера и устанавливается отдельно. Для удобства использования вы можете создать алиас `alias kubectl="minikube kubectl --"` или аналогичный на свое усмотрение, но это не обязательно.

- Для первого манифеста мы выбрали образ HashiCorp Vault, более подробно можете почитать [тут](https://www.vaultproject.io). Вам нужно будет написать манифест для развертывания "пода" HashiCorp Vault, и при этом прокинуть внутрь порт **8200**

> ВАЖНО! Вам не надо самим собирать контейнер, вы можете его взять [тут](https://hub.docker.com/_/vault/)

Первоначально была предпринята попытка скачивания образа контейнера с помощью команды, указанной в docker hub
```bash
docker pull vault
```

Однако, данная попытка не увенчалась успехом
![docker_pull_vault_fail.png](pictures%2Fdocker_pull_vault_fail.png)

С помощью Docker Desktop был найден образ, с важным указанием, что с версии 11.4 образ не будет выкладываться в DockerHub
![vault_dockerhub_image_deprication.png](pictures%2Fvault_dockerhub_image_deprication.png)

Загрузка образа, найденного с помощью Docker Desktop зависла, и образ был скачан с помощью команды

```bash
docker pull hashicorp/vault
```
![docker_pull_hashicorp_vault.png](pictures%2Fdocker_pull_hashicorp_vault.png)

- После этого вам необходимо будет создать сервис для доступа к этому контейнеру, самый просто вариант - это выполнить команду:

Для пода был написан манифест, приложенный к тексту отчета. Для этого необходимо перейти в директорию хранения манифеста и указать неймспейс при необходимости
```bash
kubectl apply -f myFirst.yaml
```
![go to dir with manifest+create namespace.png](pictures%2Fgo%20to%20dir%20with%20manifest%2Bcreate%20namespace.png)

Под был запущен в указанном неймспейсе
```bash
minikube kubectl -- expose pod vault  --namespace=first --type=NodePort --port=8200
```
![pod status in namespace.png](pictures%2Fpod%20status%20in%20namespace.png)

Далее была предпринята попытка изменения неймспейса пода, преведшая в ошибке
![change namespace in pod and get error.png](pictures%2Fchange%20namespace%20in%20pod%20and%20get%20error.png)

Существующий под был удален и вместо него был создан новый с неймспейсом default
```bash
delete pod vault -n first --grace-period=0 --force
```
![delete pod in wrong namespace and deploy.png](pictures%2Fdelete%20pod%20in%20wrong%20namespace%20and%20deploy.png)

> Эта команда будет работать только если ваш "под" имеет имя `vault`

```bash
minikube kubectl -- expose pod vault --type=NodePort --port=8200
```
![deploy statuses.png](pictures%2Fdeploy%20statuses.png)
- Команда выше создаст сервис, но как же вам попасть на ваш контейнер? Воспользуйтесь следующей командой:

```bash
minikube kubectl -- port-forward service/vault 8200:8200
```

- minikube прокинет порт вашего компьютера в контейнер и вы сможете зайти в vault по ссылке [http://localhost:8200](http://localhost:8200)

- После перехода по ссылке у открылся интерфейс 

![vault deployed.png](pictures%2Fvault%20deployed.png)

- Для успешного завершения лабораторной работы вам необходимо войти в ваш vault ипользуя токен, который вам необходимо НАЙТИ, а не сгенерировать.

Для получения токена аутентификации было прочитано содержимое лога Vault и найдено значение токена для пользователя Root
```bash
kubectl logs vault
```
![kubectl logs vault.png](pictures%2Fkubectl%20logs%20vault.png)
![kubectl logs vault logs auth token.png](pictures%2Fkubectl%20logs%20vault%20logs%20auth%20token.png)

После введения токена была пройдена авторизация
![vault_successful_auth.png](pictures%2Fvault_successful_auth.png)

- Для остановки minikube cluster вы можете воспользоваться командой

```bash
minikube stop
```
