
```markdown
# Ответы на контрольные вопросы по лабораторным работам

## Лабораторная работа № 4.1 – Использование PKI в сервисах OpenVPN

### 1. Какие пакеты разрешают приведённые строки iptables?
```bash
iptables -I INPUT -p tcp -m state --state ESTABLISHED -j ACCEPT
iptables -I INPUT -p icmp -m state --state ESTABLISHED -j ACCEPT
iptables -I INPUT -p udp -m state --state ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p icmp -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p udp -m state --state NEW,ESTABLISHED -j ACCEPT
```

Ответ:

· В цепочке INPUT разрешены входящие пакеты протоколов TCP, UDP и ICMP только для уже установленных соединений (состояние ESTABLISHED). Новые входящие соединения не пропускаются.
· В цепочке OUTPUT разрешены исходящие пакеты TCP, UDP, ICMP как для новых (NEW), так и для уже установленных соединений (ESTABLISHED). Это позволяет хосту инициировать любые исходящие соединения, а также отвечать на входящие запросы, не нарушая установленную сессию.

2. Какие порты и протоколы необходимо разрешить в цепочке INPUT межсетевого экрана на ВМ astra-1.7, чтобы появилась возможность подключения OpenVPN и прохождения ICMP-пакетов?

Ответ:
Необходимо разрешить:

· UDP пакеты, адресованные на порт 1194 (состояния NEW, ESTABLISHED);
· ICMP пакеты типа echo-request (или весь протокол ICMP) в состояниях NEW, ESTABLISHED.

Пример правил:

```bash
iptables -I INPUT -p udp --dport 1194 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -I INPUT -p icmp --icmp-type echo-request -m state --state NEW,ESTABLISHED -j ACCEPT
```

---

Лабораторная работа № 5 – Защита от руткитов и сетевых вторжений

В данной работе отсутствуют явные текстовые вопросы для письменного ответа. Результатом являются скриншоты вывода команд.
Тем не менее, приведены краткие пояснения ключевых моментов.

1. Rkhunter

· Утилита rkhunter проверяет наличие руткитов, уязвимостей и изменений системных файлов.
· При появлении Warning в логе /var/log/rkhunter.log необходимо проанализировать природу предупреждения (часто это ложные срабатывания из-за обновлений или кастомной конфигурации).

2. Snort

· Правила Snort в local.rules используют синтаксис: alert <протокол> <источник> -> <назначение> (msg:"..."; sid:...; rev:...; дополнительные опции).
· В лабораторной работе сканирования моделировались командой nmap с различными флагами (-sT, -sX, -sF, -sU, -sP), что вызывало соответствующие алерты в консоли Snort.

3. Fail2ban

· Конфигурация блокировки: bantime = 5m, findtime = 10m, maxretry = 3 — означает, что IP блокируется на 5 минут после 3 неудачных попыток в течение 10 минут.
· Проверить забаненные IP: fail2ban-client status sshd или fail2ban-client get sshd banned.
· Просмотр правил iptables, созданных fail2ban: iptables -L -n | grep <IP>.

4. Безопасная настройка SSH

· Отключение парольной аутентификации: PasswordAuthentication no.
· Запрет прямого входа root: PermitRootLogin no.
· Ограничение числа попыток: MaxAuthTries 3.
· Использование только протокола 2.
· Задание стойких алгоритмов шифрования (Ciphers, MACs, KexAlgorithms).
· Аутентификация по ключам требует правильных прав:
  · ~/.ssh — 700,
  · ~/.ssh/authorized_keys — 600,
  · владелец — пользователь.

5. Lynis

· Lynis выполняет аудит безопасности системы и даёт рекомендации.
· Индекс рекомендации (например, SSH-7408) можно изучить командой lynis show details SSH-7408.
· Для улучшения оценки необходимо применить предложенные изменения в конфигурации SSH и других сервисов, после чего выполнить повторный аудит.

```
```













echo "deb https://packages.cisofy.com/community/lynis/deb/ stable main" | sudo tee /etc/apt/sources.list.d/cisofy-lynis.list



wget -O - https://packages.cisofy.com/keys/cisofy-software-public.key | sudo apt-key add -


# Создать .ssh, если нет
mkdir -p /home/astra/.ssh
chmod 700 /home/astra/.ssh

# Добавить публичный ключ в authorized_keys
cat /home/astra/.ssh/sa_key.pub > /home/astra/.ssh/authorized_keys
chmod 600 /home/astra/.ssh/authorized_keys
chown -R astra:astra /home/astra/.ssh






cat <<EOF >> /etc/ssh/sshd_config
PasswordAuthentication no
PermitRootLogin no
PermitUserEnvironment no
IgnoreRhosts yes
MaxAuthTries 3
Ciphers aes128-ctr,aes192-ctr,aes256-ctr,arcfour256,arcfour128,aes128-cbc,3des-cbc,grasshopper-ctr
MACs hmac-md5,hmac-sha1,umac-64@openssh.com,hmac-ripemd160,hmac-gost2012-256-etm
KexAlgorithms curve25519-sha256,ecdh-sha2-nistp384,ecdh-sha2-nistp256,diffie-hellman-group-exchange-sha256
ClientAliveInterval 300
Protocol 2
EOF








cat <<EOF > /etc/fail2ban/jail.local
[DEFAULT]
bantime = 5m
findtime = 10m
maxretry = 3
EOF



for i in {1..4}; do sshpass -p wrongpass ssh -o StrictHostKeyChecking=no testuser@127.0.0.1 exit; done



fail2ban-client banned
iptables -L | grep 127.0.0.1





apt-get install nmap -y
nmap -sT -p22 127.0.0.1
nmap -sX -p22 127.0.0.1
nmap -sF -p22 127.0.0.1
nmap -sU -p22 127.0.0.1
nmap -sP 127.0.0.1 --disable-arp-ping
ping -c 1 127.0.0.1










cat <<EOF > /etc/snort/rules/local.rules
alert tcp any any -> 127.0.0.1 any (msg: "TCP Scan"; sid:10000005; rev:2; )
alert icmp any any -> 127.0.0.1 any (msg: "ICMP Scan"; dsize:0; sid:10000004; rev:1;)
alert tcp any any -> 127.0.0.1 22 (msg:"SSH XMAS Tree Scan"; flags:FPU; sid:1000006; rev:1; )
alert tcp any any -> 127.0.0.1 22 (msg:"SSH FIN Scan"; flags:F; sid:1000008; rev:1;)
alert udp any any -> any any ( msg:"UDP Scan"; sid:1000010; rev:1; )
EOF



# Домашнее задание к занятию `«GitLab»` - `Тимохин Максим`

https://drive.google.com/file/d/1T8RVOnbY0zDVyIKhufB5LB-g2Y3UWGKM/view?usp=sharing

### Задание 1

**Что нужно сделать:**

1. Разверните GitLab локально, используя Vagrantfile и инструкцию, описанные в [этом репозитории](https://github.com/netology-code/sdvps-materials/tree/main/gitlab).   
2. Создайте новый проект и пустой репозиторий в нём.
3. Зарегистрируйте gitlab-runner для этого проекта и запустите его в режиме Docker. Раннер можно регистрировать и запускать на той же виртуальной машине, на которой запущен GitLab.

В качестве ответа в репозиторий шаблона с решением добавьте скриншоты с настройками раннера в проекте.

---

> Примечание: вариант с Vagrant не был использован из-за проблем с вложенными виртуальными машинами (`VT-x is not available (VERR_VMX_NO_VMX)`). Принято решение поднять GitLab на хостовом VirtualBox с VM Ubuntu, что принципиально не противоречит условиям задания.

## Среда

- **Хост:** Windows 10 Home  
- **VM:** Ubuntu (VirtualBox)  
- **GitLab CE:** локально на VM (`http://192.168.1.88`)  
- **GitLab Runner:** Docker executor на той же VM  
- **Репозиторий:** `test-project-1`  

---

1. Установка и запуск GitLab



```
sudo apt update && sudo apt upgrade -y
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
sudo EXTERNAL_URL="http://192.168.1.88" apt install -y gitlab-ce
```

Далее 

GitLab развернут, веб-интерфейс доступен по адресу:  

с гостевой ОС: [http://localhost](http://localhost)

с хоста: [http://192.168.1.88](http://192.168.1.88)

Пароль для первого входа:

```
sudo cat /etc/gitlab/initial_root_password
```

2. Создание проекта
   
В GitLab веб-интерфейсе:

`Create new project -> Blank project`

Создан пустой проект в GitLab: `Test project 1`

3. Регистрация GitLab Runner

Использую контейнер Runner:

```
sudo docker exec -it gitlab-runner gitlab-runner register
```

GitLab Runner зарегистрирован с параметрами:

- **GitLab instance URL:** `http://192.168.1.88`  
- **Registration token:** (GR1234567890vOOdo-LV8BuGaGaW)  
- **Description:** `docker-runner`  
- **Tags:** `docker,local,ci`  
- **Maintenance note:**   
- **Executor:** `docker`  
- **Default Docker image:** `alpine:latest`  

Runner появился в GitLab -> Project -> CI/CD Settings -> Runners 

## Скриншот с настройками раннера

<img width="914" height="736" alt="Capture_runner_3" src="https://github.com/user-attachments/assets/1238a4ed-0424-4a5e-8ed1-dc06c533dc27" />


---

### Задание 2

**Что нужно сделать:**

1. Запушьте репозиторий на GitLab, изменив origin. Это изучалось на занятии по Git.
2. Создайте .gitlab-ci.yml, описав в нём все необходимые, на ваш взгляд, этапы.

В качестве ответа в шаблон с решением добавьте:
файл gitlab-ci.yml для своего проекта или вставьте код в соответствующее поле в шаблоне;
скриншоты с успешно собранными сборками.

---
1. Клонирую репозиторий в папку проекта:

```
cd ~/netology/dz
git clone https://github.com/dz/sdvps-materials.git
cp -r sdvps-materials/gitlab/* ./
rm -rf sdvps-materials
```

Инициализирую Git и подключаю локальный GitLab:

```
git init
git remote add origin http://192.168.1.88/root/test-project-1.git
git add .
git commit -m "Task 2. Initial commit"
git branch -M main
git push -u origin main
```

2. В корне проекта создаю .gitlab-ci.yml

```
stages:
  - build
  - test
  - deploy

variables:
  PROJECT_NAME: "netology-gitlab"

build-job:
  stage: build
  tags:
    - docker
  script:
    - echo "Building $PROJECT_NAME..."
    - mkdir -p build
    - echo "Build complete at $(date)" > build/result.txt
  artifacts:
    paths:
      - build/

test-job:
  stage: test
  tags:
    - docker
  script:
    - echo "Running tests for $PROJECT_NAME..."
    - cat build/result.txt
    - echo "Tests passed successfully!"

deploy-job:
  stage: deploy
  tags:
    - docker
  script:
    - echo "Simulating deploy of $PROJECT_NAME..."
    - echo "Deployment finished at $(date)"
```

Делаю коммит и запушиваю на GitLab

```
git add .
git commit -m "Initial commit with CI pipeline"
git push

```

В веб-интерфейсе вижу пайплайн.
> Не все было сразу зелено, проблема была в том, что я забыл, что local для контейнера есть сам контейнер и раннер не находил GitLab. Но, в итоге, все позеленело.

Логи джобов:

build-job

```
Running with gitlab-runner 18.4.0 (139a0ac0)
  on docker-runner aQXkPm3na, system ID: r_XLxsrsZKarbT
Preparing the "docker" executor
00:05
Using default image
Using Docker executor with image alpine:latest ...
Using default image
Using effective pull policy of [always] for container alpine:latest
Pulling docker image alpine:latest ...
Using docker image sha256:706db57fb2063f39f69632c5b5c9c439633fda35110e65587c5d85553fd1cc38 for alpine:latest with digest alpine@sha256:4b7ce07002c69e8f3d704a9c5d6fd3053be500b7f1c69fc0d80990c2ad8dd412 ...
Preparing environment
00:01
Using effective pull policy of [always] for container sha256:36d8f9ccf058dee712a3dafabeb1615104528d1188eb45d662b73b02a32a424b
Running on runner-aqxkpm3na-project-1-concurrent-0 via ubuntest...
Getting source from Git repository
00:02
Gitaly correlation ID: 01K74KKQEP8Z19B4SWS7FVZ82W
Fetching changes with git depth set to 20...
Reinitialized existing Git repository in /builds/root/test-project-1/.git/
Created fresh repository.
Checking out 88f8073d as detached HEAD (ref is main)...
Skipping Git submodules setup
Executing "step_script" stage of the job script
00:02
Using default image
Using effective pull policy of [always] for container alpine:latest
Using docker image sha256:706db57fb2063f39f69632c5b5c9c439633fda35110e65587c5d85553fd1cc38 for alpine:latest with digest alpine@sha256:4b7ce07002c69e8f3d704a9c5d6fd3053be500b7f1c69fc0d80990c2ad8dd412 ...
$ echo "Building $PROJECT_NAME..."
Building netology-gitlab...
$ mkdir -p build
$ echo "Build complete at $(date)" > build/result.txt
Uploading artifacts for successful job
00:03
Uploading artifacts...
build/: found 2 matching artifact files and directories 
Uploading artifacts as "archive" to coordinator... 201 Created  correlation_id=01K74KM29GEDNZXPA15R0W0GB6 id=6 responseStatus=201 Created token=64_Xokc_g
Cleaning up project directory and file based variables
00:01
Job succeeded
```
test-job

```
Running with gitlab-runner 18.4.0 (139a0ac0)
  on docker-runner aQXkPm3na, system ID: r_XLxsrsZKarbT
Preparing the "docker" executor
00:04
Using default image
Using Docker executor with image alpine:latest ...
Using default image
Using effective pull policy of [always] for container alpine:latest
Pulling docker image alpine:latest ...
Using docker image sha256:706db57fb2063f39f69632c5b5c9c439633fda35110e65587c5d85553fd1cc38 for alpine:latest with digest alpine@sha256:4b7ce07002c69e8f3d704a9c5d6fd3053be500b7f1c69fc0d80990c2ad8dd412 ...
Preparing environment
00:01
Using effective pull policy of [always] for container sha256:36d8f9ccf058dee712a3dafabeb1615104528d1188eb45d662b73b02a32a424b
Running on runner-aqxkpm3na-project-1-concurrent-0 via ubuntest...
Getting source from Git repository
00:01
Gitaly correlation ID: 01K74KM8J84BS6AFPBEBAZ9MG1
Fetching changes with git depth set to 20...
Reinitialized existing Git repository in /builds/root/test-project-1/.git/
Created fresh repository.
Checking out 88f8073d as detached HEAD (ref is main)...
Removing build/
Skipping Git submodules setup
Downloading artifacts
00:01
Downloading artifacts for build-job (6)...
Downloading artifacts from coordinator... ok        correlation_id=01K74KMF5E3MQ2N6006AMKDHA2 host=192.168.1.88 id=6 responseStatus=200 OK token=64_Y4sJcJ
Executing "step_script" stage of the job script
00:01
Using default image
Using effective pull policy of [always] for container alpine:latest
Using docker image sha256:706db57fb2063f39f69632c5b5c9c439633fda35110e65587c5d85553fd1cc38 for alpine:latest with digest alpine@sha256:4b7ce07002c69e8f3d704a9c5d6fd3053be500b7f1c69fc0d80990c2ad8dd412 ...
$ echo "Running tests for $PROJECT_NAME..."
Running tests for netology-gitlab...
$ cat build/result.txt
Build complete at Thu Mar 5 13:47:26 UTC 2026
$ echo "Tests passed successfully!"
Tests passed successfully!
Cleaning up project directory and file based variables
00:01
Job succeeded
```

deploy-job

```
Running with gitlab-runner 18.4.0 (139a0ac0)
  on docker-runner aQXkPm3na, system ID: r_XLxsrsZKarbT
Preparing the "docker" executor
00:03
Using default image
Using Docker executor with image alpine:latest ...
Using default image
Using effective pull policy of [always] for container alpine:latest
Pulling docker image alpine:latest ...
Using docker image sha256:706db57fb2063f39f69632c5b5c9c439633fda35110e65587c5d85553fd1cc38 for alpine:latest with digest alpine@sha256:4b7ce07002c69e8f3d704a9c5d6fd3053be500b7f1c69fc0d80990c2ad8dd412 ...
Preparing environment
00:01
Using effective pull policy of [always] for container sha256:36d8f9ccf058dee712a3dafabeb1615104528d1188eb45d662b73b02a32a424b
Running on runner-aqxkpm3na-project-1-concurrent-0 via ubuntest...
Getting source from Git repository
00:02
Gitaly correlation ID: 01K74KMN2BDFV963CB2KPWWAX9
Fetching changes with git depth set to 20...
Reinitialized existing Git repository in /builds/root/test-project-1/.git/
Created fresh repository.
Checking out 88f8073d as detached HEAD (ref is main)...
Removing build/
Skipping Git submodules setup
Downloading artifacts
00:01
Downloading artifacts for build-job (6)...
Downloading artifacts from coordinator... ok        correlation_id=01K74KMVCQG8YNMPKX6VJBB7PT host=192.168.1.88 id=6 responseStatus=200 OK token=64_aGzazH
Executing "step_script" stage of the job script
00:01
Using default image
Using effective pull policy of [always] for container alpine:latest
Using docker image sha256:706db57fb2063f39f69632c5b5c9c439633fda35110e65587c5d85553fd1cc38 for alpine:latest with digest alpine@sha256:4b7ce07002c69e8f3d704a9c5d6fd3053be500b7f1c69fc0d80990c2ad8dd412 ...
$ echo "Simulating deploy of $PROJECT_NAME..."
Simulating deploy of netology-gitlab...
$ echo "Deployment finished at $(date)"
Deployment finished at Thu Mar  5 13:47:54 UTC 2026
Cleaning up project directory and file based variables
00:00
Job succeeded
```

<img width="928" height="568" alt="Capture_pipeline" src="https://github.com/user-attachments/assets/c1e2fdc7-66cf-467a-ae18-8ab008c1068c" />


