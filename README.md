# Домашнее задание к занятию «Helm»

### Цель задания

В тестовой среде Kubernetes необходимо установить и обновить приложения с помощью Helm.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение, например, MicroK8S.
2. Установленный локальный kubectl.
3. Установленный локальный Helm.
4. Редактор YAML-файлов с подключенным репозиторием GitHub.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Инструкция](https://helm.sh/docs/intro/install/) по установке Helm. [Helm completion](https://helm.sh/docs/helm/helm_completion/).

------

### Задание 1. Подготовить Helm-чарт для приложения

1. Необходимо упаковать приложение в чарт для деплоя в разные окружения. 
2. Каждый компонент приложения деплоится отдельным deployment’ом или statefulset’ом.
3. В переменных чарта измените образ приложения для изменения версии.

## Решение
first-app
    ├── charts
    ├── Chart.yaml
    ├── .helmignore
    ├── templates
    │   ├── backend-deployment.yaml
    │   ├── backend-svc.yaml
    │   ├── frontend-deployment.yaml
    │   ├── frontend-svc.yaml
    └── values.yaml

backend-deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: {{ .Values.backend.replicaCount }}
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: multitool
        image: "{{ .Values.backend.image.repository }}:{{ .Values.backend.image.tag }}"
        env:
        - name: HTTP_PORT
          value: "80"
        ports:
        - containerPort: 80
```

backend-svc.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

frontend-deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: {{ .Values.frontend.replicaCount }}
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: nginx
        image: "{{ .Values.frontend.image.repository }}:{{ .Values.frontend.image.tag }}"
        ports:
        - containerPort: 80
```

frontend-svc.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

values.yaml
```yaml
frontend:
  replicaCount: 3
  image:
    repository: nginx
    tag: alpine

backend:
  replicaCount: 1
  image:
    repository: wbitt/network-multitool
    tag: latest
```
без файла values.yaml
![alt text](https://github.com/VN351/kube-10/raw/main/images/1-1.jpg)
с файлом values.yaml
![alt text](https://github.com/VN351/kube-10/raw/main/images/1-2.jpg)

------
### Задание 2. Запустить две версии в разных неймспейсах

1. Подготовив чарт, необходимо его проверить. Запуститe несколько копий приложения.
2. Одну версию в namespace=app1, вторую версию в том же неймспейсе, третью версию в namespace=app2.
3. Продемонстрируйте результат.

## Решение
first-app
    ├── charts
    ├── Chart.yaml
    ├── .helmignore
    ├── templates
    │   ├── backend-deployment.yaml
    │   ├── backend-svc.yaml
    │   ├── frontend-deployment.yaml
    │   ├── frontend-svc.yaml
    │   └── _helpers.tpl
    └── values.yaml

backend-deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "first-app.fullname" . }}-backend
spec:
  replicas: {{ .Values.backend.replicaCount }}
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: multitool
        image: "{{ .Values.backend.image.repository }}:{{ .Values.backend.image.tag }}"
        env:
        - name: HTTP_PORT
          value: "80"
        ports:
        - containerPort: 80
```

backend-svc.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "first-app.fullname" . }}-backend-svc
spec:
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

frontend-deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "first-app.fullname" . }}-frontend
spec:
  replicas: {{ .Values.frontend.replicaCount }}
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: nginx
        image: "{{ .Values.frontend.image.repository }}:{{ .Values.frontend.image.tag }}"
        ports:
        - containerPort: 80
```

frontend-svc.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "first-app.fullname" . }}-frontend-svc
spec:
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

_helpers.tpl
```tpl
{{- define "first-app.fullname" -}}
{{- printf "%s-%s" .Release.Name .Chart.Name | trunc 63 | trimSuffix "-" -}}
{{- end -}}
```

values.yaml
```yaml
frontend:
  replicaCount: 3
  image:
    repository: nginx
    tag: alpine

backend:
  replicaCount: 1
  image:
    repository: wbitt/network-multitool
    tag: latest
```

![alt text](https://github.com/VN351/kube-10/raw/main/images/2-1.jpg)

### Правила приёма работы

1. Домашняя работа оформляется в своём Git репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, `helm`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

