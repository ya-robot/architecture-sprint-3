# Smart Home

## Docs

### Анализ существующего монолитного решения

1. Проект отлично организован в рамках Clean Architecture by Uncle Bob. Функцианальность: существует CRUD для `Систем обогрева`, ендпоинты удаленного управления (вкл.,выкл.,установка температуры и получение текущих показателей). Есть отличная система развертки в виде Helm чарта под K8s с автоскейлером, узким горлышком будет выступать только БД и пропускная способность канала GSM сети опрашиваемых устройств.

2. Можно явно выделить 3 домена из текущего решения - `Устройства умного дома`, `Удаленное управление`, `Удалённый мониторинг`

3. Для покрытия текущих потребностей ведения бизнеса MVP (100 домов) данное решение актуально и не требует сильной доработки. Однако, что я считаю критически необходимым - брокер сообщений для телеметрии, например MQTT, отдельную систему сбора данных, устанавливаемую клиентом (роутер с ПО), которая опрашивает и управляет сенсорами. Надо вводить отдельную модель для Роутера с регистрацией по IMEI, универсальную модель\модели поддерживаемых устройств, типизацию сенсоров. Внедрять TimescaleDB для хранения и агрегации телеметрии. Квоты телеметрии на клиентов.

### Plant UML & ERD

You can find related diagrams at this path:

- C4
  - `docs/diagrams/c4-context.puml`
  - `docs/diagrams/c4-containers.puml`
  - `docs/diagrams/c4-components-homes.puml`
  - `docs/diagrams/c4-components-routers.puml`
  - `docs/diagrams/c4-components-devices.puml`
  - `docs/diagrams/c4-components-telemetry-rules.puml`
  - `docs/diagrams/c4-components-telemetry-reader.puml`
  - `docs/diagrams/c4-components-telemetry-writer.puml`
  - `docs/diagrams/c4-components-mqtt-adapter.puml`
  - `docs/diagrams/c4-components-notifications.puml`
  - `docs/diagrams/c4-code-preview.puml` (Описал очень кратко, дофига расписывать не хочу. Не фанат ООП)
- ERD
  - `docs/diagrams/erd.puml`

## Базовая настройка

### Запуск minikube

[Инструкция по установке](https://minikube.sigs.k8s.io/docs/start/)

```bash
minikube start
```

### Добавление токена авторизации GitHub

[Получение токена](https://github.com/settings/tokens/new)

```bash
kubectl create secret docker-registry ghcr --docker-server=https://ghcr.io --docker-username=<github_username> --docker-password=<github_token> -n default
```

### Установка API GW kusk

[Install Kusk CLI](https://docs.kusk.io/getting-started/install-kusk-cli)

```bash
kusk cluster install
```

### Смена адреса образа в helm chart

После того как вы сделали форк репозитория и у вас в репозитории отработал GitHub Action. Вам нужно получить адрес образа <https://github.com/><github_username>/architecture-sprint-3/pkgs/container/architecture-sprint-3

Он выглядит таким образом
```ghcr.io/<github_username>/architecture-sprint-3:latest```

Замените адрес образа в файле `helm/smart-home-monolith/values.yaml` на полученный файл:

```yaml
image:
  repository: ghcr.io/<github_username>/architecture-sprint-3
  tag: latest
```

### Настройка terraform

[Установите Terraform](https://yandex.cloud/ru/docs/tutorials/infrastructure-management/terraform-quickstart#install-terraform)

Создайте файл ~/.terraformrc

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

### Применяем terraform конфигурацию

```bash
cd terraform
terraform init
terraform apply
```

## Настройка API GW

```bash
kusk deploy -i api.yaml
```

## Проверяем работоспособность

```bash
kubectl port-forward svc/kusk-gateway-envoy-fleet -n kusk-system 8080:80
curl localhost:8080/hello
```

## Delete minikube

```bash
minikube delete
```
