# Домашнее задание к занятию "5.5. Оркестрация кластером Docker контейнеров на примере Docker Swarm"

---

## Задача 1

Дайте письменные ответы на следующие вопросы:

- В чём отличие режимов работы сервисов в Docker Swarm кластере: replication и global?
- Какой алгоритм выбора лидера используется в Docker Swarm кластере?
- Что такое Overlay Network?

### Ответ:
 - Global - запускает по одному экземпляру микросервиса на каждой ноде(например антивирус). replication- запускает заданное количество реплик 
 микросервисов, равномерно распределяя их по кластеру между нодами.
- Для проведения голосования и выбора лидера используется алгоритм raft.
- Overlay Network - распределенная сеть (в контексте Docker Swarm) которая создается между контейнерами для безопасного обмена информацией 
при включенном шифровании.

## Задача 2

Создать ваш первый Docker Swarm кластер в Яндекс.Облаке

Для получения зачета, вам необходимо предоставить скриншот из терминала (консоли), с выводом команды:
```
docker node ls
```
### Ответ:

```
root@vir-PC:/home/vir/1/src/terraform# ssh centos@51.250.79.137
[centos@node01 ~]$ sudo -i
[root@node01 ~]# docker node ls
ID                            HOSTNAME             STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
b5p0dfnrqrwmtcmb5avpgioqb *   node01.netology.yc   Ready     Active         Leader           20.10.17
3tp8puv3t1t32g3faczl4n643     node02.netology.yc   Ready     Active         Reachable        20.10.17
vvcg7vemw1qiupwt2o907sco7     node03.netology.yc   Ready     Active         Reachable        20.10.17
xwzjobxp2eg3vo9v51pwrqldk     node04.netology.yc   Ready     Active                          20.10.17
9u2dsi5thnav75l4ps3wa8lq1     node05.netology.yc   Ready     Active                          20.10.17
p0jgqca2uft6fyj1gpz7k4f9c     node06.netology.yc   Ready     Active                          20.10.17
[root@node01 ~]#




```

## Задача 3

Создать ваш первый, готовый к боевой эксплуатации кластер мониторинга, состоящий из стека микросервисов.

Для получения зачета, вам необходимо предоставить скриншот из терминала (консоли), с выводом команды:
```
docker service ls
```
### Ответ:
```
[root@node01 ~]# docker service ls
ID             NAME                                MODE         REPLICAS   IMAGE                                          PORTS
kyt80ngs6xtf   swarm_monitoring_alertmanager       replicated   1/1        stefanprodan/swarmprom-alertmanager:v0.14.0    
qfbebacslc7t   swarm_monitoring_caddy              replicated   1/1        stefanprodan/caddy:latest                      *:3000->3000/tcp, *:9090->9090/tcp, *:9093-9094->9093-9094/tcp
x4jubtipwp6q   swarm_monitoring_cadvisor           global       6/6        google/cadvisor:latest                         
w9ofpdqmhhz4   swarm_monitoring_dockerd-exporter   global       6/6        stefanprodan/caddy:latest                      
2rfmv0kqcluc   swarm_monitoring_grafana            replicated   1/1        stefanprodan/swarmprom-grafana:5.3.4           
xs8cm3kkymho   swarm_monitoring_node-exporter      global       6/6        stefanprodan/swarmprom-node-exporter:v0.16.0   
wx3hzjcrdhwl   swarm_monitoring_prometheus         replicated   1/1        stefanprodan/swarmprom-prometheus:v2.5.0       
rf3cinikpq4c   swarm_monitoring_unsee              replicated   1/1        cloudflare/unsee:v0.8.0                        
[root@node01 ~]#



```


## Задача 4 (*)

Выполнить на лидере Docker Swarm кластера команду (указанную ниже) и дать письменное описание её функционала, что она делает и зачем она нужна:
```
# см.документацию: https://docs.docker.com/engine/swarm/swarm_manager_locking/
docker swarm update --autolock=true
```
### Ответ: 
```
[root@node01 ~]# docker swarm update --autolock=true
Swarm updated.
To unlock a swarm manager after it restarts, run the `docker swarm unlock`
command and provide the following key:

    SWMKEY-1-BR0CKSvsLZIrwXYyX8lHTcLt9isQakzfEI/7jkjzEWM

Please remember to store this key in a password manager, since without it you
will not be able to restart the manager.
[root@node01 ~]#
```
Встроенная в docker swarm функция защиты от злоумышленников. Команда ``` docker swarm update --autolock=true ``` включает защиту уже запущенного кластера и после рестарта для разблокировки нужно будет ввести полученный ключ.
