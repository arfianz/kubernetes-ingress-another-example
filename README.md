# Kubernetes Ingress Another Example App

What is an Ingress? It is a door, a way into your application.

![Door](./background.jpeg?raw=true)

## What's Ingress?

An ingress is a set of rules that allows inbound connections to reach the kubernetes cluster services.

Typically, the kubernetes cluster is firewalled from the internet. It has edge routers enforcing the firewall. Kubernetes resources like services, pods, have IPs only routable by the cluster network and are not (directly) accessible outside the cluster. All traffic that ends up at an edge router is either dropped or forwarded elsewhere. So, submitting an ingress to the cluster defines a set of rules for routing external traffic to the kubernetes endpoints.

As you might have guessed, the rule is:

- all requests to myminikube.info/ should be routed to the service in the cluster named echoserver.
- requests mapping to cheeses.all/stilton should be routed to the stilton-cheese service.
- and finally, requests mapping to cheeses.all/cheddar should be routed to the cheddar-cheese service.

Of course, there’s more to it; like the backend tag which implies that unmatched requests should be routed to the default-http-backend service and there’s also the familiar kubernetes tags; for example the apiVersion tag which clearly marks ingress as a beta feature.

Note the annotation:

```bash
ingress.kubernetes.io/rewrite-target: /
```

## The Ingress Controller

In order for the Ingress resource to work, the cluster must have an Ingress controller running. When a user requests an ingress by POSTing an Ingress resource (such as the one above) to the API server, the Ingress controller is responsible for fulfilling the Ingress, usually with a loadbalancer. Though it may also configure the edge routers or additional frontends to help handle the traffic.

As such without an Ingress controller to satisfy the ingress, merely creating the ingress resource will have no effect.

You can write your own controller but you need not to. There are readily available third-party ingress controllers like the Nginx, Traefik, HAproxy controllers which you could easily leverage. I will be using the Nginx controller for this demo but feel free to try out any other.

### Deploying Application

Now that our cluster is running we can go ahead and start deploying stuff.

1. Deploy our pods(containers), using deployments. A deployment, pretty much just manages the state of our pods,

```bash
kubectl apply -f meow-echo.yaml
kubectl get deploy | grep meow-echo
kubectl get pods | grep meow-echo
```
The image is the location of the container. When deploying your own application, you would select your image. gcr.io/kubernetes-e2e-test-images/echoserver:2.1 just responds with information about the request.

Also note that containerPort: 8080 is the port that the application is running on.

2. Expose our pods using Services

```bash
kubectl apply -f meow-service.yaml
kubectl get svc | grep meow-svc
```

Note the targetPort: 8080 is the port we target when we access the service. port: 80 is the port used to access the service.

3. Setup the Ingress Rules.

```bash
kubectl apply -f meow-ingress.yaml
kubectl get ing | grep meow
```

This allows us to access the service meow-svc via the /meow path. Since we didn’t specify a host, then we can access it using the clusterIP.

Note the nginx.ingress.kubernetes.io/ssl-redirect annotation. It is used since we are not specifying a host. When no host is specified, then the default-server is hit, which is configured with a self-signed certificate, and redirects http to https.

## Accessing your application

Now the Ingress rules are configured, we can test everything out. Now let’s send some requests.

```bash
minikube ip
curl 192.168.99.100/meow

## Basic Debugging

In this section, we will explore a few basics of debugging. The following show how to gather basic information, which can be useful to determine what’s going on.

### Describe the Pods

Prints a detailed description of the selected pods, which includes events.

```bash
kubectl get pods -n kube-system | grep nginx-ingress-controller
kubectl describe pods -n kube-system nginx-ingress-controller-xxxxxx-yyyy
```

### View the Logs

Prints the logs for the nginx-ingress-controller.

```bash
kubectl logs -n kube-system nginx-ingress-controller-xxxxxx-yyyy
```

### View the Nginx Conf

Displays how nginx configures the application routing rules.

```bash
kubectl exec -it -n kube-system nginx-ingress-controller-xxxxxx-yyyy cat /etc/nginx/nginx.conf
```

## LICENSE

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

