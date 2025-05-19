# ELK Stack (Elasticsearch, Logstash, Filebeat, Kibana) to Collect Logs from Applications On Kubernetes Cluster
In this video, we use the ELK Stack to collect logs of applications deployed on a Kubernetes cluster. 

YouTube Link: https://youtu.be/A3W7ZZsBmn4?si=XOfX-FbMNVHKcxA6

## Requirements

1. Docker Desktop (on Mac & Windows) or Docker Engine (on Linux)
2. Kubectl
3. Kubernetes (Minikube)


## Check Requirements
```sh 
docker --version
```
```sh
kubectl version --client
```
```sh
minikube version
```


## Start Minikube Cluster
```sh
minikube start --cpus=4 --memory=8192 --driver=docker
```
```sh
kubectl get nodes
```


## Deploy Sample Applications in demo-app Namespace
 
```sh
kubectl create namespace demo-apps
```
```sh
kubectl apply -f app1.yaml
```
```sh
kubectl apply -f app2.yaml
```


## Deploy Elasticsearch in logging Namespace
```sh
kubectl create namespace logging
```
```sh
kubectl apply -f elasticsearch.yaml
```

### Verify Elasticsearch Pod & PVC
```sh
kubectl get pods -n logging
```
```sh
kubectl get pvc -n logging
```
```sh
kubectl get pv -n logging
```


## Deploy Kibana in logging Namespace
```sh
kubectl apply -f kibana.yaml
```

### Verify Kibana 
```sh
kubectl get pods -n logging
```


### Expose Kibana UI with Minikube Tunnel (Open URL in browser)
```sh
minikube service kibana -n logging --url
```
 
## Deploy Logstash to:
1. Listen for container logs from Filebeat
2. Parse the logs
3. Send the logs to Elasticsearch


```sh
kubectl apply -f logstash.yaml
```
```sh
kubectl get all -n logging
```


## Deploy Filebeat Daemonset to:
Listen for container logs from /var/log/containers/*.log

```sh
kubectl apply -f filebeat.yaml
```


### Debugging Commands
```sh
kubectl logs app1 -n demo-apps | tail
```
```sh
kubectl logs app2 -n demo-apps | head
```

#### Verify that Elasticsearch Indices
```sh
kubectl run -i --rm --restart=Never curl --image=curlimages/curl -n logging -- curl http://elasticsearch:9200
kubectl run -i --rm --restart=Never curl --image=curlimages/curl -n logging -- curl http://elasticsearch:9200/_cat/indices?v
```

#### Verify log collection by Filebeat
```sh
kubectl exec -n logging -it <filebeat-pod> -- ls /var/log/containers/
```

#### Verify that logstash.conf exists
```sh
kubectl exec -n logging -it <logstash-pod> -- ls /usr/share/logstash/pipeline/
kubectl exec -n logging -it <logstash-pod> -- cat /usr/share/logstash/pipeline/logstash.conf
```


### On Kibana
1. Explore on My Own 
2. Click Home Left Panel
3. Go to Stack Management
4. Click Index Patterns - "create an index pattern against hidden or system indices" name e.g: filebeat-*
5. Select @timestamp in Timestamp field
6. Click create index pattern.
7. Go to Discover on the left panel of homepage to see logs from app1 and app2.


### Verify it from Kibana UI
1. On the left panel (under Available fields)
2. Scroll down to the bottom to see e.g. log message, log.file.path, etc. 
3. Click to examine them.


## Now, Deploy and Log an NGINX application
```sh
kubectl apply -f nginx.yaml
```

### Expose Nginx with Minikube Tunnel (Open URL in browser)
```sh
minikube service nginx-service -n demo-apps --url
```

### Create some Filters in Kibana to examine Nginx logs
1. Add filter
2. Field = kubernetes.labels.app, Operator = is, Value = nginx & Save. (You may have to change timestamp next to the "Refresh button" to see some logs)



## Clean UP
```sh
kubectl delete ns logging
```
```sh
kubectl delete ns demo-apps
```
```sh
minikube stop
```
```sh
minikube delete --all
```
