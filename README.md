## Knative Version 1.2.0
### First, Set Loadbalancer <https://github.com/59nezytic/metalLB>

### Install Knative Operator
```
kubectl apply -f https://github.com/knative/operator/releases/download/knative-v1.2.0/operator.yaml
kubectl config set-context --current --namespace=default
kubectl get deployment knative-operator
```
### Install Knative Serving
```
kubectl apply -f Knative/kfs.yaml
kubectl get po -n knative-serving
```
### Install Istio
```
curl -L https://istio.io/downloadIstio | sh -
export PATH=$PWD/bin:$PATH
istioctl install --set profile=demo -y
kubectl label namespace default istio-injection=enabled
kubectl get svc istio-ingressgateway -n istio-system
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.2.0/serving-default-domain.yaml
```

### Install Knative Eventing
```
kubectl apply -f kfe.yaml
kubectl get deployment -n knative-eventing
```

### Install kn
```
wget https://github.com/knative/client/releases/download/knative-v1.2.0/kn-linux-amd64
mv kn-linux-amd64 kn
chmod +x kn
mv kn /usr/local/bin
kn version
```
