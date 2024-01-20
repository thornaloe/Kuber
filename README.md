# KuberNet
## Задание1
### Docker 
Для создания образа и контейнера из Dockerfile, вам потребуется выполнить следующие шаги:

1. Установите Docker на вашей операционной системе.

2. Создайте Dockerfile в вашем проекте.
```
FROM node:7
ADD app.js /app.js
ENTRYPOINT ["node", "app.js"]
```
3. Откройте терминал и перейдите в директорию, где находится ваш Dockerfile.

4. Выполните следующую команду, чтобы собрать образ из вашего Dockerfile:
```
docker build -t <имя_образа> .
```
Пример:
```
docker build -t replica_set .
```

5. Авторизуйтесь в Docker Hub. Выполните следующую команду для аутентификации:
```bash
docker login
```

6. Теперь, чтобы загрузить образ на Docker Hub, используйте команду `docker tag` для присвоения тегу образа, который указывает на репозиторий Docker Hub:
```bash
docker tag <имя_локального_образа> <имя_репозитория>/<имя_образа>:<тег>
```
Пример:
```bash
docker tag replica_set user2305/replica_set:v1
```

7. Выполните команду `docker push`, чтобы загрузить образ на Docker Hub:
```bash
docker push <имя_репозитория>/<имя_образа>:<тег>
```
Пример:
```bash
docker push user2305/replica_set:v1
```

### Kuber
#### Запустить кластер
1. Перейти на платформу «Play with Kubernetes» (https://labs.play-with-k8s.com/) и зарегистрироваться.
2. Первым шагом является инициализация кластера. Выполните на master пункт 1 из приветствия:
```
kubeadm init --apiserver-advertise-address $(hostname -i) --pod-network-cidr 10.5.0.0/16 
```
Это займет пару минут, в течение которых вы увидите большую активность в терминале.
В конце вы увидите что-то вроде этого:
![image](https://github.com/user-2305/KuberNet/assets/95847398/88eba9fa-17ba-4fc7-948c-df7bc62ac7ab)
Скопируйте всю строку, начинающуюся kubeadm join из первого терминала, и вставьте ее во второй и третий терминалы.
Пример:
```
kubeadm join 192.168.0.23:6443 --token wng5kx.myjvi1p3vc25i62s \
--discovery-token-ca-cert-hash sha256:dd8138809bf8c943a198c2fe1b394393074da8da50b4c5a6069fb9bd10bb472f
```
Теперь нужно инициализировать сеть кластера в первом терминале, выполнив предписание №2 из сохраненного приветствия:
```
kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml
```
Ваш кластер настроен!

#### Деплой приложения
Давайте развернём первое приложение в Kubernetes с помощью команды kubectl create deployment. Для этого потребуется указать имя деплоймента и путь к образу приложения (используйте полный URL репозитория для образов, которые располагаются вне Docker Hub iteration2020/virtcont3:latest).
```
kubectl create deployment kubernetes-lab1 --image=user2305/replica_set:latest
```
Чтобы увидеть список деплойментов, выполните команду kubectl get deployments:
```
kubectl get deployments
```
Вы увидите, что есть 1 деплоймент, в котором запущен единственный экземпляр приложения. Этот экземпляр работает в контейнере на узле кластера.

![image](https://github.com/user-2305/KuberNet/assets/95847398/f60364ee-7ee7-41b4-a9b6-3cd9783db7ba)

Далее отмасштабируем деплоймент до 3 реплик. Для этого воспользуемся командой kubectl scale, для которой укажем тип объекта (деплоймент), его название и количество желаемых экземпляров:
```
kubectl scale deployments/kubernetes-lab1 --replicas=3
```
Создадим службу типа LoadBalancer:
```
kubectl expose deployment/kubernetes-lab1 --type="LoadBalancer" --port=80
```
«Пропатчим» службу, т.к. система назначения external_IP отсутствует, назначим сами:
```
kubectl patch svc kubernetes-lab1  -p '{"spec": {"type": "LoadBalancer", "externalIPs":["192.166.0.10"]}}'
```
Проверим службу:
```
curl http:// 192.166.0.10
```
Что получаем в итоге:
![image](https://github.com/user-2305/KuberNet/assets/95847398/887be39b-b82f-4164-8c98-bc694df0b652)
![image](https://github.com/user-2305/KuberNet/assets/95847398/a3bf3052-e3e7-4541-92a6-662540561d6d)

При включении балансера мы обращаемся к разным подам

## Задание2
### Kuber
Давайте развернём первое приложение в Kubernetes с помощью команды kubectl create deployment. Для этого потребуется указать имя деплоймента и путь к образу приложения (используйте полный URL репозитория для образов, которые располагаются вне Docker Hub ariannauntilova/attm2:latest).
```
kubectl create deployment kubernetes-lab1part2 --image=user2305/test:latest
```
```
kubectl expose deployment/kubernetes-lab1part2 --type="NodePort" --port=80
```
```
kubectl describe services/kubernetes-lab1part2
```

![image](https://github.com/user-2305/KuberNet/assets/95847398/8518e6eb-7b2e-4df1-a2c8-f39111c13d59)

```
curl http://192.168.0.18:30296
```

![image](https://github.com/user-2305/KuberNet/assets/95847398/a8807e24-041b-458c-a442-14a3e8f84233)

