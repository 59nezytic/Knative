## Knative Version <1.2.0>
### First, Set Loadbalancer <https://github.com/59nezytic/metalLB>
```
cd Knative
```

### Install Knative Operator
```
kubectl apply -f operator.yaml
kubectl config set-context --current --namespace=default
kubectl get deployment knative-operator
```
### Install Knative Serving
```
kubectl apply -f kfs.yaml
kubectl get po -n knative-serving
```
### Install Istio
```
curl -L https://istio.io/downloadIstio | sh -
export PATH=$PWD/bin:$PATH
istioctl install --set profile=demo -y
kubectl label namespace default istio-injection=enabled
kubectl get svc istio-ingressgateway -n istio-system
kubectl apply -f serving-default-domain.yaml
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

### Knative Tutorial
https://knative.dev/docs/getting-started/first-service/
