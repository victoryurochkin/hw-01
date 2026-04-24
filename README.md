# Домашнее задание к занятию «Введение в Terraform»

**Выполнил:** Юрочкин В.А.  
**Задание:** `hw-01`  
**Окружение:** Ubuntu Server 22.04.5 LTS  
**Terraform:** 1.14.9  
**Docker:** 29.4.1  

---

## Задание 1

Необходимо выполнить следующие действия:

1. Перейти в каталог `src`.
2. Скачать все необходимые зависимости, использованные в проекте.
3. Изучить файл `.gitignore` и определить, в каком Terraform-файле допустимо хранить личную/секретную информацию.
4. Выполнить код проекта.
5. Найти в `terraform.tfstate` секретное содержимое ресурса `random_password`.
6. Раскомментировать блок кода в `main.tf`, выполнить `terraform validate`, объяснить и исправить ошибки.
7. Выполнить исправленный код и приложить вывод `docker ps`.
8. Заменить имя Docker-контейнера на `hello_world`, выполнить `terraform apply -auto-approve`, объяснить назначение и опасность ключа `-auto-approve`.
9. Уничтожить созданные ресурсы с помощью Terraform.
10. Убедиться, что ресурсы удалены.
11. Приложить содержимое `terraform.tfstate`.
12. Объяснить, почему Docker-образ `nginx:latest` не был удалён.

---

# Подготовка окружения

Для выполнения задания была поднята отдельная виртуальная машина на собственном сервере.

<img width="3071" height="1427" alt="image" src="https://github.com/user-attachments/assets/90049ec4-9bfc-4bf4-8970-5ed70159a3d6" />

<img width="1535" height="1009" alt="image" src="https://github.com/user-attachments/assets/631d29cd-f5ca-4cd5-a191-1d7d8f109621" />

---

## Обновление системы и установка базовых утилит

После установки Ubuntu Server 22.04 была выполнена подготовка системы:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget git nano jq unzip ca-certificates gnupg lsb-release
```

Были установлены необходимые утилиты:

- `git` — для клонирования репозитория;
- `jq` — для работы с JSON-файлом `terraform.tfstate`;
- `curl`, `wget`, `unzip` — для загрузки и распаковки файлов;
- `ca-certificates`, `gnupg`, `lsb-release` — для работы с внешними репозиториями.

---

## Установка и проверка Docker

Docker был установлен на сервер Ubuntu Server 22.04.

Команды установки Docker:

```bash
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo systemctl enable --now docker
```

Проверка версии Docker:

```bash
docker --version
sudo docker ps
```

Результат:

```text
Docker version 29.4.1, build 055a478

CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

Для работы с Docker без `sudo` пользователь был добавлен в группу `docker`:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

Проверка:

```bash
docker ps
```

Результат:

```text
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

---

## Установка и проверка Terraform

Terraform был установлен на Ubuntu Server 22.04 через `snap`.

```bash
snap install terraform --classic
terraform --version
```

Результат:

```text
terraform 1.14.9 from Snapcrafters installed
Terraform v1.14.9
on linux_amd64
```

<img width="1066" height="217" alt="image" src="https://github.com/user-attachments/assets/47fed556-5ff4-43a8-a6cf-0a7ec4805eb3" />

---

# Клонирование репозитория и переход в каталог задания

Был склонирован репозиторий с домашним заданием:

```bash
cd /home/it

git clone https://github.com/netology-code/ter-homeworks.git

cd /home/it/ter-homeworks/01/src

ls -la
```

В каталоге `01/src` находятся файлы задания:

```text
.gitignore
main.tf
.terraformrc
```

---

## Настройка версии Terraform в `main.tf`

В исходном файле `main.tf` было ограничение версии Terraform:

```hcl
required_version = "~>1.12.0"
```

Так как установленная версия Terraform — `1.14.9`, ограничение было изменено на совместимое:

```bash
sed -i 's/required_version = "~>1.12.0"/required_version = ">=1.12.0"/' main.tf

grep required_version main.tf
```

Результат:

```hcl
required_version = ">=1.12.0"
```

---

## Настройка зеркала провайдеров Terraform

В каталоге задания присутствует файл `.terraformrc`, который указывает Terraform использовать зеркало провайдеров:

```bash
cat .terraformrc
```

Содержимое файла:

```hcl
provider_installation {
  network_mirror {
    url = "https://terraform-mirror.yandexcloud.net/"
    include = ["registry.terraform.io/*/*"]
  }
  direct {
    exclude = ["registry.terraform.io/*/*"]
  }
}
```

Для использования данного файла была задана переменная окружения:

```bash
export TF_CLI_CONFIG_FILE=/home/it/ter-homeworks/01/src/.terraformrc
```

Проверка:

```bash
echo $TF_CLI_CONFIG_FILE
```

Результат:

```text
/home/it/ter-homeworks/01/src/.terraformrc
```

После этого была очищена предыдущая инициализация:

```bash
rm -rf .terraform .terraform.lock.hcl
```

---

# Инициализация Terraform

В каталоге `src` была выполнена инициализация Terraform:

```bash
terraform init
```

Результат:

```text
Initializing the backend...
Initializing provider plugins...
- Finding latest version of kreuzwerker/docker...
- Finding latest version of hashicorp/random...
- Installing kreuzwerker/docker v4.2.0...
- Installed kreuzwerker/docker v4.2.0 (unauthenticated)
- Installing hashicorp/random v3.8.1...
- Installed hashicorp/random v3.8.1 (unauthenticated)

Terraform has been successfully initialized!
```

<img width="2430" height="1118" alt="image" src="https://github.com/user-attachments/assets/b1afc36a-8dbc-499e-9aba-917d652316e5" />

Таким образом, необходимые зависимости проекта были скачаны.

---

# Анализ файла `.gitignore`

Для просмотра файла `.gitignore` была выполнена команда:

```bash
cat .gitignore
```

В файле `.gitignore` указан файл:

```text
personal.auto.tfvars
```

Ответ:

> Секретную информацию, такую как логины, пароли, ключи и токены, допустимо хранить в файле `personal.auto.tfvars`, так как он указан в `.gitignore` и не должен попадать в Git-репозиторий.

---

# Первый запуск Terraform-кода

Первоначально в проекте был активен только ресурс `random_password`.

Был выполнен запуск:

```bash
terraform apply
```

Terraform показал план создания одного ресурса:

```text
Plan: 1 to add, 0 to change, 0 to destroy.
```

После подтверждения:

```text
yes
```

Ресурс был создан:

```text
random_password.random_string: Creation complete after 0s [id=none]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

<img width="2516" height="1364" alt="image" src="https://github.com/user-attachments/assets/c48af71c-485b-4319-8d7d-47276748a6f8" />

---

# Получение секрета из `terraform.tfstate`

Секретное значение ресурса `random_password` было найдено в state-файле `terraform.tfstate`.

Команда:

```bash
jq -r '.resources[] | select(.type=="random_password") | .instances[0].attributes.result' terraform.tfstate
```

Результат:

```text
kYe29Bk9sSRVWvCm
```

<img width="2523" height="99" alt="image" src="https://github.com/user-attachments/assets/bd19e398-fc29-49df-8107-9c2446a71e3f" />

Ответ:

```text
Ключ: result
Значение: kYe29Bk9sSRVWvCm
```

Также данный ключ находится в структуре state-файла по пути:

```text
.resources[].instances[0].attributes.result
```

---

# Раскомментирование и исправление блока Docker-ресурсов

В исходном файле `main.tf` был закомментирован блок Docker-ресурсов.

Исходный проблемный блок выглядел следующим образом:

```hcl
/*
resource "docker_image" {
  name         = "nginx:latest"
  keep_locally = true
}

resource "docker_container" "1nginx" {
  image = docker_image.nginx.image_id
  name  = "example_${random_password.random_string_FAKE.resulT}"

  ports {
    internal = 80
    external = 9090
  }
}
*/
```

После раскомментирования и выполнения проверки:

```bash
terraform validate
```

были выявлены намеренно допущенные ошибки.

---

## Ошибка 1. У ресурса `docker_image` отсутствовало локальное имя

Было:

```hcl
resource "docker_image" {
```

В Terraform ресурс должен иметь два имени:

```hcl
resource "<TYPE>" "<LOCAL_NAME>" {
```

Исправлено:

```hcl
resource "docker_image" "nginx" {
```

---

## Ошибка 2. Имя ресурса начиналось с цифры

Было:

```hcl
resource "docker_container" "1nginx" {
```

Имя ресурса не должно начинаться с цифры.

Исправлено:

```hcl
resource "docker_container" "nginx" {
```

---

## Ошибка 3. Неверная ссылка на ресурс `random_password`

Было:

```hcl
random_password.random_string_FAKE.resulT
```

Проблемы:

- ресурса `random_string_FAKE` не существует;
- атрибут `resulT` указан с неправильным регистром;
- корректный ресурс называется `random_password.random_string`;
- корректный атрибут называется `result`.

Исправлено:

```hcl
random_password.random_string.result
```

---

# Исправленный файл `main.tf`

В результате файл `main.tf` был приведён к следующему виду:

```hcl
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
    }
  }

  required_version = ">=1.12.0"
}

provider "docker" {}

resource "random_password" "random_string" {
  length      = 16
  special     = false
  min_upper   = 1
  min_lower   = 1
  min_numeric = 1
}

resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = true
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "example_${random_password.random_string.result}"

  ports {
    internal = 80
    external = 9090
  }
}
```

---

# Проверка исправленного кода

После исправления была выполнена команда форматирования:

```bash
terraform fmt
```

Затем была выполнена проверка конфигурации:

```bash
terraform validate
```

Результат:

```text
Success! The configuration is valid.
```

---

# Выполнение исправленного Terraform-кода

Был выполнен запуск:

```bash
terraform apply
```

Terraform показал план создания двух ресурсов:

```text
Plan: 2 to add, 0 to change, 0 to destroy.
```

Создаваемые ресурсы:

- `docker_image.nginx`;
- `docker_container.nginx`.

После подтверждения:

```text
yes
```

Terraform создал Docker-образ и Docker-контейнер:

```text
docker_image.nginx: Creation complete after 10s
docker_container.nginx: Creation complete after 3s

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

<img width="2516" height="1536" alt="image" src="https://github.com/user-attachments/assets/138c0dcf-8469-4ca6-8969-75599695ac43" />

---

## Проверка запущенного контейнера

Для проверки был выполнен:

```bash
docker ps
```

Результат:

```text
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                  NAMES
0b9a4d8a35b2   6e23479198b9   "/docker-entrypoint.…"   5 seconds ago   Up 2 seconds   0.0.0.0:9090->80/tcp   example_kYe29Bk9sSRVWvCm
```

<img width="2521" height="1246" alt="image" src="https://github.com/user-attachments/assets/a87919e7-bda8-4b7f-98b0-9534e6f6c64e" />

Контейнер был успешно создан и запущен на порту `9090`.

---

# Изменение имени Docker-контейнера на `hello_world`

По заданию нужно было заменить имя Docker-контейнера на:

```text
hello_world
```

Важно: имя Docker-образа должно остаться прежним:

```hcl
name = "nginx:latest"
```

Для изменения имени контейнера была выполнена команда:

```bash
sed -i 's/name  = "example_${random_password.random_string.result}"/name  = "hello_world"/' main.tf
```

Проверка изменённого блока:

```bash
grep -A10 'resource "docker_container"' main.tf
```

Результат:

```hcl
resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "hello_world"

  ports {
    internal = 80
    external = 9090
  }
}
```

---

# Применение изменений с ключом `-auto-approve`

Изменения были применены командой:

```bash
terraform apply -auto-approve
```

Terraform определил, что контейнер нужно пересоздать, так как изменилось его имя:

```text
Plan: 1 to add, 0 to change, 1 to destroy.
```

После выполнения:

```text
docker_container.nginx: Destruction complete after 1s
docker_container.nginx: Creation complete after 1s

Apply complete! Resources: 1 added, 0 changed, 1 destroyed.
```

<img width="2520" height="1536" alt="image" src="https://github.com/user-attachments/assets/6cb6ec19-0dfd-4012-be4e-d369298f4917" />

---

## Проверка контейнера после переименования

Команда:

```bash
docker ps
```

Результат:

```text
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                  NAMES
23dea0c0c984   6e23479198b9   "/docker-entrypoint.…"   8 seconds ago   Up 7 seconds   0.0.0.0:9090->80/tcp   hello_world
```

<img width="2520" height="1546" alt="image" src="https://github.com/user-attachments/assets/05deb016-2648-4cbb-a914-1deb00e95de7" />

---

## Объяснение ключа `-auto-approve`

Ключ:

```bash
-auto-approve
```

используется для автоматического подтверждения выполнения `terraform apply` без интерактивного ввода `yes`.

Обычно при выполнении команды:

```bash
terraform apply
```

Terraform показывает план изменений и просит подтверждение:

```text
Only 'yes' will be accepted to approve.
```

При использовании:

```bash
terraform apply -auto-approve
```

подтверждение не запрашивается, и изменения применяются сразу.

### В чём опасность `-auto-approve`

Опасность заключается в том, что изменения применяются без ручной проверки плана.

Это может привести к следующим последствиям:

- случайное удаление ресурсов;
- случайное пересоздание ресурсов;
- простой сервиса;
- потеря данных;
- применение ошибочной конфигурации;
- изменение инфраструктуры без дополнительного контроля.

### Где может пригодиться `-auto-approve`

Ключ может быть полезен:

- в CI/CD-пайплайнах;
- в автоматизированных тестовых окружениях;
- в учебных лабораторных стендах;
- в скриптах автоматического развёртывания;
- при автоматическом создании временной инфраструктуры.

---

# Уничтожение созданных ресурсов

Для удаления созданных ресурсов была выполнена команда:

```bash
terraform destroy
```

Terraform показал план удаления трёх ресурсов:

```text
Plan: 0 to add, 0 to change, 3 to destroy.
```

Удаляемые ресурсы:

- `docker_container.nginx`;
- `docker_image.nginx`;
- `random_password.random_string`.

После подтверждения:

```text
yes
```

Terraform удалил ресурсы:

```text
Destroy complete! Resources: 3 destroyed.
```

<img width="2519" height="1535" alt="image" src="https://github.com/user-attachments/assets/f1fc6b4d-5568-4150-bd82-47c907d0661a" />

---

# Проверка удаления контейнеров

После выполнения `terraform destroy` была выполнена проверка:

```bash
docker ps -a
```

Результат:

```text
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

Контейнеры отсутствуют, значит Docker-контейнер был успешно удалён.

---

# Содержимое файла `terraform.tfstate` после удаления ресурсов

После удаления ресурсов файл `terraform.tfstate` содержит пустой список ресурсов:

```json
{
  "version": 4,
  "terraform_version": "1.14.9",
  "serial": 11,
  "lineage": "bc80438b-18c0-72a5-e8da-98d1a911ed25",
  "outputs": {},
  "resources": [],
  "check_results": null
}
```

Ключевой момент:

```json
"resources": []
```

Это означает, что Terraform больше не управляет созданными ранее ресурсами.

---

# Проверка Docker-образа `nginx:latest`

После удаления ресурсов была выполнена проверка локальных Docker-образов:

```bash
docker images | grep nginx
```

Результат:

```text
nginx:latest   6e23479198b9        240MB         65.8MB
```

<img width="2514" height="1485" alt="image" src="https://github.com/user-attachments/assets/04dda206-746a-4b02-b0fe-815653d5deea" />

---

# Почему Docker-образ `nginx:latest` не был удалён

Docker-образ `nginx:latest` не был удалён после выполнения `terraform destroy`, потому что в коде ресурса `docker_image` был указан параметр:

```hcl
keep_locally = true
```

Фрагмент кода:

```hcl
resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = true
}
```

Именно этот параметр отвечает за сохранение локального Docker-образа после удаления ресурса Terraform.

В плане удаления также видно, что параметр был установлен:

```text
# docker_image.nginx will be destroyed
- resource "docker_image" "nginx" {
    - keep_locally = true -> null
    - name         = "nginx:latest" -> null
  }
```

Согласно документации Terraform Docker Provider для ресурса `docker_image`, параметр `keep_locally` описан следующим образом:

```text
If true, then the Docker image won't be deleted on destroy operation.
```

То есть если `keep_locally = true`, то при выполнении `terraform destroy` Terraform удаляет ресурс из своего state-файла, но не удаляет сам Docker-образ из локального Docker-хранилища.

---

# Итог

В ходе выполнения задания было сделано:

- подготовлена виртуальная машина с Ubuntu Server 22.04;
- установлен Docker;
- установлен Terraform;
- склонирован репозиторий с домашним заданием;
- выполнена инициализация Terraform-проекта;
- изучен `.gitignore`;
- определён файл для хранения секретов: `personal.auto.tfvars`;
- выполнен исходный Terraform-код;
- найден секрет ресурса `random_password` в `terraform.tfstate`;
- исправлены ошибки в Terraform-коде;
- создан Docker-образ `nginx:latest`;
- создан Docker-контейнер;
- имя контейнера изменено на `hello_world`;
- применён ключ `-auto-approve`;
- ресурсы удалены через `terraform destroy`;
- подтверждено, что контейнеры удалены;
- подтверждено, что Docker-образ `nginx:latest` остался локально из-за параметра `keep_locally = true`.
