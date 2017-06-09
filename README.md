# Kubernetes in Digital Ocean

Be sure that you have already created an account in Digital Ocean.

Go to the [Digital Ocean Console](https://cloud.digitalocean.com/) and create (for instance) 3 new droplets.

Do not forget to add your ssh public key to each vm and to enable the private network.

## Create the Cluster with Kubeadm

### Create the Cluster

Select the master and connect to it via ssh.

```bash
ssh root@<ip>
```

1. Install kubectl in the master

```bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
```

2. Install kubelet and kubeadm

```bash
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
# Install docker if you don't have it already.
apt-get install -y docker-engine
apt-get install -y kubelet kubeadm kubernetes-cni
```

Once kubectl, kubelet and kubeadm are properly installed in the master, connect to the other nodes and
do the same. (We will see how to create a custom image avoiding to follow these steps any time we want to create a new node.)

3. Initializing the master

```bash
kubeadm init
```

4. Set the KUBECONFIG variable to start using your cluster:

```bash
sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf
```

### Setup the Pod Network (Calico - Kubeadm Hosted Install)

For Kubeadm 1.6 with Kubernetes 1.6.x:

```bash
kubectl apply -f http://docs.projectcalico.org/v2.2/getting-started/kubernetes/installation/hosted/kubeadm/1.6/calico.yaml
```

### Join the Nodes

Connect to each node via ssh and do the following to join the nodes to the Cluster

```bash
kubeadm join --token <token> <master-ip>:<master-port>
```

You can see the token just using the command `kubeadm token list` in the master.

### Check

Check if everthing was fine doing:

`kubectl cluster-info`
`kubectl get nodes`

### Useful commands

```bash
kubectl get nodes
kubectl get deployments
kubectl get services
kubectl describe pod <pod>
kubectl describe service <service>
kubectl logs <pod>
kubectl delete <type> <name>
kubectl expose deployment <deployment> --type=<type>
```

## Microservices

### How to create a Deployment

- Name
- Replicas
- Port
- Image
- Volume

### Services and types of Services

- Defining a ClusterIP Service for the Microservices
- Creating a NodePort with a fixed port for the API GATEWAY

### GRPC

- How to register GRPC Microservices

## Scale the cluter with Terraform

```bash
export DIGITALOCEAN_TOKEN="Put Your Token Here"
terraform plan
terraform apply
```

### Terraform Config files

### Create the base image

## CI

### Jenkins and Jenkins pipelines with Jenkinsfile

### Using an external Container Builder

### Private vs Public Container Registry

## API GATEWAY

### Load Balancing with NGINX

- Proxypass to each NodeIP to the fixed API GATEWAY port

### Istio

#### Installing Istio

1. Get istio: `curl -L https://git.io/getIstio | sh -`

2. Add the istioctl client to your PATH `export PATH=$PWD/bin:$PATH`

3. Run the following command to determine if your cluster has RBAC `kubectl api-versions | grep rbac`

4. It is highly recommended to create a clusterrolebinding `kubectl create clusterrolebinding myname-cluster-admin-binding --clusterrole=cluster-admin --user=myname@example.org`

5. Install the istio-rbac config: `kubectl apply -f install/kubernetes/istio-rbac-alpha.yaml`

6. Install Istio: `kubectl apply -f install/kubernetes/istio.yaml`

Optional:

Enabling Metrics:

```bash
kubectl apply -f install/kubernetes/addons/prometheus.yaml
kubectl apply -f install/kubernetes/addons/grafana.yaml
kubectl apply -f install/kubernetes/addons/servicegraph.yaml
```

#### Enabling Ingress Traffic

1. Start the httpbin sample: `kubectl apply -f <(istioctl kube-inject -f samples/apps/httpbin/httpbin.yaml)`

2. Create the Ingress Resource for the httpbin service:

```bash
cat <<EOF | kubectl create -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: simple-ingress
  annotations:
    kubernetes.io/ingress.class: istio
spec:
  rules:
  - http:
      paths:
      - path: /headers
        backend:
          serviceName: httpbin
          servicePort: 8000
      - path: /delay/.*
        backend:
          serviceName: httpbin
          servicePort: 8000
EOF
```

3. Determine the ingress URL:

Because this cluster is running on Digital Ocean, there is no LoadBalancer service available.

Get the ingress-istio NodeIP:

`kubectl get po -l istio=ingress -o jsonpath='{.items[0].status.hostIP}'`

Get the Port:

`kubectl get svc istio-ingress`

Go to the URL: http://<IP>:<PORT>/headers

[Kubernetes][https://kubernetes.io]
[Calico][http://docs.projectcalico.org/v2.2/getting-started/kubernetes/installation/hosted/kubeadm/]
[Istio][https://istio.io/docs/tasks/installing-istio.html]
[Terraform][https://github.com/hashicorp/terraform/tree/master/examples]
