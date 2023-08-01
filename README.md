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

3. Create the table and insert a sample value
   
   ```sql
   use userapi;

   CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_name VARCHAR(255) NOT NULL,
    user_email VARCHAR(255) NOT NULL,
    user_password VARCHAR(255) NOT NULL
   );
   
   INSERT INTO users (user_name, user_email, user_password) VALUES ('John Doe', 'john.doe@example.com', 'secretpassword');
   ```

4. Deploy our app on the k8s using our app chart
   
   ```bash
   cd charts/app
   helm upgrade --install userapi .
   ```

5. Test the application by creating new test pod 
   
   ```bash
   $ kubectl run --rm -it test --image nginx -- bash
      > curl http://userapi:5000
      > curl http://userapi:5000/users
   ```

----------------------------------------------------------------

## Continuous Integration (CI) and Continuous Delivery (CD) Pipeline:

1. All Helm charts will be stored in a centralized Git repository for version control and collaboration.

2. CI/CD pipelines will automatically build Docker images for each application upon code changes, tagging them with the corresponding application version.
   
   ```bash
   docker build -t userapi:0.1.0 .
   docker build -t userapi:2.2.0 .
   ```

3. The CI pipeline will update Helm charts with the new Docker image tags, ensuring up-to-date charts.

4. Upon successful CI pipeline runs, the CD pipeline will deploy the updated Helm charts to the target Kubernetes environment (e.g., staging, production).
   
   ```bash
   helm upgrade --install userapi --set image.tag="0.1.0" .
   helm upgrade --install userapi --set image.tag="2.2.0" .
   ```

## Conclusion 

1. Set up a Kubernetes cluster.

2. Install Helm on the Kubernetes cluster.

3. Create a Helm chart for the Flask app.

4. Configure the Helm chart with the necessary Kubernetes resources (e.g., Deployment, Service, Ingress).

5. Store the Helm chart in a Git repository.

6. Set up a CI/CD pipeline to build the Docker image of the Flask app on code changes.

7. Tag the Docker image with the appropriate version.

8. Update the Helm chart with the new Docker image tag.

9. Trigger the CD pipeline to deploy the updated Helm chart to the Kubernetes cluster.

10. Verify the deployment and application functionality.