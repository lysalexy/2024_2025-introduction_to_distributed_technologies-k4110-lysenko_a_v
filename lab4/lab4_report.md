University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2024/2025
Group: K4110
Author: Lysenko Aleksandra Vladimirovna
Lab: Lab4
Date of create: 19.01.2025
Date of finished: 19.01.2025

## Лабораторная работа №4 "Сети связи в Minikube, CNI и CoreDNS"
### Описание

Это последняя лабораторная работа в которой вы познакомитесь с сетями связи в Minikube. Особенность Kubernetes заключается в том, что у него одновременно работают `underlay` и `overlay`  сети, а управление может быть организованно различными CNI.

### Цель работы

Познакомиться с CNI Calico и функцией `IPAM Plugin`, изучить особенности работы CNI и CoreDNS.

### Правила по оформлению

Правила по оформлению отчета по лабораторной работе вы можете изучить по [ссылке](../reportdesign.md)

### Ход работы

- При запуске minikube установите плагин `CNI=calico` и режим работы `Multi-Node Clusters` одновеременно, в рамках данной лабораторной работы вам нужно развернуть 2 ноды.

> Оригинальная инструкция для установки Calico в Minikube [ссылка](https://projectcalico.docs.tigera.io/getting-started/kubernetes/minikube)
```bash
minikube start --network-plugin=cni
```
![minikube start.png](pictures%2Fminikube%20start.png)

> Оригинальная инструкция для включение 2-ух нод в Minikube [ссылка](https://minikube.sigs.k8s.io/docs/tutorials/multi_node/)
```bash
minikube start --nodes 2 -p multinode-demo
```
![minikube start 2.png](pictures%2Fminikube%20start%202.png)

- Проверьте работу CNI плагина Calico и количество нод, результаты проверки приложите в отчет.

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.1/manifests/calico.yaml
```
![kubectl apply calico.png](pictures%2Fkubectl%20apply%20calico.png)

```bash
kubectl get pods -l k8s-app=calico-node -A
```
![kubectl get pods.png](pictures%2Fkubectl%20get%20pods.png)
```bash
kubectl get pod -o=custom-columns=NODE:.spec.nodeName,NAME:.metadata.name --all-namespaces
```
![two node.png](pictures%2Ftwo%20node.png)

- Для проверки работы Calico мы попробуем одну из функций под названием `IPAM Plugin`.

- Установите Calicoctl как плагин kubectl на одном хосте.
```bash
Invoke-WebRequest -Uri "https://github.com/projectcalico/calico/releases/download/v3.29.1/calicoctl-windows-amd64.exe" -OutFile kubectl-calico.exe
```
![Invoke-WebRequest.png](pictures%2FInvoke-WebRequest.png)

```bash
kubectl calico -h
```
![kubectl calico.png](pictures%2Fkubectl%20apply%20calico.png)

ИЛИ

Установите Calicoctl как двоичный файл на одном хосте
```bash
Invoke-WebRequest -Uri "https://github.com/projectcalico/calico/releases/download/v3.29.1/calicoctl-windows-amd64.exe" -OutFile "calicoctl.exe"
```
![Invoke-WebRequest 2.png](pictures%2FInvoke-WebRequest%202.png)

- Для проверки режима `IPAM` необходимо для запущеных ранее нод указать `label` по признаку стойки или географического расположения (на ваш выбор).

```bash
kubectl label nodes multinode-demo  rack=0
kubectl label nodes multinode-demo-m02  rack=1
```
![label nodes.png](pictures%2Flabel%20nodes.png)
> Оригинальная инструкция для назначения IP адресов в Calico [ссылка](https://projectcalico.docs.tigera.io/networking/assign-ip-addresses-topology)
```bash
calicoctl create -f rack_1.yaml
```
![failed to create.png](pictures%2Ffailed%20to%20create.png)
В случае возникновения ошибки необходимо создать файл с конфигурацией и передавать его при вызове функции
```bash
calicoctl create -f rack_1.yaml --config=calico_cfg.yaml
```
![use config.png](pictures%2Fuse%20config.png)

```bash
calicoctl create -f rack_2.yaml --config=calico_cfg.yaml
```
![create second pool.png](pictures%2Fcreate%20second%20pool.png)

```bash
calicoctl get ippool -o wide --config=calico_cfg.yaml
```
![get_pools.png](pictures%2Fget_pools.png)

- После этого вам необходимо разработать манифест для Calico который бы на основе ранее указанных меток назначал бы IP адреса "подам" исходя из пулов IP адресов которые вы указали в манифесте.

```bash
kubectl apply -f ip_to_labels.yaml
```
![ip_to_labels.png](pictures%2Fip_to_labels.png)

- Вам необходимо создать `deployment` с 2 репликами контейнера [ifilyaninitmo/itdt-contained-frontend:master](https://hub.docker.com/repository/docker/ifilyaninitmo/itdt-contained-frontend) и передать переменные в эти реплики: `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME`.

```bash
kubectl apply -f config_map_lab4.yaml
kubectl apply -f lab4.yaml
```
![deployment.png](pictures%2Fdeployment.png)

```bash
kubectl get events --sort-by=".metadata.managedFields[0].time"
```
![pods events.png](pictures%2Fpods%20events.png)

При зависании подов в статусе "ContainerCreating" [из-за зависания событий пулла образа контейнера](https://serverfault.com/questions/728727/kubernetes-stuck-on-containercreating) необходимо внести изменения в конфигурацию DockerEngine и перезагрузить его
![edit docker engine conf.png](pictures%2Fedit%20docker%20engine%20conf.png)

- Создать сервис через который у вас будет доступ на эти "поды". Выбор типа сервиса остается на ваше усмотрение.

```bash
kubectl apply -f service_lab4.yaml
```
![service.png](pictures%2Fservice.png)

```bash
kubectl get all,cm,secret,ing -A
```
![all_data.png](pictures%2Fall_data.png)
![all_data_2.png](pictures%2Fall_data_2.png)

```bash
minikube service list -p multinode-demo
```
![service list in claster.png](pictures%2Fservice%20list%20in%20claster.png)

```bash
minikube service lab4 -p multinode-demo
```
![view url of service from claster.png](pictures%2Fview%20url%20of%20service%20from%20claster.png)

![frontend.png](pictures%2Ffrontend.png)
- 
- Запустить в `minikube` режим проброса портов и подключитесь к вашим контейнерам через веб браузер.

```bash
minikube kubectl -- port-forward service/lab4 3000:3000
```

- Используя `kubectl exec` зайдите в любой "под" и попробуйте попинговать "поды" используя `FQDN` имя соседенего "пода", результаты пингов необходимо приложить к отчету.
```bash 
kubectl exec pod/lab4-867f9789b-tt6k6 -- ping 10.244.239.7
```
![ping another pod.png](pictures%2Fping%20another%20pod.png)