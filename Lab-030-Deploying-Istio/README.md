# Lab # 3 : Deploying and Testing Istio 

In this Lab, we will deploy helm and Istio

* **Deploy Helm**
</br>
first install the command line tools
```
cd ~/environment

curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh

chmod +x get_helm.sh

./get_helm.sh

```
</br>
**create a new service account manifest**
```
cat <<EoF > ~/environment/rbac.yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
EoF
```
</br>
**Apply the Config**
```
kubectl apply -f ~/environment/rbac.yaml


//install tiller using the helm

helm init --service-account tiller

helm repo update
```

</br>
* **Deploy Istio and Ecosystem Services**

```
cd ~/environment

curl -L https://git.io/getLatestIstio | sh -

// version can be different as istio gets upgraded
cd istio-*

sudo mv -v bin/istioctl /usr/local/bin/

kubectl create namespace istio-system

helm template install/kubernetes/helm/istio-init --name istio-init --namespace istio-system | kubectl apply -f -

//Verify that all 23 Istio CRDs were committed to the Kubernetes api-server
kubectl get crds | grep 'istio.io\|certmanager.k8s.io' | wc -l


helm template install/kubernetes/helm/istio --name istio --namespace istio-system \
    --values install/kubernetes/helm/istio/values-istio-demo.yaml | kubectl apply -f -


//Verifying the installation

kubectl get pods -n istio-system
kubectl get svc -n istio-system

//Injecting Pods inside containers

kubectl get pods
istioctl kube-inject -f /tmp/deployment-back-end-pi-array.yaml | kubectl apply -f -
kubectl get pods

//wait till the backend pods initialize, then inject the front-end deployment
istioctl kube-inject -f /tmp/deployment-front-end.yaml | kubectl apply -f -
kubectl get pods
```



* **Checking Metrics and Tracing**
```




// kiali. with default demo, username, password = admin. check in Preview of Cloud9

kubectl -n istio-system get svc kiali

kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=kiali -o jsonpath='{.items[0].metadata.name}') 8080:20001


```


* **Enable SSM Permission to the Container Nodes**
