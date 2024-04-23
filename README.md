## 3-tier Monitoring with Prometheus and Grafana

In this project, we delve into using Prometheus for monitoring on different levels and visualizing the data with Grafana:
- Infrastructure Level: CPU, RAM, Network etc
- Platform Level: Kubernetes components
- Application Level: 3rd-party and Own application

Pre-requisites
- Installation of [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html), [Kubectl](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html), [Helm](https://helm.sh/docs/intro/install/) and [EKSCTL](https://github.com/eksctl-io/eksctl/releases)
- Configuration of AWS authentication using access keys

## Step 1: Kubernetes Level Monitoring 
With the configuration file below, we first create the cluster using eksctl
```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: my-eks-cluster
  region: us-east-1

nodeGroups:
  - name: ng-1
    instanceType: t3.medium
    desiredCapacity: 2
    minSize: 2
    maxSize: 4
    volumeSize: 40
    volumeType: gp2
    tags:
      nodegroup-type: worker
```
```
eksctl create cluster -f eks-cluster.yaml
```
![Screenshot (600)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/b6935571-13de-4285-b702-b97d4c410c78)

Next, we connect to the cluster, add prometheus chart and install prometheus chart for kubernetes into the cluster using the commands below:
```
aws eks update-kubeconfig --name my-eks-cluster --region us-east-1
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring
```
![Screenshot (604 1)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/8a460784-ed2f-4366-9b6f-18763ba777b6)

![Screenshot (605)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/6268301e-1e67-4648-be9a-f324bd0857fa)

![Screenshot (607)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/ae3d6b86-197f-433c-8206-9269fbc6fce0)

Next, we update the Prometheus and Grafana services to use LoadBalancer to enable us access them through browser

![Screenshot (610)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/5c306572-37e8-414f-a040-3e73347473ba)

![Screenshot (611)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/913ffbb5-30ca-440e-8428-3d8bb113a163)

We then access UI for Prometheus and Grafana (with required credentials)

![Screenshot (612)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/ee6f7797-20cd-4da8-aaab-d0c1e09ff262)

![Screenshot (614)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/54a4db38-c954-422a-bf3a-afe64004bf1d)

![Screenshot (615)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/91b7407b-6920-4615-adcf-f037ad0e121a)

![Screenshot (617)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/4f246a31-ad26-4a6f-84a0-c5ff43093137)

Below are the metrics obtained from the clusters including the nodes also

![Screenshot (616)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/9ce619ae-19b1-40a1-b9d7-60adfb907fb8)

![Screenshot (627)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/c4e7c942-4427-4731-8f7e-b3455359163e)

Next, we deploy a microservices application and create a test anomaly using the commands below:
```
kubectl apply -f config-microservices.yaml
```

![Screenshot (620)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/b22a7056-e3f6-463f-b289-f2990839a08a)

![Screenshot (621)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/f7baf7c4-096a-46cb-844e-b5ff7e1ea5e9)

Next, we execute a script that hits the endpoint of the microservices application in a loop using the curl command:
```
kubectl run curl-test --image=radial/busyboxplus:curl -i --tty --rm
vi test.sh
```
```
for i in $(seq 1 10000)
do
  curl <frontend-microservice-endpoint> > test.txt
done
```
![Screenshot (624)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/c532468d-3f64-4651-95fb-6491d6d2a06b)

![Screenshot (625)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/7231c6b9-bbed-435b-be47-57707da9900c)

![Screenshot (629)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/0ea18c7c-1bb6-4d2f-a375-b9afe5aadcbd)

The expression `rate(node_cpu_seconds_total{mode="idle"}[2m])` is a Prometheus query that calculates the per-second rate of change for the `node_cpu_seconds_total` metric with the `mode` label set to `"idle". This type of expression is used by Grafana to display metrics needed.

![Screenshot (628)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/33acac47-b711-4861-8a7f-fbc98dbdd013)

Next, we create alert rules to notify us when something unusual occurs
- We define what we want to be notified about e.g when CPU usage is above is 50%
- Then, send notification i.e email/slack notification (will be integrated later)

Here, we set the alert rules and prometheusrule handles the rules

![Screenshot (634)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/0de48860-6da3-49d1-9d5c-6f556adca443)

When setting up these rules, we ensure the appropriate labels as noted in the prometheusrule are included in the alert rules. Here, `app=kube-prometheus-stack` and `release=monitoring` labels are noted.

![Screenshot (636)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/488968da-92db-4387-8ea4-3334c3649948)

For illustration, we want to be notified when CPU usage is high and when pods cannot start. Hence, we create the `HostHighCPULoad` and `down` alert rules.

![Screenshot (723)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/320bf45a-0d7f-4cbf-9b57-62874eb26b05)

![Screenshot (724)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/8a091ed0-6f85-49cf-9cd3-f02175d915a1)

After the application of the rules, we check prometheus log of config-reloader container to confirm a reload has triggered(which implies the rule has been added) using the commands below
```
kubectl logs prometheus-monitoring-kube-prometheus-prometheus-0 -n monitoring -c config-reloader | grep Reload
kubectl get prometheusrule -n monitoring | grep -E 'main-rules|up-rules'
```

![Screenshot (643)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/9eb4cbd6-aa6a-4435-8187-c876f0e83970)

From Prometheus UI, we see the alerts have been updated

![Screenshot (640)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/1e431896-2050-46fd-b49d-72fb5c71fd30)

![Screenshot (641)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/3701910c-5dae-4a23-bbaa-240df9adfd71)

Next, we run cpustress container to generate CPU load and see if the alert will be triggered with the command below:
```
kubectl run cpu-test --image=containerstack/cpustress -- --cpu 4 --timeout 60s --metrics-brief
```

![Screenshot (644)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/eb3b21f1-587e-46ed-9857-7e1576829623)

![Screenshot (645 1)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/8410c024-7a16-4e13-bab1-de302943834e)

## Step 2: Monitoring 3rd party application - Redis
After completing the Kubernetes-level monitoring, we next monitor Redis, a 3rd party application within our microservices. To achieve this, we use an exporter. The exporter gets metrics data from the service, translates these service specific metrics to Prometheus understandable metrics and exposes these translated metrics under `/metrics` endpoint. To ensure Prometheus is aware of this new exporter, we will deploy a ServiceMonitor, a custom Kubernetes resource designed for this purpose.

Deploy redis exporter, ensure you set ServiceMonitor to true and specify the redis service name in the helms value (in the redis-values.yaml)

![Screenshot (725)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/4b7a669d-d1af-499e-bb0a-4c0f70c53bbc)

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update
helm install redis-exporter prometheus-community/prometheus-redis-exporter -f redis-values.yaml
```

![Screenshot (647)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/85dcebba-c2f7-4ec4-8b03-697f4e2f9f7d)

![Screenshot (648)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/e3184b65-85d8-436e-bec5-db0c2aec5cb2)

Redis-exporter is deployed by checking the targets under the status section in the Prometheus UI and we can query different redis metrics.

![Screenshot (649 1)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/e3c52c0c-1405-43b1-9e2a-f46afd4a5fd0)

![Screenshot (650 1)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/073f641c-36b1-4b29-94ac-eaaece279ff9)

We observe the redis-exporter pod and ServiceMonitor details

![Screenshot (653)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/d279592a-a9de-4483-8c61-257165bdcd91)

![Screenshot (654)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/75892d11-a342-42fd-8e21-598355b47ece)

We can setup a prometheusrule using ready made rules from [Exporter Rules](https://samber.github.io/awesome-prometheus-alerts/) to define the rules to specify for our exporters. For our redis application, we will be using [Redis rule](https://samber.github.io/awesome-prometheus-alerts/rules#redis) as in the screenshot below.

![Screenshot (656)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/6ce6748b-7000-4d96-9385-d1d8775b8f3f)

```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: redis-rules
  labels:
    app: kube-prometheus-stack 
    release: monitoring
spec:
  groups:
  - name: redis.rules
    rules:
    - alert: RedisDown
      expr: redis_up == 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Redis down (instance {{ $labels.instance }})
        description: "Redis instance is down\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
    - alert: RedisTooManyConnections
      expr: redis_connected_clients > 100
      for: 15s
      labels:
        severity: warning
      annotations:
        summary: Redis too many connections (instance {{ $labels.instance }})
        description: "Redis instance has {{ $value }} connections\n LABELS = {{ $labels }}"
```

![Screenshot (658)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/3ca1bcb2-8d7d-4f73-965f-668b53c034ff)

![Screenshot (659 1)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/b06cf7a2-7d7c-46e6-bc7d-0565688b8cd5)

Next, we scale the pod replica to zero to observe the alert triggered
```
kubectl scale deployment redis-cart --replicas=0
```

![Screenshot (662)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/a ssets/88329191/c568afc2-e3ea-4524-b67a-4c6a868e2b19)

![Screenshot (661 1)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/aa307919-6b87-4510-a0e7-1e27ab2f60a5)

Now, lets try and create a dashboard for our redis application. We can make use of ready made dashboards from [Grafana](https://grafana.com/grafana/dashboards/). Here, we will be making use of one of the numerous dashoards for [Redis](https://grafana.com/grafana/dashboards/11835-redis-dashboard-for-prometheus-redis-exporter-helm-stable-redis-ha/)

![Screenshot (663)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/ac22ffe5-abf8-4a22-b18b-dc57b2d95715)

![Screenshot (664)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/d1436d0a-46cb-4296-bc8d-042b5ca2bb34)

![Screenshot (665)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/2b15f72c-6a60-4ed7-b1d4-77e9fb4583cf)

![Screenshot (666)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/4de67a39-9b76-4b6d-a812-49c0fed0b8c7)

![Screenshot (669)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/17a4f575-0778-41de-80f2-2ae8339f9457)

## Step 3: Monitoring own application
Finally, we monitor our own application deployed in the cluster and we are using a nodejs application. There are no exporters available for our own application, so we have to define the metrics. There are prometheus client libraries for different languages in which the application is written in. Developers write the code using the prometheus client library. The screenshot shows metrics to monitor in the application.

To-do
- Expose metrics for our Nodejs app using Nodejs client library
- Deploy Nodejs app in the cluster
- Configure Prometheus to scrape new target(ServiceMonitor)
- Visualize scraped metrics in Grafana dashboard

![Screenshot (727)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/2b0404be-e03d-42b6-b033-c8dab9ed2415)

Next, we setup a ServiceMonitor with the appropriate label for the application `nodeapp` and prometheus `release=monitoring` respectively so that the targets can be included in prometheus for monitoring

![Screenshot (729)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/4f2f4ca8-cba0-451e-bc33-9a09906bf004)

![Screenshot (730)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/28c19a23-f4fc-45d2-ac88-2d72fe8d5f5f)

The targets have been included and we can query metrics for the application

![Screenshot (704)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/20ac8ca8-8e33-45c5-b8bf-f580f5c9c029)

![Screenshot (705)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/64921a1c-9713-4c91-b589-989e3d1b7955)

![Screenshot (706)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/1dc83cdc-afcc-4116-a899-329a9f0a9128)

![Screenshot (707)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/c3e3ed81-9e93-429b-8ea1-f33a8eb9ca2f)

We define the metrics, `rate(http_request_operations_total{container="nodeapp"}[5m])` and `rate(http_request_duration_seconds_sum{container="nodeapp"}[2m])`  we want to monitor and setup a Grafana dashboard to display the metrics.

![Screenshot (709)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/32186cee-17fc-42c8-b193-7607e467239b)

![Screenshot (711)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/99707da6-5509-4a70-8888-4621a9391d63)

![Screenshot (712)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/48e1c80d-7892-40e8-afc5-2d4cc34e271c)

![Screenshot (713)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/382d5102-c343-4d18-9a8f-b0f62abc6e6d)

![Screenshot (714)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/9e5d8a7d-1d86-4d14-8d92-401b911b0a32)

![Screenshot (715)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/489fd486-9bf1-4c30-9989-071f7dd4f717)

![Screenshot (716)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/1e8d0c72-6581-459f-8dd0-a88be8e6ff44)

![Screenshot (718)](https://github.com/kenchuks44/3-tier-Monitoring-with-Prometheus-and-Grafana/assets/88329191/c5de166e-827c-44b7-8412-98f951852630)

## Congratulations











































































 

