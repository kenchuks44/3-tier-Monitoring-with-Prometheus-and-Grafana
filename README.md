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










 

