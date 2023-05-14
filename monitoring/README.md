download: https://github.com/prometheus-operator/prometheus-operator/releases/download/v0.65.1/bundle.yaml 
```
kubectl create -f bundle.yaml
```
this installs the CRDs and deploys the operator
you need serviceMonitors to define targets to be monitored in our case you edit the hlf CRDs with the following configs:
```
  serviceMonitor:
    enabled: true
    interval: 10s
    labels: {}
    sampleLimit: 0
    scrapeTimeout: 10s
```
apply RBACrules in order to give access 
apply prometheus.yaml with  
``` 
    serviceMonitorNamespaceSelector: {}
    serviceMonitorSelector: {}
```
this makes the prometheus select every serviceMonitor
if you want it to select a certain label or a namespace specify it
and then we deploy the grafana with 
```
kubectl apply -f grafana.yaml 
```
for the datasource set it with the name of your prometheus-operated svc with the port 9090