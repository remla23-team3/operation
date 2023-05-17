# operation

## Run the Docker Compose file

```bash
docker compose up
```

### Accessing the service

One can go to `localhost:8081`, where they can submit a review and receive either a positive or a negative sentiment in response. To see the API specification, one can go to `localhost:6789/apidocs`. There they can also perform individual requests to the various endpoints.

## Packaging as a Helm chart

To package the application as a Helm chart, run the following command.

```bash
helm package ./remla23-team3
```

Then, to create an updated `index.yaml` file, run the following command:

```bash
helm repo index --url https://remla23-team3.github.io/operation/ .
```

This will create a `remla23-team3-x.x.x.tgz` file, and push this to the repository to update the Helm chart. The URL in the previous command is the link to the Github pages of the repository, which will serve as the Helm repository when installing the Helm chart.

## Installing as a Helm Chart

Firstly, you have to add our repository as a Helm repository to be able to install it. Run the following command and it should show you the output.

```bash
~ $ helm repo add remla23-team3-repo https://remla23-team3.github.io/operation/
"remla23-team3-repo" has been added to your repositories
```

Following, you can install the Helm chart with the following command and it will show you the given output.

```bash
~ $ helm install remla23-team3 remla23-team3-repo/remla23-team3
NAME: remla23-team3
LAST DEPLOYED: Wed May 17 14:14:07 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

To ensure all services are correctly start, you can run the following command and compare the output.

```bash
~ $ kubectl get services
NAME                                      TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
...
app-service                               LoadBalancer   xxx.xxx.xxx.xxx  <pending>     8081:31636/TCP               1s
grafana                                   LoadBalancer   xxx.xxx.xxx.xxx  <pending>     3000:30015/TCP               1s
model-service                             LoadBalancer   xxx.xxx.xxx.xxx  <pending>     6789:32346/TCP               1s
...
```

### Accessing the service

To port-forward the services, run `minikube tunnel` in a separate terminal window. Then, to access the service, one can visit `http://localhost` for the application and `http://localhost:3000` for the dashboard.

## Kubernetes

Before you can start the services, one needs to create the credentials to pull the images from the Github Crate Registry. Do this with the following command.

```bash
kubectl create secret docker-registry registry-credentials \
--docker-server=ghcr.io \
--docker-username=<github_username> \
--docker-password=<personal_access_token>
```

To start the services one has to run the following command.

```bash
kubectl apply -f kubernetes.yml
```

When you run this command it should show that it creates a number of deployments, services and an ingress. It should precisely show the following.

```bash
deployment.apps/model-deployment created
service/model-service created
deployment.apps/app-deployment created
service/app-service created
ingress.networking.k8s.io/app-ingress created
```

To check all services, deployments and the ingress are running, one can follow these steps.

```bash
~ $ kubectl get services
NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
...
app-service             ClusterIP   xxx.xxx.xxx.xxx <none>        8081/TCP                     60s
model-service           ClusterIP   xxx.xxx.xxx.xxx <none>        6789/TCP                     60s
...

~ $ kubectl get pods
NAME                                                     READY   STATUS             RESTARTS          AGE
...
app-deployment-xxxxxxxxx-xxxxx                           1/1     Running            0                 60s
model-deployment-xxxxxxxxx-xxxxx                         1/1     Running            0                 60s
...

~ $ kubectl get ingress
NAME          CLASS   HOSTS       ADDRESS         PORTS   AGE
app-ingress   nginx   localhost   xxx.xxx.xxx.xxx 80      60s
```

The main two important parts are in the second and third command output.

Firstly, the status of the `app-deployment` and the `model-deployment` should be `RUNNING`. If it is any different, run the command `kubectl describe pod app-deployment-xxxxxxxxx-xxxxx` to debug the what went wrong with pulling the image. If the `STATUS` either images is `ErrImagePull`, you probably misconfigured the registry credentials and should ensure the credentials are correct.

Secondly, there should be an IP address listed under `ADDRESS` for the ingress.

The only thing left to do is to create a minikube tunnel to access the services on `localhost`. You can do this by running the following command in a separate terminal.

```bash
minikube tunnel
```

### Accessing the service

You can access the service by visiting `http://localhost`.