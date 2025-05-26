# ml-big-data-lab-8

## Цель работы

- Получить навыки оркестрации контейнеров с использованием Kubernetes путём миграции сервиса модели на PySpark, сервиса витрины на Spark и сервиса источника данных.

## Stack

- Scala, Python, Cassandra, PySpark, Kubernetes

## Project Structure

```text
.
├── Dockerfile
├── Dockerfile.clickhouse
├── README.md
├── build.sbt
├── conf
│   └── spark.ini
├── docker-compose.yml
├── entrypoint.sh
├── k8s
│   ├── clickhouse-deployment.yaml
│   ├── model-deployment.yaml
│   └── namespace.yaml
├── notebooks
│   ├── openfoodfacts_clustering.ipynb
│   ├── openfoodfacts_preprocessing.ipynb
│   └── word_count.ipynb
├── project
│   ├── build.properties
│   └── plugins.sbt
├── requirements.txt
├── scripts
│   └── seed_db.sh
├── sparkdata
│   ├── en.openfoodfacts.org.products.csv
│   └── googleplaystore_user_reviews.csv
├── src
│   └── main
│       ├── python
│       │   ├── clusterize.py
│       │   └── logger.py
│       └── scala
│           └── Datamart.scala
└── static
    ├── Лабораторная работа 5 (весна 2025).pdf
    ├── Лабораторная работа 6 (весна 2025).pdf
    ├── Лабораторная работа 7 (весна 2025).pdf
    └── Лабораторная работа 8 (весна 2025).pdf
```

## Steps

- Compile scala code:

```bash
sbt assebmly
```

- Build images:

```bash
docker build -t model:latest .
docker build -t custom-clickhouse:latest -f ./Dockerfile.clickhouse .
```

- Push image to DockerHub:

```bash
# Log in to DockerHub (if not already logged in)
docker login

# Tag your local image with your DockerHub username
docker tag model:latest zarus03/model:latest
docker tag custom-clickhouse:latest zarus03/custom-clickhouse:latest

docker push zarus03/model:latest
docker push zarus03/custom-clickhouse:latest
```

- Start minikube:

```bash
minikube start
```

- Mount sparkdata directory:

```bash
minikube mount ./sparkdata:/sparkdata
```

- Create a spark-app namespace:

```bash
kubectl apply -f ./k8s/namespace.yaml
```

- Verify created namespace:

```bash
kubectl get namespaces
```

- Create ConfigMap from env file:

```bash
kubectl create configmap env-config --from-env-file=.env -n spark-app
kubectl describe configmaps env-config -n spark-app
```

- Apply a configuration:

```bash
kubectl apply -f ./k8s/clickhouse-deployment.yaml
kubectl apply -f ./k8s/model-deployment.yaml
```

- Check pods:

```bash
kubectl get pods -n spark-app
```

## Additional

- Deletes a local Kubernetes cluster. This command deletes the VM, and removes all associated files:

```bash
minikube delete --all --purge
```

- Attach to logs in case of exceptions:

```bash
minikube logs -f
```

- Verify that docker images loaded successfully:

```bash
minikube ssh docker images
```

- Check services:

```bash
kubectl get svc -n spark-app
```

- Delete stuck pods/deployment (set deployment name in place `clickhouse`):

```bash
kubectl delete deployment clickhouse -n spark-app
```

- Execute a command in a container (set pod name in place `clickhouse-xxxx`):

```bash
kubectl exec -it clickhouse-xxxx -n spark-app -- bash
```

- Show logs (set pod name in place `clickhouse-xxxx`):

```bash
kubectl logs -f clickhouse-xxxx -n spark-app
```
