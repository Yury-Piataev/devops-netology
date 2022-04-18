Домашнее задание к занятию "4.2. Использование Python для решения типовых DevOps задач"

## Обязательная задача 1

Есть скрипт:
```python
#!/usr/bin/env python3
a = 1
b = '2'
c = a + b
```

### Вопросы:
| Вопрос  | Ответ |
| ------------- | ------------- |
| Какое значение будет присвоено переменной `c`?  |Будет ошибка, т.к. складывается строка и число: TypeError: unsupported operand type(s) |
| Как получить для переменной `c` значение 12?  | Присвоить строчное значение a=’1’ или преобразовать в вычислении c=str(a)+b |
| Как получить для переменной `c` значение 3?  | присвоить b числовое значение b=2 или преобразовать в вычислении c=a+int(b)  |

## Обязательная задача 2
Мы устроились на работу в компанию, где раньше уже был DevOps Engineer. Он написал скрипт, позволяющий узнать, какие файлы модифицированы в репозитории, относительно локальных изменений. Этим скриптом недовольно начальство, потому что в его выводе есть не все изменённые файлы, а также непонятен полный путь к директории, где они находятся. Как можно доработать скрипт ниже, чтобы он исполнял требования вашего руководителя?

```python
#!/usr/bin/env python3

import os

bash_command = ["cd ~/netology/sysadm-homeworks", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
is_change = False
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(prepare_result)
        break
```

### Ваш скрипт:
```python
#!/usr/bin/env python3

import os

print ('hi')
bash_command = ["cd /vagrant/git/Project", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
# is_change = False
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(prepare_result)
  #      break
```

### Вывод скрипта при запуске при тестировании:
```
# python git_stat.py
readme.md
devops-netology (new commits)
```

## Обязательная задача 3
1. Доработать скрипт выше так, чтобы он мог проверять не только локальный репозиторий в текущей директории, а также умел воспринимать путь к репозиторию, который мы передаём как входной параметр. Мы точно знаем, что начальство коварное и будет проверять работу этого скрипта в директориях, которые не являются локальными репозиториями.

### Ваш скрипт:
```python

#!/usr/bin/env python3

import os
import sys

cmd=os.getcwd()
if len(sys.argv)>=2:
    cmd = sys.argv[1]
bash_command = ["cd "+cmd, "git status 2>&1"]
result_os = os.popen(' && '.join(bash_command)).read()
#is_change = False
for result in result_os.split('\n'):
    if result.find('fatal')!=-1:
        print(cmd+ ' no GIT repo')
    if result.find('modified')!= -1:
        prepare_result = result.replace('\tmodified:', '')
        print(cmd+prepare_result)
#        break


```

### Вывод скрипта при запуске при тестировании:
```
root@vagrant:~# python git_stat.py
/root no GIT repo

root@vagrant:~# python git_stat.py /vagrant/git/project
/vagrant/git/project   readme.md
/vagrant/git/project   devops-netology (new commits)

```

## Обязательная задача 4
1. Наша команда разрабатывает несколько веб-сервисов, доступных по http. Мы точно знаем, что на их стенде нет никакой балансировки, кластеризации, за DNS прячется конкретный IP сервера, где установлен сервис. Проблема в том, что отдел, занимающийся нашей инфраструктурой очень часто меняет нам сервера, поэтому IP меняются примерно раз в неделю, при этом сервисы сохраняют за собой DNS имена. Это бы совсем никого не беспокоило, если бы несколько раз сервера не уезжали в такой сегмент сети нашей компании, который недоступен для разработчиков. Мы хотим написать скрипт, который опрашивает веб-сервисы, получает их IP, выводит информацию в стандартный вывод в виде: <URL сервиса> - <его IP>. Также, должна быть реализована возможность проверки текущего IP сервиса c его IP из предыдущей проверки. Если проверка будет провалена - оповестить об этом в стандартный вывод сообщением: [ERROR] <URL сервиса> IP mismatch: <старый IP> <Новый IP>. Будем считать, что наша разработка реализовала сервисы: `drive.google.com`, `mail.google.com`, `google.com`.

### Ваш скрипт:
```python
import socket as s
import time as t
import datetime as dt

i = 1
wait = 2
srv = {'drive.google.com':'64.233.161.194', 'mail.google.com':'142.251.1.19', 'google.com':'142.250.150.101'}
init=0

print(srv)

while 1==1 :
  for host in srv:
    ip = s.gethostbyname(host)
    if ip != srv[host]:
      if i==1 and init !=1:
        print(str(dt.datetime.now().strftime("%Y-%m-%d %H:%M:%S")) +' [ERROR] ' + str(host) +' IP mistmatch: '+srv[host>      srv[host]=ip

  i+=1
  if i >= 50 :
    break
  t.sleep(wait)
```

### Вывод скрипта при запуске при тестировании:
```
root@rni-nk-111023:~# python3 test_ip.py
{'drive.google.com': '64.233.161.194', 'mail.google.com': '142.251.1.19', 'google.com': '142.250.150.101'}
2022-04-18 16:01:35 [ERROR] drive.google.com IP mistmatch: 64.233.161.194 173.194.221.194
2022-04-18 16:01:35 [ERROR] mail.google.com IP mistmatch: 142.251.1.19 64.233.165.83
2022-04-18 16:01:35 [ERROR] google.com IP mistmatch: 142.250.150.101 64.233.162.138
```

