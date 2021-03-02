# kubernetes-k8s-lab15
Installing Weave Scope on Kubernetes

Weave Scope lets you monitor and control your containerized microservices applications. By providing a visual map of your Docker Containers, you can see the dependencies and communication links between them.

Scope automatically detects processes, containers, hosts. No kernel modules, no agents, no special libraries, no coding.

This scenario explains how to map and visualise connections your Docker containers running on a Kubernetes cluster.

######

Step 2 - Launch Weave Scope

Weave Scope is deployed as a Pod running on the Kubernetes cluster. From here it can visualise the containers running and traffic between different containers.

You can view the configuration used to deploy Weave via 

curl -L https://cloud.weave.works/launch/k8s/weavescope.yaml

The configuration starts a Replication Controller and Service. It also deploys a DaemonSet. A DaemonSet automatically deploys Pods onto new hosts are deployed into the cluster. The result is you can visualise your entire network without as it changes without having to manage the Scope deployment or deploying it onto these new hosts.

Task

To deploy Scope use the yaml with kubectl. It currently requires to set the validate to false; this will be fixed in future versions of Kubernetes.


$ kubectl create -f 'https://cloud.weave.works/launch/k8s/weavescope.yaml'
namespace/weave created
serviceaccount/weave-scope created
clusterrole.rbac.authorization.k8s.io/weave-scope created
clusterrolebinding.rbac.authorization.k8s.io/weave-scope created
deployment.apps/weave-scope-app created
service/weave-scope-app created
deployment.apps/weave-scope-cluster-agent created
daemonset.apps/weave-scope-agent created
automationmgr@master1:~/workbench/kubenetesbench/kubernetes-k8s-lab15$ 


Wait for it to be deployed by checking the status of the pods using 

$ kubectl get pods -n weave -o wide
NAME                                         READY   STATUS              RESTARTS   AGE     IP             NODE      NOMINATED NODE   READINESS GATES
weave-scope-agent-dqh9j                      1/1     Running             0          2m35s   192.168.1.86   node1     <none>           <none>
weave-scope-agent-v459j                      1/1     Running             0          2m35s   192.168.1.82   node2     <none>           <none>
weave-scope-agent-v7fmk                      1/1     Running             0          2m35s   192.168.1.85   master1   <none>           <none>
weave-scope-app-545ddf96b4-lnfmv             0/1     ContainerCreating   0          2m34s   <none>         node1     <none>           <none>
weave-scope-cluster-agent-74c596c6b7-q7jbb   0/1     ContainerCreating   0          2m35s   <none>         node1     <none>           <none>


By default, once deployed it will only be accessible from inside the cluster. You need to create a Service which exposes the port. In the command below we also expose the Service to the outside world via the external-ip parameter. Exposing the service to on a public IP is not recommended. Instead, it should require a VPN connection to access.

pod=$(kubectl get pod -n weave --selector=name=weave-scope-app -o jsonpath={.items..metadata.name})
kubectl expose pod $pod -n weave --external-ip="172.17.0.42" --port=4040 --target-port=4040

######