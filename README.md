# operation

## Run the Docker Compose file

```bash
docker compose up
```

### Accessing the service

One can go to `localhost:8081`, where they can submit a review and receive either a positive or a negative sentiment in response. To see the API specification, one can go to `localhost:6789/apidocs`. There they can also perform individual requests to the various endpoints.

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

## Running Grafana
Apply the grafana.yml with `kubectl apply -f grafana.yml`. Setup port forwarding with `kubectl port-forward service/grafana 3000:3000`. Go to the dashboard at `localhost:3000`.