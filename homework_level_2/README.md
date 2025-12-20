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
- В каждом манифесте замените `STUDENT_NAME_GROUP` на ваши инициалы и группу в формате "ФамилияИО_Группа" (например, "ivanow-av-bivt-22-9").
- Также замените `student-workspace` на `student-<STUDENT_NAME_GROUP>` в namespace и командах, чтобы на скриншотах отображалось ваше имя.
- Файлы для редактирования:
  - `01-namespace.yaml`
  - `02-deployment.yaml`
  - `03-service.yaml`
  - `04-hpa.yaml`
  - `05-canary-deployment.yaml`

### 3. Развертывание приложения
- Примените манифесты 0-4
- Проверьте статус развертывания: `kubectl get pods -n student-<STUDENT_NAME_GROUP>`
- Протестируйте трафик на stable версии (до canary): `kubectl run test-traffic --image=busybox --restart=Never -i --tty -- /bin/sh -c "for i in \$(seq 1 10); do wget -q -O- http://<EXTERNAL_IP>/ | grep -o -E 'OK|nginx' | head -1; done | sort | uniq -c"` (все запросы должны быть на v1, без "nginx")

### 4. Тестирование Horizontal Pod Autoscaler (HPA)
- Убедитесь, что HPA активен: `kubectl get hpa -n student-<STUDENT_NAME_GROUP>`
- Создайте нагрузку на приложение для проверки масштабирования:
  - Используйте инструмент для генерации HTTP-запросов (например, curl в цикле или специализированный инструмент). Можно создать нагрузку командой: `kubectl run loadgen --image=busybox --rm -i --tty -- /bin/sh -c "while true; do wget -q -O- http://<EXTERNAL_IP>/; done"`
  - Мониторьте количество подов (выполните команду в другом окне терминала): `kubectl get pods -n student-<STUDENT_NAME_GROUP>` или в realtime командой `kubectl get hpa -w -n student-<STUDENT_NAME_GROUP>`
  - HPA должен увеличить количество реплик при CPU utilization > 50%.
- !Зафиксируйте начальное и конечное количество подов скриншотами!

### 5. Выполнение Canary Deployment
Canary deployment - это стратегия постепенного развертывания новой версии приложения, при которой трафик сначала направляется преимущественно на стабильную версию, а затем постепенно переключается на новую (canary) версию. Это позволяет протестировать новую версию на реальном трафике с минимальными рисками.

- Разверните canary версию приложения: `kubectl apply -f 05-canary-deployment.yaml`
- Это создаст deployment с новой версией (nginx:alpine) и 1 репликой.
- Service автоматически начнет балансировать трафик между stable (v1) и canary (v2) версиями.
- Получите внешний IP сервиса: `kubectl get svc student-app-service -n student-<STUDENT_NAME_GROUP>`
- Протестируйте распределение трафика:
  - Выполните несколько запросов: `kubectl run test-traffic --image=busybox --restart=Never -i --tty -- /bin/sh -c "for i in \$(seq 1 10); do wget -q -O- http://<EXTERNAL_IP>/ | grep -o -E 'OK|nginx' | head -1; done | sort | uniq -c"`
  - Это покажет, сколько запросов попало на v1 (OK) и v2 (nginx).
- Для отслеживания перенаправления трафика в реальном времени:
  - Используйте скрипт для непрерывного мониторинга: `kubectl run monitor --image=busybox --rm -i --tty -- /bin/sh -c "while true; do echo 'Текущее распределение:'; results=''; for i in \$(seq 1 20); do result=\$(wget -q -O- http://<EXTERNAL_IP>/ | grep -o -E 'OK|nginx' | head -1); results=\"\$results \$result\"; done; echo \$results | tr ' ' '\\n' | sort | uniq -c | sed 's/OK/Stable (v1)/; s/nginx/Canary (v2)/'; sleep 5; done"`
  - Это будет каждые 5 секунд показывать, сколько запросов попало на каждую версию.
- Для полноценного canary:
  - Постепенно увеличивайте реплики canary deployment: `kubectl scale deployment student-app-canary --replicas=2 -n student-<STUDENT_NAME_GROUP>`
  - Уменьшайте реплики stable deployment: `kubectl scale deployment student-app --replicas=1 -n student-<STUDENT_NAME_GROUP>`
  - Повторно протестируйте трафик: `kubectl run test-traffic2 --image=busybox --restart=Never -i --tty -- /bin/sh -c "for i in \$(seq 1 20); do wget -q -O- http://<EXTERNAL_IP>/ | grep -o -E 'OK|nginx' | head -1; done | sort | uniq -c"`
  - Мониторьте распределение: убедитесь, что трафик распределяется пропорционально количеству реплик (например, 1:2 для 1 stable и 2 canary).
- После успешного тестирования переключите весь трафик на canary: `kubectl delete deployment student-app -n student-<STUDENT_NAME_GROUP>` и `kubectl scale deployment student-app-canary --replicas=2 -n student-<STUDENT_NAME_GROUP>`

### 6. Очистка ресурсов
- После завершения тестирования удалите все ресурсы: `kubectl delete namespace student-<STUDENT_NAME_GROUP>`
- Удалите кластер в Yandex Cloud, если он больше не нужен.

## Важная информация
- Все манифесты должны быть применены в указанном порядке.
- Не изменяйте другие параметры в манифестах без необходимости.
- В случае ошибок проверяйте логи подов: `kubectl logs <pod-name> -n student-<STUDENT_NAME_GROUP>`
- HPA масштабирует от 1 до 10 реплик на основе CPU utilization 50%.
- Canary deployment позволяет постепенно вводить новую версию, минимизируя риски.