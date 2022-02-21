## Kubernetes Version <1.21.9>, Knative Version <1.2.0>
### First, Set Loadbalancer at <https://github.com/59nezytic/metalLB>, and Install Ingress-controller
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/cloud/deploy.yaml
```

### Install Knative Operator
```
cd Knative
kubectl apply -f operator/operator.yaml
kubectl config set-context --current --namespace=default
kubectl get deployment knative-operator
```
### Install Knative Serving
```
kubectl apply -f serving/kfs.yaml
kubectl get po -n knative-serving
```
### Install Istio
```
curl -L https://istio.io/downloadIstio | sh -
export PATH=$PWD/bin:$PATH
istioctl install --set profile=demo -y
kubectl label namespace default istio-injection=enabled
kubectl get svc istio-ingressgateway -n istio-system
kubectl apply -f istio/serving-default-domain.yaml
```

### Install Knative Eventing
```
kubectl apply -f eventing/kfe.yaml
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

### Kafka (https://knative.dev/docs/eventing/sources/kafka-source/#verify)
* Install Kafka
```
# Install Kafka "Distributed" Channel
kubectl apply -f kafka/channel-distributed.yaml

# Install Eventing Kafka Controller
kubectl apply -f kafka/eventing-kafka-controller.yaml

# Install Eventing Kafka Broker
kubectl apply -f kafka/eventing-kafka-broker.yaml

# Install Kafka Source
kubectl apply -f kafka/kafka-source.yaml

```
* Kafka Example
```
# Setting Kafka Topic
kubectl apply -f kafka/kafka-topic.yaml
kubectl -n kafka get kafkatopics.kafka.strimzi.io

# Setting Kafka Event Display Service
kubectl apply -f kafka/event_display.yaml
kubectl get pods

# Setting Kafka Event Source Service
kubectl apply -f kafka/event_source.yaml
kubectl logs {EVENT_SOURCE_POD_ID}
(Output Exapmle: {"level":"info","ts":"2020-05-28T10:39:42.104Z","caller":"adapter/adapter.go:81","msg":"Starting with config: ","Topics":".","ConsumerGroup":"...","SinkURI":"...","Name":".","Namespace":"."})


# Create Kafka Topic Message
kubectl -n kafka run kafka-producer -ti --image=strimzi/kafka:0.14.0-kafka-2.3.0 --rm=true --restart=Never -- bin/kafka-console-producer.sh --broker-list my-cluster-kafka-bootstrap.my-kafka-project:9092 --topic knative-demo-topic

Enter: {"msg": "This is a test!"}



# Edit configmap "config-logging"
kubectl edit cm -n knative-eventing config-logging

data:
  loglevel.controller: info
  loglevel.webhook: info
  zap-logger-config: |
    {
      "level": "debug",
      "development": false,
      "outputPaths": ["stdout"],
      "errorOutputPaths": ["stderr"],
      "encoding": "json",
      "encoderConfig": {
        "timeKey": "ts",
        "levelKey": "level",
        "nameKey": "logger",
        "callerKey": "caller",
        "messageKey": "msg",
        "stacktraceKey": "stacktrace",
        "lineEnding": "",
        "levelEncoder": "",
        "timeEncoder": "iso8601",
        "durationEncoder": "",
        "callerEncoder": ""
      }
    }



# Restart Kafka Source Service
kubectl delete po {KAFKA_SOURCE_POD_ID}
kubectl logs po {NEW_KAFKA_SOURCE_POD_ID}
(Output Example: {"level":"debug","ts":"2020-05-28T10:40:29.400Z","caller":"kafka/consumer_handler.go:77","msg":"Message claimed","topic":".","value":"."}
{"level":"debug","ts":"2020-05-28T10:40:31.722Z","caller":"kafka/consumer_handler.go:89","msg":"Message marked","topic":".","value":"."})



# Check Message
kubectl logs {KAFKA_EVENT_DISPLAY} -c user-container
(Output Exapmle: 
☁️ cloudevents.Event
Validation: valid
Context Attributes,
  specversion: 1.0
  type: dev.knative.kafka.event
  source: /apis/v1/namespaces/default/kafkasources/kafka-source#my-topic
  subject: partition:0#564
  id: partition:0/offset:564
  time: 2020-02-10T18:10:23.861866615Z
  datacontenttype: application/json
Extensions,
  key:
Data,
    {
      "msg": "This is a test!"
    }
)
```
