## 3-tier Monitoring with Prometheus and Grafana

In this project, we delve into using Prometheus for monitoring on different levels and visualizing the data with Grafana:
- Infrastructure Level: CPU, RAM, Network etc
- Platform Level: Kubernetes components
- Application Level: 3rd-party and Own application

Pre-requisites
- Installation of [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html), [Kubectl](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html), [Helm](https://helm.sh/docs/intro/install/) and [EKSCTL](https://github.com/eksctl-io/eksctl/releases)
- Configuration of AWS authentication using access keys

## Step 1: Setup EKS cluster using eksctl 
With the definition file below, we create the cluster
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

Next, we connect to the cluster, add prometheus chart and install prometheus chart for kubernetes into the cluster using the command below:
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

## Step 2: Deploy a microservices application and create a test anomaly
We deploy a microservices application using the comman below:
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

The expression `rate(node_cpu_seconds_total{mode="idle"}[2m])` is a Prometheus query that calculates the per-second rate of change for the `node_cpu_seconds_total` metric with the `mode` label set to `"idle". This type of expression is what is used by grafana to display metrics needed.























 

