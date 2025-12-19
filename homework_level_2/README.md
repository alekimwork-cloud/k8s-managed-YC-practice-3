# Домашнее задание: Развертывание приложения в Kubernetes с HPA и Canary Deployment

## Цель задания
Освоить базовые операции с Kubernetes в облачной среде Yandex Cloud, включая развертывание приложений, настройку Horizontal Pod Autoscaler (HPA) и выполнение canary deployment.

## Предварительные требования
- Аккаунт в Yandex Cloud с достаточными квотами для создания кластера Kubernetes.
- Установленный kubectl и настроенный доступ к кластеру.
- Знание основных команд Kubernetes.

## Шаги выполнения задания

### 1. Создание кластера Kubernetes в Yandex Cloud
- Создайте новый кластер Kubernetes в Yandex Cloud через веб-консоль или CLI.
- Убедитесь, что кластер имеет хотя бы одну рабочую ноду.
- Настройте kubectl для доступа к кластеру (скачайте kubeconfig).

### 2. Подготовка манифестов
- Скачайте предоставленные манифесты из директории `homework_level_2`.
- В каждом манифесте замените `STUDENT_NAME_GROUP` на ваше ФИО и группу в формате "ФамилияИО_Группа" (например, "ИвановИИ_БВТ2201").
- Файлы для редактирования:
  - `01-namespace.yaml`
  - `02-deployment.yaml`
  - `03-service.yaml`
  - `04-hpa.yaml`
  - `05-canary-deployment.yaml`

### 3. Развертывание приложения
- Примените манифесты в следующем порядке:
  1. `kubectl apply -f 01-namespace.yaml`
  2. `kubectl apply -f 02-deployment.yaml`
  3. `kubectl apply -f 03-service.yaml`
  4. `kubectl apply -f 04-hpa.yaml`
- Проверьте статус развертывания: `kubectl get pods -n student-workspace`
- Получите внешний IP сервиса: `kubectl get svc -n student-workspace`

### 4. Тестирование Horizontal Pod Autoscaler (HPA)
- Убедитесь, что HPA активен: `kubectl get hpa -n student-workspace`
- Создайте нагрузку на приложение для проверки масштабирования:
  - Используйте инструмент для генерации HTTP-запросов (например, curl в цикле или специализированный инструмент).
  - Мониторьте количество подов: `kubectl get pods -n student-workspace`
  - HPA должен увеличить количество реплик при CPU utilization > 50%.
- Зафиксируйте начальное и конечное количество подов.

### 5. Выполнение Canary Deployment
- Разверните canary версию приложения: `kubectl apply -f 05-canary-deployment.yaml`
- Это создаст deployment с новой версией (nginx:alpine) и 1 репликой.
- Service автоматически начнет балансировать трафик между stable (v1) и canary (v2) версиями.
- Протестируйте работу приложения, убедитесь, что обе версии доступны.
- Для полноценного canary:
  - Постепенно увеличивайте реплики canary deployment: `kubectl scale deployment student-app-canary --replicas=2 -n student-workspace`
  - Уменьшайте реплики stable deployment: `kubectl scale deployment student-app --replicas=1 -n student-workspace`
  - Мониторьте распределение трафика и работоспособность.
- После успешного тестирования переключите весь трафик на canary: удалите stable deployment и переименуйте canary в основной.

### 6. Очистка ресурсов
- После завершения тестирования удалите все ресурсы: `kubectl delete namespace student-workspace`
- Удалите кластер в Yandex Cloud, если он больше не нужен.

## Важная информация
- Все манифесты должны быть применены в указанном порядке.
- Не изменяйте другие параметры в манифестах без необходимости.
- В случае ошибок проверяйте логи подов: `kubectl logs <pod-name> -n student-workspace`
- HPA масштабирует от 1 до 10 реплик на основе CPU utilization 50%.
- Canary deployment позволяет постепенно вводить новую версию, минимизируя риски.

## Критерии оценки
- Корректное развертывание всех компонентов.
- Успешное тестирование HPA (масштабирование под нагрузкой).
- Успешное выполнение canary deployment (постепенное переключение трафика).
- Чистый код манифестов с правильными метками.