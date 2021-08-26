## Kong Gateway with OPA

#### Prerequisites

- Kubernetes cluster with kubectl access to it. We recommend creating a new cluster using minikube
- Enterprise licence for Kong Gateway

#### Setup

- Create kong namespace
`kubectl create namespace kong`

- Create secret for Kong Enterprise license. Make sure license file does not have extension like .json,.txt,etc
`kubectl create secret generic kong-enterprise-license -n kong --from-file=./license`

- Add and update helm repo which consist of charts for Kong Gateway deployment
`helm repo add kong https://charts.konghq.com`
`helm repo update`

  
