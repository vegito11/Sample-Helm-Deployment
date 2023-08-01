## Prerequisite software/packages

- Docker
- Helm
- Kubectl
- Minikube/k8s cluster

--------------------------------------------

1. Build and Push application image

    - Build A docker image  
    
      ```bash
      cd app
      docker build -t userapi .
      ```
  - Push it to dockerhub or ECR
     
     ```bash
     DOCKERHUB_USERNAME="vegito"
     REPO_IMAGE_NAME="${DOCKERHUB_USERNAME}/userapi"
     docker login --username $DOCKERHUB_USERNAME
     docker tag userapi ${REPO_IMAGE_NAME}:latest
     docker push ${REPO_IMAGE_NAME}:latest
     ```
2. Setup Mysql Db Using Helm Chart
    
    ```bash
    helm repo add bitnami https://charts.bitnami.com/bitnami
    helm repo update
    helm install my-mysql --values charts/mysql.yml bitnami/mysql
    ```
    Test this mysql database using below command

    ```bash
    $ MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default my-mysql -o jsonpath="{.data.mysql-root-password}" | base64 -d)
    $ kubectl run my-mysql-client --rm --tty -i --restart='Never' --image \
      docker.io/bitnami/mysql:8.0.34-debian-11-r8 --namespace default \
      --env MYSQL_ROOT_PASSWORD=$MYSQL_ROOT_PASSWORD --command -- bash
    
    $ mysql -h my-mysql.default.svc.cluster.local -uroot -p"$MYSQL_ROOT_PASSWORD"
    ```


3. Deploy our app on the k8s using our app chart
   
   ```bash
   cd charts/app
   helm upgrade --install userapi
   ```

4. Test the application by creating new test pod 
   
   ```bash
   $ kubectl run --rm -it test --image nginx -- bash
   ```