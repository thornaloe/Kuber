Задание 1: Эксперимент с ReplicaSet

Шаг 1: Создание Docker-образа

Создаю файл app.js с содержимым:
const http = require('http');
const os = require('os');
console.log("Kubia server starting...");
var handler = function(request, response) {
  console.log("Received request from " + request.connection.remoteAddress);
  response.writeHead(200);
  response.end("You've hit " + os.hostname() + "\n");
};
var www = http.createServer(handler);
www.listen(8080);

Создаю Dockerfile:
FROM node:14
WORKDIR /app
COPY app.js .
CMD ["node", "app.js"]

Собираю и загружаю Docker-образ:
docker build -t thornaloe/kuber:latest .
docker push thornaloe/kuber:latest
Шаг 2: Развертывание ReplicaSet на кластере Kubernetes

Создаю файл replicaset.yaml:
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: kubia-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kubia
  template:
    metadata:
      labels:
        app: kubia
    spec:
      containers:
      - name: kubia
        image: thornaloe/kuber:latest

Применяю конфигурации:
kubectl apply -f replicaset.yaml

Создаю файл service.yaml:
apiVersion: v1
kind: Service
metadata:
  name: kubia-svc
spec:
  selector:
    app: kubia
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer

Применяю конфигурацию службы:
kubectl apply -f service.yaml

Дожидаюсь, пока служба получит внешний IP-адрес:
kubectl get svc

Шаг 3: Тестирование
Обращаюсь несколько раз к приложению с помощью утилиты curl:
curl http://EXTERNAL_IP

Задание 2: Запуск собственного приложения
Создаю и загружаю свой Docker-образ на Docker Hub:
docker build -t thornaloe/myapp:latest путь_к_приложению
docker push thornaloe/myapp:latest

Создаю файл replicaset_custom.yaml для своего приложения:
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: custom-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: thornaloe/myapp:latest

Применяю конфигурацию ReplicaSet для своего приложения:
kubectl apply -f replicaset_custom.yaml
Тестирую:



kubectl get pods  # Проверяю, что поды моего приложения запущены
curl http://EXTERNAL_IP
Теперь у меня развернут ReplicaSet с моим приложением и служба с типом LoadBalancer для доступа к этим приложениям.
