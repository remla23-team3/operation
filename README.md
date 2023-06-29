# Operation 
# Prerequisites for each assignment

Make sure you have installed minikube!
Create the credentials to pull the images from the Github Container Registry. Do this with the following command.

```bash
kubectl create secret docker-registry registry-credentials \
--docker-server=ghcr.io \
--docker-username=<github_username> \
--docker-password=<personal_access_token>
```

# Assignment 3
Specific prerequisites: You should have a working istio installation on your cluster
and be able to access istioctl from the command line! 

## Traffic Management and Continuous Experimentation

In order to deploy two versions of app - v0.0.4 and v0.0.5 (the version with red buttons), one can run:
```bash
kubectl apply -f continuous-experimentation.yml
```
and visit `http://localhost`. Refreshing the page a few times will display the two versions.
The weights for both versions are set to 50/50 in order to easily check the expected behaviour.
You can perform a few requests and also check the configuration and the traffic on the Kaili dashboard
by selecting the default namespace and Graph tab on the right.

### Stabilize the subset of requests
To check the stabilization of the requests, execute the following commands:
```bash
kubectl delete -f continuous-experimentation.yml

kubectl apply -f cont-experimentation-stabilize-requests.yml
```
Next, open Postman and create a GET request to the following address `http://localhost`.
Select the Headers tab and create a new key - `user` and give it a value `user-red-button`.
After the request is sent, you should now receive the html of the page with the red button style.
You can confirm this by finding the `.big-red-button` style in the head tag.

### Verify Prometheus
First make sure that you have applied the prometheus.yaml in `istio-1.17.2/samples/addons/`.
You can access it as follows:
```bash
istioctl dashboard prometheus
```
You can now search for the metrics by typing `remla23_team3:` in the expression text box.
You can also see the full list of metrics on the metrics endpoint `http://localhost/metrics`.

### Experimental setup
For conducting an experiment to find out which version of the app gives better results, one
could make use of the `remla23_team3:feedback_percentage`. A high percentage indicates that
many users leave feedback after submitting a review, which could be beneficial for the product.

### Continuous Experimentation - Two services
Make sure to delete the previous deployment. You can test the two services configuration by running:
```bash
kubectl apply -f continuous-experientation-two-services.yml
```
On the Kaili dashboard you can observe the two services, each one connected to a different version of the app.

## Additional Use Case
We decided to implement `Rate Limiting`, which blocks the user when more than 10 requests are sent per minute.
To check this use case, again make sure you have deleted all other deployments and run:
```bash
kubectl apply -f rate-limiting.yml
```
Now either try to send 10 requests with Postman (it is easier) or try to perform 10 requests in the browser.
After you have performed 10 requests, the website should stop loading, and you should get a response of `local_rate_limited`.
Wait a minute and reload, the website will work again!

# Assignment 2

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

### Accessing the Alert Manager

To access the alert manager, you have to install the kube-prometheus-stack. 

```bash
helm repo add prom-repo https://prometheus-community.github.io/helm-charts
helm repo update
helm install myprom -f rule_file.yaml prom-repo/kube-prometheus-stack
```

The `rule_file.yaml` can be found in the repository's root. 
By port-forwarding the pod usually called `prometheus-myprom-kube-prometheus-sta-prometheus-0`, you can access the alerts in Prometheus' Alerts section. 
One rule is set up in the `rule_file.yaml`. 
It will fire when the feedback of the users drops below 10%, indicating a need to investigate why the model is making bad predictions.

## Kubernetes

Don't forget to create the credentional, look in the Prerequisites.


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

#### Alternative for Linux
If you are using Linux and localhost returns a refused connection, tunneling has failed. You can use an alternative approach
by navigating to the external IP of app-service, which can be found with `kubectl get svc`.

### Access Prometheus and check metrics
In order to access Prometheus, you will need to first install the Promethues stack - kube-prometheus-stack package on Artifact Hub. 
The following commands briefly introduce how to do that:

```bash
$ helm repo add prom-repo https://prometheus-community.github.io/helm-charts
"prom-repo" has been added to your repositories

$ helm repo update
... Successfully got an update ...

$ helm install myprom prom-repo/kube-prometheus-stack
...
```

Once you have installed the prometheus stack, you can check that you have Prometheus by running again:

```bash
~ $ kubectl get services
```
You should have a few new myprom-.. services.

Next, delete and apply the monitoring.yml file as follows:

```bash
kubectl delete -f kubernetes.yml

kubectl apply -f monitoring.yml
```
Don't forget to have the tunnel running in a separate terminal and finally you can access Prometheus through the url that will be shown after running:

```bash
$ minikube service myprom-kube-prometheus-sta-prometheus --url
```
Once, your Prometheus has loaded, you can look into querying the metrics. For this, switch the view from Table to Graph.

Enter one of the metrics as a query (e.g., remla23_team3:num_sentiment_total_requests) and press Execute. This will show the number of requests made.
The metrics which are identified with remla23_team3:... are generated from the webapp.

### Accessing Grafana
Apply the grafana.yml:
```bash
kubectl apply -f remla23-team3/templates/grafana.yml
```

Navigate to dashboards and import the dashboard called `restaurant_dashboard.yml`. Now you can see the metrics of the app in real time in your grafana dashboard.

# Assignment 1

## Run the Docker Compose file

```bash
docker compose up
```

### Accessing the service

One can go to `localhost:8081`, where they can submit a review and receive either a positive or a negative sentiment in response. To see the API specification, one can go to `localhost:6789/apidocs`. There they can also perform individual requests to the various endpoints.
