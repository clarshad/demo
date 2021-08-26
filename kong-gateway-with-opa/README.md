## Kong Gateway with OPA

This implements a Kong Gateway with external authorization by OPA on a Kubernetes cluster environment.

#### Prerequisites

- Kubernetes cluster with kubectl access to it. We recommend creating a new cluster using minikube
- Enterprise licence for Kong Gateway

#### Installation

- Git clone this repository and follow below steps to install Kong Gateway with OPA plugin

1. Create kong namespace
``
kubectl create namespace kong
``

2. Create secret for Kong Enterprise license. Make sure license file does not have extension like .json,.txt,etc
``
kubectl create secret generic kong-enterprise-license -n kong --from-file=./license
``

3. Create secret with superuser password for enterprise Kong
``
kubectl create secret generic kong-enterprise-superuser-password -n kong --from-literal=password=Digital@1234
``

4. Create secret with GUI and portal session configuration
``
kubectl create secret generic kong-session-config -n kong --from-file=admin_gui_session_conf --from-file=portal_session_conf
``

5. Create configmap with opa plugin code
``
kubectl create configmap kong-plugin-opa --from-file=opa -n kong
```

6. Add and update helm repo which consist of charts for Kong Gateway deployment
``
helm repo add kong https://charts.konghq.com
helm repo update
``

7. Deploy Kong by helm install
``
helm install my-kong kong/kong -n kong --values ./values.yaml
``

#### Setup

Follow below steps to setup Kong Gateway authorization with OPA

1. Deploy opa pod in kong namespace and create a NodePort service for it
``
kubectl apply -f opa.yaml -n kong
kubectl expose pod opa --port=8181 --type=NodePort
``

2. Configure custom KongPlugin resource for OPA
``
kubectl apply -f kong-plugin.yaml
``

3. Deploy a dummy application - echo server
``
kubectl apply -f dummy-application.yaml
``

4. Create an Ingress/Route for echo server with OPA plugin annotation
``
kubectl apply -f ingress.yaml
``

#### Testing



 
