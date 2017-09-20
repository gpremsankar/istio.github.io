---
title: Intelligent Routing
overview: This sample deploys the Bookinfo application in a Kubernetes environment and demonstrates how to use various traffic management capabilities of Istio service mesh.

order: 20

layout: docs
type: markdown
draft: true
---
{% include home.html %}

This sample deploys the Bookinfo application in a Kubernetes environment and demonstrates how to use various traffic management capabilities of Istio service mesh.

## Before you begin
* If you use GKE, please ensure your cluster has at least 4 standard GKE nodes.

* Setup Istio by following the instructions in the
[Installation guide]({{home}}/docs/tasks/installing-istio.html).

## Overview

Plaeholder.

## Application Setup

1. Change directory to the root of the Istio installation directory.

1. Bring up the application containers:

   ```bash
   kubectl apply -f <(istioctl kube-inject -f samples/apps/bookinfo/bookinfo.yaml)
   ```

   The above command launches four microservices and creates the gateway
   ingress resource as illustrated in the diagram below.
   The reviews microservice has 3 versions: v1, v2, and v3.

   > Note that in a realistic deployment, new versions of a microservice are deployed
   over time instead of deploying all versions simultaneously.

   Notice that the `istioctl kube-inject` command is used to modify the `bookinfo.yaml`
   file before creating the deployments. This injects Envoy into Kubernetes resources
   as documented [here]({{home}}/docs/reference/commands/istioctl.html#istioctl-kube-inject).

1. Confirm all services and pods are correctly defined and running:

   ```bash
   kubectl get services
   ```

   which produces the following output:
   
   ```bash
   NAME                       CLUSTER-IP   EXTERNAL-IP   PORT(S)              AGE
   details                    10.0.0.31    <none>        9080/TCP             6m
   istio-ingress              10.0.0.122   <pending>     80:31565/TCP         8m
   istio-pilot                10.0.0.189   <none>        8080/TCP             8m
   istio-mixer                10.0.0.132   <none>        9091/TCP,42422/TCP   8m
   kubernetes                 10.0.0.1     <none>        443/TCP              14d
   productpage                10.0.0.120   <none>        9080/TCP             6m
   ratings                    10.0.0.15    <none>        9080/TCP             6m
   reviews                    10.0.0.170   <none>        9080/TCP             6m
   ```

   and

   ```bash
   kubectl get pods
   ```
   
   which produces
   
   ```bash
   NAME                                        READY     STATUS    RESTARTS   AGE
   details-v1-1520924117-48z17                 2/2       Running   0          6m
   istio-ingress-3181829929-xrrk5              1/1       Running   0          8m
   istio-pilot-175173354-d6jm7                 2/2       Running   0          8m
   istio-mixer-3883863574-jt09j                2/2       Running   0          8m
   productpage-v1-560495357-jk1lz              2/2       Running   0          6m
   ratings-v1-734492171-rnr5l                  2/2       Running   0          6m
   reviews-v1-874083890-f0qf0                  2/2       Running   0          6m
   reviews-v2-1343845940-b34q5                 2/2       Running   0          6m
   reviews-v3-1813607990-8ch52                 2/2       Running   0          6m
   ```

1. Determine the gateway ingress URL:

   ```bash
   kubectl get ingress -o wide
   ```
   
   ```bash
   NAME      HOSTS     ADDRESS                 PORTS     AGE
   gateway   *         130.211.10.121          80        1d
   ```

   If your Kubernetes cluster is running in an environment that supports external load balancers,
   and the Istio ingress service was able to obtain an External IP, the ingress resource ADDRESS will be equal to the
   ingress service external IP.

   ```bash
   export GATEWAY_URL=130.211.10.121:80
   ```
   
   > Sometimes when the service is unable to obtain an external IP, the ingress ADDRESS may display a list
   > of NodePort addresses. In this case, you can use any of the addresses, along with the NodePort, to access the ingress. 
   > If, however, the cluster has a firewall, you will also need to create a firewall rule to allow TCP traffic to the NodePort.
   > In GKE, for instance, you can create a firewall rule using the following command:
   > ```bash
   > gcloud compute firewall-rules create allow-book --allow tcp:$(kubectl get svc istio-ingress -o jsonpath='{.spec.ports[0].nodePort}')
   > ```

   If your deployment environment does not support external load balancers (e.g., minikube), the ADDRESS field will be empty.
   In this case you can use the service NodePort instead:
   
   ```bash
   export GATEWAY_URL=$(kubectl get po -l istio=ingress -o 'jsonpath={.items[0].status.hostIP}'):$(kubectl get svc istio-ingress -o 'jsonpath={.spec.ports[0].nodePort}')
   ```

1. Confirm that the BookInfo application is running with the following `curl` command:

   ```bash
   curl -o /dev/null -s -w "%{http_code}\n" http://${GATEWAY_URL}/productpage
   ```
   ```bash
   200
   ```
   
## Tasks

1. [Request routing]({{home}}/docs/tasks/request-routing.html).

1. [Fault injection]({{home}}/docs/tasks/fault-injection.html).

## Cleanup

When you're finished experimenting with the BookInfo sample, you can
uninstall it as follows in a Kubernetes environment:

1. Delete the routing rules and terminate the application pods

   ```bash
   samples/apps/bookinfo/cleanup.sh
   ```

1. Confirm shutdown

   ```bash
   istioctl get route-rules   #-- there should be no more routing rules
   kubectl get pods           #-- the BookInfo pods should be deleted
   ```

If you are using the Docker Compose version of the demo, run the following
command to clean up:

  ```bash
  docker-compose -f samples/apps/bookinfo/consul/docker-compose.yaml down
  ```
