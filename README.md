# Домашнее задание к занятию `«GitLab»` - `Тимохин Максим`

---

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


