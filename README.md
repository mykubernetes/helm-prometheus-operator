安装prometheus-operator  
```
git clone https://github.com/mykubernetes/helm-prometheus-operator.git
helm install ./helm-prometheus-operator/ --name prom --namespace monitoring
```  