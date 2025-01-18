University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2024/2025
Group: K4110
Author: Lysenko Aleksandra Vladimirovna
Lab: Lab3
Date of create: 18.01.2025
Date of finished: 19.01.2025

## Лабораторная работа №3 "Сертификаты и "секреты" в Minikube, безопасное хранение данных."
### Описание
В данной лабораторной работе вы познакомитесь с сертификатами и "секретами" в Minikube, правилами безопасного хранения данных в Minikube.

### Цель работы
Познакомиться с сертификатами и "секретами" в Minikube, правилами безопасного хранения данных в Minikube.

### Правила по оформлению

Правила по оформлению отчета по лабораторной работе вы можете изучить по [ссылке](../reportdesign.md)


### Ход работы
```bash
minikube start
```
![minikube_start.png](pictures%2Fminikube_start.png)

- Вам необходимо создать `configMap` с переменными: `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME`.
```bash
kubectl apply -f config_map_lab3.yaml
```
![crete_config_map.png](pictures%2Fcrete_config_map.png)

- Вам необходимо создать `replicaSet` с 2 репликами контейнера [ifilyaninitmo/itdt-contained-frontend:master](https://hub.docker.com/repository/docker/ifilyaninitmo/itdt-contained-frontend) и используя ранее созданный `configMap` передать переменные `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME` .
```bash
kubectl apply -f replica_set_lab3.yaml
```
![create_replica_set.png](pictures%2Fcreate_replica_set.png)

- Включить `minikube addons enable ingress` и сгенерировать TLS сертификат, импортировать сертификат в minikube.
```bash
minikube addons enable ingress
```
![add_on_ingress.png](pictures%2Fadd_on_ingress.png)

Для генерации TLS сертификата воспользуемся openssl. openssl входит в поставку GitBash, так что нет необходимости скачивать его, необходимо лишь ребактировать переменную окружения Path(добавить C:\Program Files\Git\usr\bin\ )
Создадим ca.key с длиной 2048 бит:
```bash
openssl genrsa -out ca.key 2048
```
![openssl genrsa.png](pictures%2Fopenssl%20genrsa.png)
Узнаем MASTER-IP minicube
```bash
minikube ip
```
![minikube_ip.png](pictures%2Fminikube_ip.png)
В соответствии с ca.key сгенерируем ca.crt (используйте -days, чтобы установить время действия сертификата)
```bash
openssl req -x509 -new -nodes -key ca.key -subj "/CN=192.168.49.2" -days 10000 -out ca.crt -addext 'keyUsage = cRLSign, keyCertSign'
```
![openssl_req.png](pictures%2Fopenssl_req.png)
Создадим файл server.key с длиной 2048 бит
```bash
openssl genrsa -out server.key 2048
```
![openssl genrsa_private.png](pictures%2Fopenssl%20genrsa_private.png)

Создадим запрос на подпись сертификата на основе файла конфигурации.
```bash
openssl req -new -key server.key -out server.csr -config crs.conf
```
![подпись сертификата.png](pictures%2F%D0%BF%D0%BE%D0%B4%D0%BF%D0%B8%D1%81%D1%8C%20%D1%81%D0%B5%D1%80%D1%82%D0%B8%D1%84%D0%B8%D0%BA%D0%B0%D1%82%D0%B0.png)

И 

Создадим сертификат сервера, используя ca.key, ca.crt и server.csr.
```bash
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 10000 -extensions v3_ext -extfile crs.conf -sha256
```
ИЛИ (вместо конфигурационного файла)

```bash
kubectl -n kube-system create secret tls lab3 --key server.key --cert server.crt
```
![сертификат сервера.png](pictures%2F%D1%81%D0%B5%D1%80%D1%82%D0%B8%D1%84%D0%B8%D0%BA%D0%B0%D1%82%20%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80%D0%B0.png)

Посмотрим запрос на подпись сертификата
```bash
openssl req  -noout -text -in ./server.csr
```
![просмотр запроса на подпись.png](pictures%2F%D0%BF%D1%80%D0%BE%D1%81%D0%BC%D0%BE%D1%82%D1%80%20%D0%B7%D0%B0%D0%BF%D1%80%D0%BE%D1%81%D0%B0%20%D0%BD%D0%B0%20%D0%BF%D0%BE%D0%B4%D0%BF%D0%B8%D1%81%D1%8C.png)
![просмотр запроса на подпись_2.png](pictures%2F%D0%BF%D1%80%D0%BE%D1%81%D0%BC%D0%BE%D1%82%D1%80%20%D0%B7%D0%B0%D0%BF%D1%80%D0%BE%D1%81%D0%B0%20%D0%BD%D0%B0%20%D0%BF%D0%BE%D0%B4%D0%BF%D0%B8%D1%81%D1%8C_2.png)

- Создать ingress в minikube, где указан ранее импортированный сертификат, FQDN по которому вы будете заходить и имя сервиса который вы создали ранее.

> Если вы делаете эту работу на Windows/macOS для доступа к ingress вам необходимо использовать команду `minikube tunnel` к созданному ingress.
```bash
minikube tunnel
```
![minilube tunnel.png](pictures%2Fminilube%20tunnel.png)

> Если вы делаете эту работу на Windows/macOS для доступа к ingress вам необходимо в hosts добавить ip address localhost и ваш FQDN. Если установлен Linux, то нужно указывать minikube ip.

- В `hosts` пропишите FQDN и IP адрес вашего ingress и попробуйте перейти в браузере по FQDN имени.

- Войдите в веб приложение по вашему FQDN используя HTTPS и проверьте наличие сертификата.

> Обычно в браузере это маленький замочек рядом с FQDN сайта, нажмите на него и сделайте скриншот с информацией.

### Результаты лабораторной работы

- Файлы с разработанными вами манифестами с расширением `.yaml`.

- Схема организации контейеров и сервисов нарисованная вами в [draw.io](https://app.diagrams.net) или Visio.

- Скриншоты c результатами работы.
