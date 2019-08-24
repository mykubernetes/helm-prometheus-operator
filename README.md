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



在prometheus字段配置秘钥
prometheus:

  enabled: true

  ## Service account for Prometheuses to use.
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
  ##
  serviceAccount:
    create: true
    name: ""

  ## Configuration for Prometheus service
  ##
  service:
    annotations: {}
    labels: {}
    clusterIP: ""


    ## To be used with a proxy extraContainer port
    targetPort: 9090

    ## List of IP addresses at which the Prometheus server service is available
    ## Ref: https://kubernetes.io/docs/user-guide/services/#external-ips
    ##
    externalIPs: []

    ## Port to expose on each node
    ## Only used if service.type is 'NodePort'
    ##
    nodePort: 30090

    ## Loadbalancer IP
    ## Only use if service.type is "loadbalancer"
    loadBalancerIP: ""
    loadBalancerSourceRanges: []
    ## Service type
    ##
    type: ClusterIP

    sessionAffinity: ""

  rbac:
    ## Create role bindings in the specified namespaces, to allow Prometheus monitoring
    ## a role binding in the release namespace will always be created.
    ##
    roleNamespaces:
      - kube-system

  ## Configure pod disruption budgets for Prometheus
  ## ref: https://kubernetes.io/docs/tasks/run-application/configure-pdb/#specifying-a-poddisruptionbudget
  ## This configuration is immutable once created and will require the PDB to be deleted to be changed
  ## https://github.com/kubernetes/kubernetes/issues/45398
  ##
  podDisruptionBudget:
    enabled: false
    minAvailable: 1
    maxUnavailable: ""

  ingress:
    enabled: false
    annotations: {}
    labels: {}

    ## Hostnames.
    ## Must be provided if Ingress is enabled.
    ##
    # hosts:
    #   - prometheus.domain.com
    hosts: []

    ## Paths to use for ingress rules - one path should match the prometheusSpec.routePrefix
    ##
    paths: []
    # - /

    ## TLS configuration for Prometheus Ingress
    ## Secret must be manually created in the namespace
    ##
    tls: []
      # - secretName: prometheus-general-tls
      #   hosts:
      #     - prometheus.example.com

  serviceMonitor:
    ## Scrape interval. If not set, the Prometheus default scrape interval is used.
    ##
    interval: ""
    selfMonitor: true

  ## Settings affecting prometheusSpec
  ## ref: https://github.com/coreos/prometheus-operator/blob/master/Documentation/api.md#prometheusspec
  ##
  prometheusSpec:

    ## Interval between consecutive scrapes.
    ##
    scrapeInterval: ""

    ## Interval between consecutive evaluations.
    ##
    evaluationInterval: ""

    ## ListenLocal makes the Prometheus server listen on loopback, so that it does not bind against the Pod IP.
    ##
    listenLocal: false

    ## Image of Prometheus.
    ##
    image:
      repository: quay.io/prometheus/prometheus
      tag: v2.9.1

    ## Tolerations for use with node taints
    ## ref: https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/
    ##
    tolerations: []
    #  - key: "key"
    #    operator: "Equal"
    #    value: "value"
    #    effect: "NoSchedule"

    ## Alertmanagers to which alerts will be sent
    ## ref: https://github.com/coreos/prometheus-operator/blob/master/Documentation/api.md#alertmanagerendpoints
    ##
    ## Default configuration will connect to the alertmanager deployed as part of this release
    ##
    alertingEndpoints: []
    # - name: ""
    #   namespace: ""
    #   port: http
    #   scheme: http

    ## External labels to add to any time series or alerts when communicating with external systems
    ##
    externalLabels: {}

    ## External URL at which Prometheus will be reachable.
    ##
    externalUrl: ""

    ## Define which Nodes the Pods are scheduled on.
    ## ref: https://kubernetes.io/docs/user-guide/node-selection/
    ##
    nodeSelector: {}

    ## Secrets is a list of Secrets in the same namespace as the Prometheus object, which shall be mounted into the Prometheus Pods.
    ## The Secrets are mounted into /etc/prometheus/secrets/. Secrets changes after initial creation of a Prometheus object are not
    ## reflected in the running Pods. To change the secrets mounted into the Prometheus Pods, the object must be deleted and recreated
    ## with the new list of secrets.
    ##
    secrets:
    - etcd-certs              #添加秘钥，使prom可以和etcd交互

    ## ConfigMaps is a list of ConfigMaps in the same namespace as the Prometheus object, which shall be mounted into the Prometheus Pods.
    ## The ConfigMaps are mounted into /etc/prometheus/configmaps/.
    ##
    configMaps: []

    ## QuerySpec defines the query command line flags when starting Prometheus.
    ## ref: https://github.com/coreos/prometheus-operator/blob/master/Documentation/api.md#queryspec
    ##
    query: {}

    ## Namespaces to be selected for PrometheusRules discovery.
    ## If nil, select own namespace. Namespaces to be selected for ServiceMonitor discovery.
    ## See https://github.com/coreos/prometheus-operator/blob/master/Documentation/api.md#namespaceselector for usage
    ##
    ruleNamespaceSelector: {}

    ## If true, a nil or {} value for prometheus.prometheusSpec.ruleSelector will cause the
    ## prometheus resource to be created with selectors based on values in the helm deployment,
    ## which will also match the PrometheusRule resources created
    ##
    ruleSelectorNilUsesHelmValues: true

    ## PrometheusRules to be selected for target discovery.
    ## If {}, select all ServiceMonitors
    ##
    ruleSelector: {}
    ## Example which select all prometheusrules resources
    ## with label "prometheus" with values any of "example-rules" or "example-rules-2"
```  


3、scheduler  
```
修改scheduler自动发现
# vim values.yaml
kubeScheduler:
  enabled: true

  ## If your kube scheduler is not deployed as a pod, specify IPs it can be found on
  ##
  endpoints:
  - 10.155.20.50                 #填写二进制部署的scheduler地址
  # - 10.141.4.23
  # - 10.141.4.24

  ## If using kubeScheduler.endpoints only the port and targetPort are used
  ##
  service:
    port: 10251
    targetPort: 10251
    selector:
      component: kube-scheduler
```  

4、kubelet
```
修改kubelet自动发现
# vim values.yaml
kubelet:
  enabled: true
  namespace: kube-system

  serviceMonitor:
    ## Scrape interval. If not set, the Prometheus default scrape interval is used.
    ##
    interval: ""

    ## Enable scraping the kubelet over https. For requirements to enable this see
    ## https://github.com/coreos/prometheus-operator/issues/926
    ##
    https: false           #修改http为false
```  

5、修改完配置后更新  
```
# helm upgrad prom ./prometheus-operator/ -f ./prometheus-operator/values.yaml 
```  
