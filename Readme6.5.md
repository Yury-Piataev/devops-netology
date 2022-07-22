# Домашнее задание к занятию "6.5. Elasticsearch"

## Задача 1

В этом задании вы потренируетесь в:
- установке elasticsearch
- первоначальном конфигурировании elastcisearch
- запуске elasticsearch в docker

Используя докер образ [centos:7](https://hub.docker.com/_/centos) как базовый и
[документацию по установке и запуску Elastcisearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html):

- составьте Dockerfile-манифест для elasticsearch
- соберите docker-образ и сделайте `push` в ваш docker.io репозиторий
- запустите контейнер из получившегося образа и выполните запрос пути `/` c хост-машины

Требования к `elasticsearch.yml`:
- данные `path` должны сохраняться в `/var/lib`
- имя ноды должно быть `netology_test`

В ответе приведите:
- текст Dockerfile манифеста
- ссылку на образ в репозитории dockerhub
- ответ `elasticsearch` на запрос пути `/` в json виде

Подсказки:
- возможно вам понадобится установка пакета perl-Digest-SHA для корректной работы пакета shasum
- при сетевых проблемах внимательно изучите кластерные и сетевые настройки в elasticsearch.yml
- при некоторых проблемах вам поможет docker директива ulimit
- elasticsearch в логах обычно описывает проблему и пути ее решения

### Ответ:
```
elasticsearch.yml

cluster.name: "netology"
path:
  data: /var/lib/data
  logs: /var/lib/logs
node.name: netology_test
transport.host: 127.0.0.1
http.host: 0.0.0.0



Dockerfile: 
from centos:7
LABEL piataevyus <piataevyus@example.com>

RUN yum install wget -y
RUN yum install -y java-11-openjdk wget curl perl-Digest-SHA

RUN wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.17.3-linux-x86_64.tar.gz
RUN mkdir /usr/elasticsearch && \
        mv elasticsearch-7.17.3-linux-x86_64.tar.gz /usr/elasticsearch && \
        cd /usr/elasticsearch && \
        tar -xzf elasticsearch-7.17.3-linux-x86_64.tar.gz

COPY elasticsearch.yml /usr/elasticsearch/elasticsearch-7.17.3/config/elasticsearch.yml

RUN groupadd elasticsearch && \
        useradd elasticsearch -g elasticsearch -p elasticsearch && \
        chown -R elasticsearch:elasticsearch /usr/elasticsearch

RUN mkdir /var/lib/data && \
        chown -R elasticsearch:elasticsearch /var/lib/data && \
        mkdir /var/lib/logs && \
        chown -R elasticsearch:elasticsearch /var/lib/logs

USER elasticsearch

CMD ["/usr/sbin/init"]
CMD ["/usr/elasticsearch/elasticsearch-7.17.3/bin/elasticsearch"]
```
Собрать образ:
```docker build -t piataevyus/elasticsearch:0.0.1 .```

Запуск контейнера из собранного образа:
```docker run --name elastic -it -p9200:9200 -d piataevyus/elasticsearch:0.0.1```

Ссылка не репозиторий: 
``` https://hub.docker.com/repository/docker/piataevyus/elasticsearch ```

```
root@vir-PC:~# curl -X GET http://localhost:9200/
{
  "name" : "netology_test",
  "cluster_name" : "netology",
  "cluster_uuid" : "4xQqLVLtRB6R-W46akRZyA",
  "version" : {
    "number" : "7.17.3",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "5ad023604c8d7416c9eb6c0eadb62b14e766caff",
    "build_date" : "2022-04-19T08:11:19.070913226Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}



```



## Задача 2

В этом задании вы научитесь:
- создавать и удалять индексы
- изучать состояние кластера
- обосновывать причину деградации доступности данных

Ознакомтесь с [документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html)
и добавьте в `elasticsearch` 3 индекса, в соответствии с таблицей:

| Имя | Количество реплик | Количество шард |
|-----|-------------------|-----------------|
| ind-1| 0 | 1 |
| ind-2 | 1 | 2 |
| ind-3 | 2 | 4 |

Получите список индексов и их статусов, используя API и **приведите в ответе** на задание.

Получите состояние кластера `elasticsearch`, используя API.

Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?

Удалите все индексы.

**Важно**

При проектировании кластера elasticsearch нужно корректно рассчитывать количество реплик и шард,
иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

### Ответ:
```
Создание индексов:
# curl -X PUT localhost:9200/ind-1 -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 1,  "number_of_replicas": 0 }}'
# curl -X PUT localhost:9200/ind-2 -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 2,  "number_of_replicas": 1 }}'
# curl -X PUT localhost:9200/ind-3 -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 4,  "number_of_replicas": 2 }}'

Список индексов:
root@vir-PC:~# curl -X GET http://localhost:9200/_cat/indices?v
health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .geoip_databases Cc05FPB9RDeE414cLdvOrw   1   0         40            0     37.8mb         37.8mb
green  open   ind-1            0x4UH_ZRSdSyLu6JsGyDFg   1   0          0            0       226b           226b
yellow open   ind-3            Gt2UwANdSCyXm2e9BtOoiQ   4   2          0            0       904b           904b
yellow open   ind-2            a64N1nbyTIKKymJfmUnghg   2   1          0            0       452b           452b

Статус индекса ind-1: 
root@vir-PC:~# curl -X GET http://localhost:9200/_cluster/health/ind-1?pretty
{
  "cluster_name" : "netology",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 1,
  "active_shards" : 1,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}

Статус индекса ind-2: 
root@vir-PC:~# curl -X GET http://localhost:9200/_cluster/health/ind-2?pretty
{
  "cluster_name" : "netology",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 2,
  "active_shards" : 2,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 2,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 50.0
}

Статус индекса ind-3:
root@vir-PC:~# curl -X GET http://localhost:9200/_cluster/health/ind-3?pretty
{
  "cluster_name" : "netology",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 4,
  "active_shards" : 4,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 8,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 50.0
}

Статус кластера:
root@vir-PC:~# curl -XGET localhost:9200/_cluster/health/?pretty
{
  "cluster_name" : "netology",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 10,
  "active_shards" : 10,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 10,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 50.0
}

Часть индексов и кластер в состоянии yellow, потому что поднята всего одна нода, а в настройках индекса указано количество реплик - реплицировать некуда.

Удаление индексов:
root@vir-PC:~# curl -X DELETE http://localhost:9200/ind-1?pretty
{
  "acknowledged" : true
}
root@vir-PC:~# curl -X DELETE http://localhost:9200/ind-2?pretty
{
  "acknowledged" : true
}
root@vir-PC:~# curl -X DELETE http://localhost:9200/ind-3?pretty
{
  "acknowledged" : true
}

```

## Задача 3

В данном задании вы научитесь:
- создавать бэкапы данных
- восстанавливать индексы из бэкапов

Создайте директорию `{путь до корневой директории с elasticsearch в образе}/snapshots`.

Используя API [зарегистрируйте](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository)
данную директорию как `snapshot repository` c именем `netology_backup`.

**Приведите в ответе** запрос API и результат вызова API для создания репозитория.

Создайте индекс `test` с 0 реплик и 1 шардом и **приведите в ответе** список индексов.

[Создайте `snapshot`](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html)
состояния кластера `elasticsearch`.

**Приведите в ответе** список файлов в директории со `snapshot`ами.

Удалите индекс `test` и создайте индекс `test-2`. **Приведите в ответе** список индексов.

[Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние
кластера `elasticsearch` из `snapshot`, созданного ранее.

**Приведите в ответе** запрос к API восстановления и итоговый список индексов.

Подсказки:
- возможно вам понадобится доработать `elasticsearch.yml` в части директивы `path.repo` и перезапустить `elasticsearch`

### Ответ:
```
Доработал .yml, доработал dockerfile, пересобрал образ.

Регистрация репозитория:
root@vir-PC:~/dz/6.5# curl -XPUT http://localhost:9200/_snapshot/netology_backup?pretty -H 'content-type: application/json' -d'{ "type": "fs", "settings": { "location": "/usr/elasticsearch/elasticsearch-7.17.3/snapshots"}}'
{
  "acknowledged" : true
}

root@vir-PC:~/dz/6.5# curl -XGET http://localhost:9200/_snapshot/netology_backup?pretty
{
  "netology_backup" : {
    "type" : "fs",
    "settings" : {
      "location" : "/usr/elasticsearch/elasticsearch-7.17.3/snapshots"
    }
  }
}

root@vir-PC:~/dz/6.5# curl -XPUT http://localhost:9200/test -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 1,  "number_of_replicas": 0 }}'

root@vir-PC:~/dz/6.5# curl -XGET http://localhost:9200/_cat/indices?v
health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .geoip_databases r8hQq51LRT6xj0qLUMsrxQ   1   0         40            0     37.8mb         37.8mb
green  open   test             O7mDoRAIS6y3Hful6r6XtA   1   0          0            0       226b           226b

Создание snapshot:
curl -XPUT http://localhost:9200/_snapshot/netology_backup/snapshot?wait_for_completion=true

root@vir-PC:~/dz/6.5# curl -XGET 'http://localhost:9200/_snapshot?pretty'
{
  "netology_backup" : {
    "type" : "fs",
    "uuid" : "nTJhO1woThmZTnq5mSQjcw",
    "settings" : {
      "location" : "/usr/elasticsearch/elasticsearch-7.17.3/snapshots"
    }
  }
}

[elasticsearch@a885750a67b5 snapshots]$ ll
total 48
-rw-r--r-- 1 elasticsearch elasticsearch  1420 Jul 17 17:31 index-0
-rw-r--r-- 1 elasticsearch elasticsearch     8 Jul 17 17:31 index.latest
drwxr-xr-x 6 elasticsearch elasticsearch  4096 Jul 17 17:31 indices
-rw-r--r-- 1 elasticsearch elasticsearch 29281 Jul 17 17:31 meta-kpITT9eqSjSIEQh5yB4Ztg.dat
-rw-r--r-- 1 elasticsearch elasticsearch   707 Jul 17 17:31 snap-kpITT9eqSjSIEQh5yB4Ztg.dat

Удаление индекса test:
root@vir-PC:~/dz/6.5# curl -X DELETE http://localhost:9200/test?pretty

Создание индекса test-2:
root@vir-PC:~/dz/6.5# curl -X PUT localhost:9200/test-2?pretty -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 1,  "number_of_replicas": 0 }}'

root@vir-PC:~/dz/6.5# curl -XGET http://localhost:9200/_cat/indices?v
health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .geoip_databases r8hQq51LRT6xj0qLUMsrxQ   1   0         40            0     37.8mb         37.8mb
green  open   test-2           QnF4c1dBT4iGI8rkY-6-6A   1   0          0            0       226b           226b



Восстановление snapshot:
root@vir-PC:~/dz/6.5# curl -X POST localhost:9200/_snapshot/netology_backup/snapshot/_restore?pretty -H 'Content-Type: application/json' -d'{"indices": "test"}'
{
  "accepted" : true
}

root@vir-PC:~/dz/6.5# curl -XGET http://localhost:9200/_cat/indices?v
health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .geoip_databases r8hQq51LRT6xj0qLUMsrxQ   1   0         40           66     70.4mb         70.4mb
green  open   test-2           QnF4c1dBT4iGI8rkY-6-6A   1   0          0            0       226b           226b
green  open   test             eRZng7FFRjiEduhHE1vanw   1   0          0            0       226b           226b
```
---
