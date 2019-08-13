# Last Lab : Deleting all Resources created

> In this final Lab, we will delete all resources created during the demo.


</br>

* **Deleting Istio Components**

```
cd ~/environment/istio*
 
helm delete --purge mywebserver

kubectl delete -f istio-telemetry.yaml

kubectl delete -f samples/bookinfo/networking/virtual-service-all-v1.yaml

kubectl delete -f samples/bookinfo/networking/destination-rule-all.yaml

kubectl delete -f samples/bookinfo/networking/bookinfo-gateway.yaml

kubectl delete -f samples/bookinfo/platform/kube/bookinfo.yaml

kubectl delete -f install/kubernetes/istio-demo.yaml
for i in install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl delete -f $i; done

helm template install/kubernetes/helm/istio --name istio --namespace istio-system \
    --values install/kubernetes/helm/istio/values-istio-demo.yaml | kubectl delete -f -
kubectl delete namespace istio-system

kubectl delete -f install/kubernetes/helm/istio-init/files


helm delete --purge istio
helm delete --purge istio-init

```
</br>

* **Removing all Deployments and Services**
```
//Deleting the key front-end and back-end services and deployments.
kubectl delete -f /tmp/deployment-back-end-pi-array.yaml
kubectl delete -f /tmp/deployment-front-end.yaml

kubectl delete -f ~/environment/container-tracing-app/front-end/service-front-end.yaml 

kubectl delete -f ~/environment/container-tracing-app/backend-pi-array/service-back-end-pi-array.yaml 


//Below ones in case the additional demo apps have been deployed.
kubectl delete -f ~/environment/pod-with-node-affinity.yaml
kubectl delete -f ~/environment/redis-with-node-affinity.yaml
kubectl delete -f ~/environment/web-with-node-affinity.yaml

cd ~/environment/ecsdemo-frontend
kubectl delete -f kubernetes/service.yaml
kubectl delete -f kubernetes/deployment.yaml

cd ~/environment/ecsdemo-crystal
kubectl delete -f kubernetes/service.yaml
kubectl delete -f kubernetes/deployment.yaml

cd ~/environment/ecsdemo-nodejs
kubectl delete -f kubernetes/service.yaml
kubectl delete -f kubernetes/deployment.yaml

kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml



//check services and deployments
kubectl get svc,deploy

kubectl get ns

```

</br>

* **Deleting EKS Cluster and ECR Repositories**
```


eksctl delete cluster --name=eksworkshop-eksctl


```

</br>

* **Removing ad-hoc components on AWS**
```
// ssh Keys

// Load Balancers

//Route53 Entries

//Role for Worker Nodes


//SSM Paramter Store entries

```

</br>

* **Removing cloud-9 Environment**
```
    Go to your Cloud9 Environment
    Select the environment created and pick delete
```
