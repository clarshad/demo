## Kong Gateway with OPA

This implements a Kong Gateway with external authorization by OPA on a Kubernetes cluster environment.

#### Prerequisites

- Kubernetes cluster with kubectl access to it. We recommend creating a new cluster using minikube
- Enterprise licence for Kong Gateway

#### Installation

- Git clone this repository and follow below steps to install Kong Gateway with OPA plugin

1. Create kong namespace
```
kubectl create namespace kong
```

2. Create secret for Kong Enterprise license. Make sure license file does not have any extensions like .json,.txt,etc
```
kubectl create secret generic kong-enterprise-license -n kong --from-file=./license
```

3. Create secret with superuser password for enterprise Kong
```
kubectl create secret generic kong-enterprise-superuser-password -n kong --from-literal=password=DummyPass@1234
```

4. Create secret with GUI and portal session configuration
```
kubectl create secret generic kong-session-config -n kong --from-file=admin_gui_session_conf --from-file=portal_session_conf
```

5. Create configmap with opa plugin code
```
kubectl create configmap kong-plugin-opa --from-file=opa -n kong
```

6. Add and update helm repo which consist of charts for Kong Gateway deployment
```
helm repo add kong https://charts.konghq.com
helm repo update
```

7. Deploy Kong by helm install
```
helm install my-kong kong/kong -n kong --values ./values.yaml
```

#### Setup

Follow below steps to setup Kong Gateway authorization with OPA

1. Deploy opa pod in kong namespace and create a NodePort service for it
```
kubectl apply -f opa.yaml -n kong
kubectl expose pod opa -n kong --port=8181 --type=NodePort
```

2. Configure custom KongPlugin resource for OPA
```
kubectl apply -f kong-plugin.yaml
```

3. Deploy a dummy application - echo server
```
kubectl apply -f dummy-application.yaml
```

4. Create an Ingress/Route for echo server with OPA plugin annotation
```
kubectl apply -f ingress.yaml
```

#### Testing

Set below environment variables to access OPA and Kong Proxy
```
export HOST=$(kubectl get nodes --namespace kong -o jsonpath='{.items[0].status.addresses[0].address}')
export PROXY_PORT=$(kubectl get svc --namespace kong my-kong-kong-proxy -o jsonpath='{.spec.ports[0].nodePort}')
export OPA_PORT=$(kubectl get svc --namespace kong opa -o jsonpath='{.spec.ports[0].nodePort}')
```

Apply OPA policy, written in rego to OPA and server
```
curl -XPUT http://$HOST:$OPA_PORT/v1/policies/example --data-binary @example.rego
```

Make request to Kong Gateway - proxy service at path `/foo` as set up ingress resourse (step 4 in setup section)
```
curl -X GET http://$HOST:$PROXY_PORT/foo -v
```

You will get 200 HTTP response code with headers, pod and request information as response body. 

Edit the policy `example.rego`, change `input.path == "/foo"` to `input.path == "/bar"` in `allow` rule, this will force `allow` rule to evaluate to `false`. 

Apply OPA policy again and make same request to kong proxy, you will get 403 HTTP response code with `{"message":"Access Forbidden"}` as response body from gateway. This ensures that authorization is done by OPA and it has rejected the request. 
