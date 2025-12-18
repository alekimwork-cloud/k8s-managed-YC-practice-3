# Практическая работа по Managed Kubernetes в Yandex Cloud
Этот репозиторий содержит материалы для третьей практической работы по развертыванию и управлению Kubernetes-кластером в Yandex Managed Service for Kubernetes (MSK). Проект демонстрирует создание production-ready кластера с использованием IaC подхода.[web:1][web:3]


## Требования
- Yandex Cloud аккаунт с доступом к Managed Kubernetes
- yc CLI версии 0.100+ [web:3]
- kubectl версии 1.29+
- Terraform 1.5+ (для IaC манифестов)
- Git и доступ к репозиторию


## Структура проекта
k8s-managed-YC-practice-3/
├── manifests/ # Kubernetes манифесты
│ ├── namespaces.yaml
│ ├── network-policies/
│ └── deployments/
├── terraform/ # Terraform конфигурация кластера
│ ├── main.tf
│ ├── variables.tf
│ └── outputs.tf
├── scripts/ # Автоматизация развертывания
│ ├── create-cluster.sh
│ └── connect-kubeconfig.sh
├── docs/ # Документация и инструкции
└── README.md # Этот файл


## Быстрый старт
1. Клонируйте репозиторий:
git clone https://github.com/alekimwork-cloud/k8s-managed-YC-practice-3.git
cd k8s-managed-YC-practice-3

2. Настройте Yandex Cloud CLI:
yc config set cloud <your-cloud-id>
yc config set folder-id <your-folder-id>
yc auth init

3. Создайте кластер:
./scripts/connect-kubeconfig.sh
kubectl get nodes



## Развертывание манифестов
После создания кластера примените манифесты:
kubectl apply -f manifests/namespaces.yaml
kubectl apply -f manifests/


## Очистка ресурсов
Удаление манифестов
kubectl delete -f manifests/

Уничтожение Terraform ресурсов
cd terraform && terraform destroy -auto-approve

Удаление кластера через CLI
yc managed-kubernetes cluster delete <cluster-name>


## Дополнительные ресурсы
- [Официальная документация MSK](https://yandex.cloud/ru/docs/managed-kubernetes/) [web:1]
- [Yandex Cloud CLI Reference](https://yandex.cloud/ru/docs/cli/cli-ref/managed-kubernetes/) [web:3]
- Курс по Kubernetes в Yandex Cloud [web:10]

