安装prometheus-operator  
```
git clone https://github.com/mykubernetes/helm-prometheus-operator.git
helm install ./helm-prometheus-operator/ --name prom --namespace monitoring
```  

二进制安装需要修改配置文件  
1、ControllerManager  
```
修改ControllerManager自动发现
# vim values.yaml
## Component scraping the kube controller manager
##
kubeControllerManager:
  enabled: true

  ## If your kube controller manager is not deployed as a pod, specify IPs it can be found on
  ##
  endpoints:
  - 10.155.20.50                           #填写二进制部署的controller地址
  # - 10.141.4.23
  # - 10.141.4.24

  ## If using kubeControllerManager.endpoints only the port and targetPort are used
  ##
  service:
    port: 10252
    targetPort: 10252
#    selector:                        #注释两行，不需要通过标签匹配
#      component: kube-controller-manager

  serviceMonitor:
    ## Scrape interval. If not set, the Prometheus default scrape interval is used.
    ##
    interval: ""

    ## Enable scraping kube-controller-manager over https.
    ## Requires proper certs (not self-signed) and delegated authentication/authorization checks
    ##
    https: true                      #开启https
```

2、etcd  
```
修改etcd自动发现
# vim values.yaml
  ## If your etcd is not deployed as a pod, specify IPs it can be found on
  ##
  endpoints:
  - 10.155.20.50                     #填写二进制部署的etcd地址
  # - 10.141.4.23
  # - 10.141.4.24

  ## Etcd service. If using kubeEtcd.endpoints only the port and targetPort are used
  ##
  service:
    port: 2379
    targetPort: 2379
    #selector:                       #注释两行，不需要通过标签匹配
    #  component: etcd
  ## Etcd service. If using kubeEtcd.endpoints only the port and targetPort are used
  ##
  service:
    port: 2379
    targetPort: 2379
    #selector:
    #  component: etcd

  ## Configure secure access to the etcd cluster by loading a secret into prometheus and
  ## specifying security configuration below. For example, with a secret named etcd-client-cert
  ##
  ## serviceMonitor:
  ##   scheme: https
  ##   insecureSkipVerify: false
  ##   serverName: localhost
  ##   caFile: /etc/prometheus/secrets/etcd-client-cert/etcd-ca
  ##   certFile: /etc/prometheus/secrets/etcd-client-cert/etcd-client
  ##   keyFile: /etc/prometheus/secrets/etcd-client-cert/etcd-client-key
  ##
  serviceMonitor:
    ## Scrape interval. If not set, the Prometheus default scrape interval is used.
    ##
    interval: ""
    scheme: https                     #修改为https
    insecureSkipVerify: false
    serverName: ""
    caFile: "/etc/prometheus/secrets/etcd-certs/ca.pem"              #配置下边三个etcd的ca、证书和私钥
    certFile: "/etc/prometheus/secrets/etcd-certs/etcd.pem"
    keyFile: "/etc/prometheus/secrets/etcd-certs/etcd-key.pem"


创建etcd证书  
# kubectl create secret generic etcd-certs -n monitoring --from-file=/etc/kubernetes/pki/ca.pem --from-file=/etc/kubernetes/pki/etcd-key.pem --from-file=/etc/kubernetes/pki/etcd.pem
```  
