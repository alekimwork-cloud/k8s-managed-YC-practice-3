# Практическая работа по Managed Kubernetes в Yandex Cloud
Этот репозиторий содержит материалы для третьей практической работы по развертыванию и управлению Kubernetes-кластером в Yandex Managed Service for Kubernetes (MSK). Проект демонстрирует создание production-ready кластера с использованием IaC подхода.


## Требования
- Yandex Cloud аккаунт с доступом к Managed Kubernetes
- yc CLI версии 0.100+
- kubectl версии 1.29+
- Terraform 1.5+ (для IaC манифестов)
- Git и доступ к репозиторию


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
- [Официальная документация MSK](https://yandex.cloud/ru/docs/managed-kubernetes/)
- [Yandex Cloud CLI Reference](https://yandex.cloud/ru/docs/cli/cli-ref/managed-kubernetes/)
- Курс по Kubernetes в Yandex Cloud

