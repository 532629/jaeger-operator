curl  http://www.google.com

First try with Helm chart
--------------------------
root@automationmaster1:/home/ubuntu# helm repo add jaegertracing https://jaegertracing.github.io/helm-charts
"jaegertracing" has been added to your repositories
root@automationmaster1:/home/ubuntu#


root@automationmaster1:/home/ubuntu# helm repo list
NAME            URL
bitnami         https://charts.bitnami.com/bitnami
stable          https://kubernetes-charts.storage.googleapis.com
jaegertracing   https://jaegertracing.github.io/helm-charts
root@automationmaster1:/home/ubuntu#

root@automationmaster1:/home/ubuntu/jaeger# helm repo add jaegertracing https://jaegertracing.github.io/helm-charts
Error: looks like "https://jaegertracing.github.io/helm-charts" is not a valid chart repository or cannot be reached: Get https://jaegertracing.github.io/helm-charts/index.yaml: dial tcp 185.199.111.153:443: i/o timeout
root@automationmaster1:/home/ubuntu/jaeger#


root@automationmaster1:/home/ubuntu/jaeger# helm install helm-jaeger-operator -n tracing jaeger-operator
manifest_sorter.go:175: info: skipping unknown hook: "crd-install"
NAME: helm-jaeger-operator
LAST DEPLOYED: Fri Sep  4 14:07:50 2020
NAMESPACE: tracing
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
jaeger-operator is installed.


Check the jaeger-operator logs
  export POD=$(kubectl get pods-l app.kubernetes.io/instance=helm-jaeger-operator -lapp.kubernetes.io/name=jaeger-operator --namespace tracing --output name)
  kubectl logs $POD --namespace=tracing
  
root@automationmaster1:/home/ubuntu/jaeger# kubectl get pods -n tracing
NAME                                    READY   STATUS    RESTARTS   AGE
helm-jaeger-operator-554467bd7b-6ggww   1/1     Running   0          111s

root@automationmaster1:/home/ubuntu/jaeger# kubectl get svc -n tracing
NAME                           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
helm-jaeger-operator-metrics   NodePort   10.105.57.133   <none>        8383:30037/TCP   2m

root@automationmaster1:/home/ubuntu/jaeger# kubectl get crd -n tracing
NAME                       CREATED AT
jaegers.jaegertracing.io   2020-09-03T23:13:51Z
root@automationmaster1:/home/ubuntu/jaeger#


root@automationmaster1:/home/ubuntu/jaeger/jaeger-instance# kubectl create -f simplest.yaml
jaeger.jaegertracing.io/simplest created
root@automationmaster1:/home/ubuntu/jaeger/jaeger-instance#

root@automationmaster1:/home/ubuntu/jaeger/jaeger-instance# kubectl get pods -n tracing
NAME                                    READY   STATUS    RESTARTS   AGE
helm-jaeger-operator-554467bd7b-6ggww   1/1     Running   0          5m53s
simplest-5d674b68b8-stgnb               1/1     Running   0          30s


root@automationmaster1:/home/ubuntu/jaeger/git/jaeger-operator/deploy/examples# kubectl get svc -n tracing
NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                                  AGE
helm-jaeger-operator-metrics   NodePort    10.105.57.133    <none>        8383:30037/TCP                           11m
hotrodweb-svc                  NodePort    10.111.3.118     <none>        80:30545/TCP                             9s
simplest-agent                 ClusterIP   None             <none>        5775/UDP,5778/TCP,6831/UDP,6832/UDP      6m20s
simplest-collector             ClusterIP   10.111.115.151   <none>        9411/TCP,14250/TCP,14267/TCP,14268/TCP   6m20s
simplest-collector-headless    ClusterIP   None             <none>        9411/TCP,14250/TCP,14267/TCP,14268/TCP   6m20s
simplest-query                 NodePort    10.105.31.16     <none>        16686:30017/TCP                          6m20s
root@automationmaster1:/home/ubuntu/jaeger/git/jaeger-operator/deploy/examples#

root@automationmaster1:/home/ubuntu/jaeger/git/jaeger-operator/deploy/examples# kubectl get ingress -n tracing
NAME             CLASS    HOSTS   ADDRESS   PORTS   AGE
simplest-query   <none>   *                 80      12m
root@automationmaster1:/home/ubuntu/jaeger/git/jaeger-operator/deploy/examples#


------------------------------------------------------------------------------------------------------------------------------------------------------------

Installing the Jaeger Operator
--------------------------------
kubectl create namespace observability
kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/crds/jaegertracing.io_jaegers_crd.yaml
kubectl create -n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/service_account.yaml
kubectl create -n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/role.yaml
kubectl create -n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/role_binding.yaml
kubectl create -n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/operator.yaml

kubectl delete-n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/operator.yaml
root@automationmaster1:/home/ubuntu/jaeger/git/jaeger-operator/deploy/examples# kubectl create -n observability -f operator-with-tracing.yaml
deployment.apps/jaeger-operator created
root@automationmaster1:/home/ubuntu/jaeger/git/jaeger-operator/deploy/examples#


root@automationmaster1:/home/ubuntu/jaeger# kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/crds/jaegertracing.io_jaegers_crd.yaml
customresourcedefinition.apiextensions.k8s.io/jaegers.jaegertracing.io created
root@automationmaster1:/home/ubuntu/jaeger# kubectl create -n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/service_account.yaml
serviceaccount/jaeger-operator created
root@automationmaster1:/home/ubuntu/jaeger# kubectl create -n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/role.yaml
role.rbac.authorization.k8s.io/jaeger-operator created
root@automationmaster1:/home/ubuntu/jaeger# kubectl create -n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/role_binding.yaml
rolebinding.rbac.authorization.k8s.io/jaeger-operator created
root@automationmaster1:/home/ubuntu/jaeger# kubectl create -n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/operator.yaml
deployment.apps/jaeger-operator created
root@automationmaster1:/home/ubuntu/jaeger#


kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/cluster_role.yaml
kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/cluster_role_binding.yaml

root@automationmaster1:/home/ubuntu/jaeger# kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/cluster_role.yaml
clusterrole.rbac.authorization.k8s.io/jaeger-operator created
root@automationmaster1:/home/ubuntu/jaeger# kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/cluster_role_binding.yaml
clusterrolebinding.rbac.authorization.k8s.io/jaeger-operator created
root@automationmaster1:/home/ubuntu/jaeger#

root@automationmaster1:/home/ubuntu/jaeger# kubectl get deployment jaeger-operator -n observability
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
jaeger-operator   1/1     1            1           2m33s
root@automationmaster1:/home/ubuntu/jaeger#


root@automationmaster1:/home/ubuntu/jaeger/git/jaeger-operator/deploy/examples# kubectl get pods -n observability
NAME                               READY   STATUS    RESTARTS   AGE
example-statefulset-0              2/2     Running   0          3m35s
jaeger-operator-7775447c77-89m92   2/2     Running   0          15m
simplest-77b8c6fc95-x82bs          1/1     Running   0          14m
root@automationmaster1:/home/ubuntu/jaeger/git/jaeger-operator/deploy/examples#


root@automationmaster1:/home/ubuntu/jaeger# kubectl get jaeger -n observability
NAME       STATUS    VERSION   STRATEGY   STORAGE   AGE
simplest   Running   1.19.2    allinone   memory    14s
root@automationmaster1:/home/ubuntu/jaeger#



root@automationmaster1:/home/ubuntu/jaeger/git/jaeger-operator/deploy/examples# kubectl get pods  -l  app.kubernetes.io/instance=simplest -n observability
NAME                        READY   STATUS    RESTARTS   AGE
simplest-574989b7bf-p4c5p   1/1     Running   0          78m
root@automationmaster1:/home/ubuntu/jaeger/git/jaeger-operator/deploy/examples# kubectl logs  -l  app.kubernetes.io/instance=simplest -n observability
{"level":"info","ts":1599184515.3705988,"caller":"app/flags.go:103","msg":"Archive storage not initialized"}
{"level":"info","ts":1599184515.3712559,"caller":"app/agent.go:69","msg":"Starting jaeger-agent HTTP server","http-port":5778}
{"level":"info","ts":1599184515.3721042,"caller":"base/balancer.go:196","msg":"roundrobinPicker: newPicker called with info: {map[0xc0009dd4a0:{{:14250  <nil> 0 <nil>}}]}","system":"grpc","grpc_log":true}
{"level":"info","ts":1599184515.37227,"caller":"app/static_handler.go:180","msg":"watching","file":"/etc/config/ui.json"}
{"level":"info","ts":1599184515.372292,"caller":"app/static_handler.go:188","msg":"watching","dir":"/etc/config"}
{"level":"info","ts":1599184515.3723853,"caller":"app/server.go:114","msg":"Query server started","port":16686,"addr":":16686"}
{"level":"info","ts":1599184515.3724585,"caller":"healthcheck/handler.go:128","msg":"Health Check state change","status":"ready"}
{"level":"info","ts":1599184515.372485,"caller":"app/server.go:152","msg":"Starting CMUX server","port":16686,"addr":":16686"}
{"level":"info","ts":1599184515.372489,"caller":"app/server.go:142","msg":"Starting GRPC server","port":16686,"addr":":16686"}
{"level":"info","ts":1599184515.372485,"caller":"app/server.go:129","msg":"Starting HTTP server","port":16686,"addr":":16686"}
root@automationmaster1:/home/ubuntu/jaeger/git/jaeger-operator/deploy/examples#


root@automationmaster1:/home/ubuntu/jaeger# kubectl describe jaegers simplest
Name:         simplest
Namespace:    default
Labels:       <none>
Annotations:  API Version:  jaegertracing.io/v1
Kind:         Jaeger
Metadata:
  Creation Timestamp:  2020-09-03T23:51:27Z
  Generation:          1
  Managed Fields:
    API Version:  jaegertracing.io/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:kubectl.kubernetes.io/last-applied-configuration:
    Manager:         kubectl
    Operation:       Update
    Time:            2020-09-03T23:51:27Z
  Resource Version:  15969425
  Self Link:         /apis/jaegertracing.io/v1/namespaces/default/jaegers/simplest
  UID:               c0ffeccf-79f1-4744-9ac4-d6d1785e581e
Events:              <none>
root@automationmaster1:/home/ubuntu/jaeger#



root@automationmaster1:/home/ubuntu/jaeger# kubectl describe jaegers simplest -n observability
Name:         simplest
Namespace:    observability
Labels:       jaegertracing.io/operated-by=observability.jaeger-operator
Annotations:  API Version:  jaegertracing.io/v1
Kind:         Jaeger
Metadata:
  Creation Timestamp:  2020-09-04T00:22:52Z
  Generation:          3
  Managed Fields:
    API Version:  jaegertracing.io/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:labels:
          .:
          f:jaegertracing.io/operated-by:
      f:spec:
        .:
        f:agent:
          .:
          f:config:
          f:options:
          f:resources:
        f:allInOne:
          .:
          f:config:
          f:options:
          f:resources:
        f:collector:
          .:
          f:config:
          f:options:
          f:resources:
        f:ingester:
          .:
          f:config:
          f:options:
          f:resources:
        f:ingress:
          .:
          f:openshift:
          f:options:
          f:resources:
          f:security:
        f:query:
          .:
          f:options:
          f:resources:
        f:resources:
        f:sampling:
          .:
          f:options:
        f:storage:
          .:
          f:cassandraCreateSchema:
          f:dependencies:
            .:
            f:resources:
            f:schedule:
          f:elasticsearch:
            .:
            f:nodeCount:
            f:redundancyPolicy:
            f:resources:
              .:
              f:limits:
                .:
                f:memory:
              f:requests:
                .:
                f:cpu:
                f:memory:
            f:storage:
          f:esIndexCleaner:
            .:
            f:numberOfDays:
            f:resources:
            f:schedule:
          f:esRollover:
            .:
            f:resources:
            f:schedule:
          f:options:
          f:type:
        f:strategy:
        f:ui:
          .:
          f:options:
            .:
            f:menu:
      f:status:
        .:
        f:phase:
        f:version:
    Manager:      jaeger-operator
    Operation:    Update
    Time:         2020-09-04T00:22:52Z
    API Version:  jaegertracing.io/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:kubectl.kubernetes.io/last-applied-configuration:
    Manager:         kubectl
    Operation:       Update
    Time:            2020-09-04T00:22:52Z
  Resource Version:  15975898
  Self Link:         /apis/jaegertracing.io/v1/namespaces/observability/jaegers/simplest
  UID:               728d8613-3811-48bc-92e7-4c1ecff93020
Spec:
  Agent:
    Config:
    Options:
    Resources:
  All In One:
    Config:
    Options:
    Resources:
  Collector:
    Config:
    Options:
    Resources:
  Ingester:
    Config:
    Options:
    Resources:
  Ingress:
    Openshift:
    Options:
    Resources:
    Security:  none
  Query:
    Options:
    Resources:
  Resources:
  Sampling:
    Options:
  Storage:
    Cassandra Create Schema:
    Dependencies:
      Resources:
      Schedule:  55 23 * * *
    Elasticsearch:
      Node Count:         3
      Redundancy Policy:  SingleRedundancy
      Resources:
        Limits:
          Memory:  16Gi
        Requests:
          Cpu:     1
          Memory:  16Gi
      Storage:
    Es Index Cleaner:
      Number Of Days:  7
      Resources:
      Schedule:  55 23 * * *
    Es Rollover:
      Resources:
      Schedule:  0 0 * * *
    Options:
    Type:    memory
  Strategy:  allinone
  Ui:
    Options:
      Menu:
        Items:
          Label:  Documentation
          URL:    https://www.jaegertracing.io/docs/1.19
        Label:    About
Status:
  Phase:    Running
  Version:  1.19.2
Events:     <none>
root@automationmaster1:/home/ubuntu/jaeger#



root@automationmaster1:/home/ubuntu/jaeger# kubectl get ingress -n observability
NAME             CLASS    HOSTS   ADDRESS   PORTS   AGE
simplest-query   <none>   *                 80      2m52s
root@automationmaster1:/home/ubuntu/jaeger#

Ingress is not showing IP address


helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install my-ingress-nginx  ingress-nginx/ingress-nginx -n observability

root@automationmaster1:/home/ubuntu/jaeger/git/jaeger-operator/deploy/examples# helm install my-ingress-nginx  ingress-nginx/ingress-nginx -n observability
NAME: my-ingress-nginx
LAST DEPLOYED: Fri Sep  4 01:11:00 2020
NAMESPACE: observability
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace observability get services -o wide -w my-ingress-nginx-controller'

An example Ingress that makes use of the controller:

  apiVersion: networking.k8s.io/v1beta1
  kind: Ingress
  metadata:
    annotations:
      kubernetes.io/ingress.class: nginx
    name: example
    namespace: foo
  spec:
    rules:
      - host: www.example.com
        http:
          paths:
            - backend:
                serviceName: exampleService
                servicePort: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
        - hosts:
            - www.example.com
          secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
root@automationmaster1:/home/ubuntu/jaeger/git/jaeger-operator/deploy/examples#


POD_NAME=$(kubectl get pods -l app.kubernetes.io/name=ingress-nginx -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD_NAME -- /nginx-ingress-controller --version

root@automationmaster1:/home/ubuntu/jaeger/git/jaeger-operator/deploy/examples# kubectl exec -it $POD_NAME -- /nginx-ingress-controller --version
-------------------------------------------------------------------------------
NGINX Ingress controller
  Release:       v0.35.0
  Build:         54ad65e32bcab32791ab18531a838d1c0f0811ef
  Repository:    https://github.com/kubernetes/ingress-nginx
  nginx version: nginx/1.19.2

-------------------------------------------------------------------------------

Expose simplest-query on NodePort
root@automationmaster1:/home/ubuntu/jaeger/git/jaeger-operator/deploy/examples# kubectl --namespace observability get services
NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                                  AGE
jaeger-operator-metrics       ClusterIP   10.101.184.129   <none>        8383/TCP,8686/TCP                        129m
simplest-agent                ClusterIP   None             <none>        5775/UDP,5778/TCP,6831/UDP,6832/UDP      61m
simplest-collector            ClusterIP   10.96.72.180     <none>        9411/TCP,14250/TCP,14267/TCP,14268/TCP   61m
simplest-collector-headless   ClusterIP   None             <none>        9411/TCP,14250/TCP,14267/TCP,14268/TCP   61m
simplest-query                NodePort    10.104.189.81    <none>        16686:30231/TCP                          61m
root@automationmaster1:/home/ubuntu/jaeger/git/jaeger-operator/deploy/examples#


root@automationmaster1:/home/ubuntu/jaeger/git/jaeger-operator/deploy/examples# kubectl apply -f hotrodweb.yaml
service/hotrodweb-svc unchanged
error: unable to recognize "hotrodweb.yaml": no matches for kind "Deployment" in version "extensions/v1beta1"
root@automationmaster1:/home/ubuntu/jaeger/git/jaeger-operator/deploy/examples#

root@automationmaster1:/home/ubuntu/jaeger/git/jaeger-operator/deploy/examples# kubectl apply -f hotrodweb.yaml
deployment.apps/hotrodweb created
service/hotrodweb-svc unchanged
root@automationmaster1:/home/ubuntu/jaeger/git/jaeger-operator/deploy/examples#

root@automationmaster1:/home/ubuntu/jaeger/git/jaeger-operator/deploy/examples# telnet 10.96.72.180 14268
Trying 10.96.72.180...
Connected to 10.96.72.180.
Escape character is '^]'.

telnet jaeger-collector-headless.observability.svc.cluster.local 14250
telnet simplest-jaeger-collector-headless.observability.svc.cluster.local 14250

[1:14 PM] Lavanya Subbarayalu
    
https://medium.com/opentracing/take-opentracing-for-a-hotrod-ride-f6e3141f7941
Take OpenTracing for a HotROD rideUpdate (21 March 2019): my book Mastering Distributed Tracing has a newer version of this tutorial.medium.com







---------------------------------------------------------------------------------------------------------------------------------------------------
































































