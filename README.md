# Smart Home

## Docs

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
