# Kubernetes Ingress Another Example App

What is an Ingress? It is a door, a way into your application.

![Door](./background.jpeg?raw=true)

## What's Ingress?

How does Ingress work in Kubernetes? In Kubernetes, Ingress works by configuring rules for accessing your applications.

There are 4 items which we will be looking at, Pods, Services, Ingress Resources, and the Ingress Controller. WARNING: I’m going to use alot of door metaphors.

- A Pod is a group of one or more containers, with shared storage/network, and a specification for how to run the containers. That definition is straight from Kubernetes. It’s the actual application, sitting behind the door.
- A Service identifies a set of pods that are running your application, and routes traffic to those pods. It does this using label selectors. It’s what’s behind the door, pointing at the application.
- The Ingress Resource is a collection of rules that allows incoming connections to reach your Services. It is the door.
- The Ingress Controller uses Ingress Resources to configure application access. In Ingress-Nginx, an Nginx configuration is built from the ingress resources, setting up the routing rules. It is the door person, which opens and closes the door to users.

## Tutorial

Now, lets get started with putting all the above into action!

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
```

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

