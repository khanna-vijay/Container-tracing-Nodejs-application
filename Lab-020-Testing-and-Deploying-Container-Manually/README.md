# Lab # 2 : Testing and Deploying the Containerized application manually
In this Lab, we will deploy and test the application manually step by step. This will help to figure out any issues. 

* **Clone Repo**
```
git clone https://github.com/vijay-khanna/container-tracing-app
cd ~/environment/container-tracing-app/
```


* **Enable SSM Permission to the Container Nodes**
The Application Secrets will be stored in Parameter Store, which will be securely accessed by the Node Application. 

>Open one of the Worker Instance in EC2 Console, note the "IAM role" assigned to instance, Open the Role in IAM. 
It will look something like "eksctl-EKS-Cluster-Istio-vk-nodeg-NodeInstanceRole-abcd1234", based on EKS Cluster name selected earlier. </br>

> Attach the below policies (These are for Test purpose only. For Production, give more restrictive access)</br>
>#**AmazonEC2RoleforSSM**</br>
>#**AmazonSSMFullAccess**</br>


Create Application Token on the below websites, and store the Token in SSM Store</br>
>https://darksky.net/dev</br>
>https://account.mapbox.com/</br>

Test the aws cli commands from Cloud9 Console

```
read -p "Enter the DarkSkyAPISecret : " DarkSkyAPISecret ; echo "DarkSkyAPISecret :  "$DarkSkyAPISecret

aws ssm put-parameter --name "/Params/keys/DarkSkyAPISecret" --value $DarkSkyAPISecret --type String --overwrite

read -p "Enter the MapBoxAccessToken : " MapBoxAccessToken ; echo "MapBoxAccessToken :  "$MapBoxAccessToken

aws ssm put-parameter --name "/Params/keys/MapBoxAccessToken" --value $MapBoxAccessToken --type String --overwrite

//To Verify the Successfull entry to SSM Store
aws ssm get-parameters --names "/Params/keys/DarkSkyAPISecret"
aws ssm get-parameters --names "/Params/keys/MapBoxAccessToken"
```
</br>
Optionally Test the above commands from the Worker Nodes. Use SSH Key created earlier.

</br>

* **Deploy the Service, so it is ready by the time Deployment finishes**
```
kubectl apply -f ~/environment/container-tracing-app/front-end/service-front-end.yaml 
kubectl apply -f ~/environment/container-tracing-app/backend-pi-array/service-back-end-pi-array.yaml 
```
</br>

* **Update the Load Balancer for front end Service in client js file, and Optionally a Route53 Entry**
```
//update the value of field : frontEndDNSURLandPort with the LB-URL or DNS A-Record. If it is a Route53 Entry, then create 'A' record and point to LB.

nano ~/environment/container-tracing-app/front-end/public/js/app-client-script.js 
```

* **creating Container from Dockerfile, and saving to ECR Repo in own account**

>#**FrontEnd Service**</br>
```
cd ~/environment/container-tracing-app/front-end/       
aws ecr get-login --region us-east-1 --no-include-email  
//Copy-Paste the Console output of above command output into the console to login. Starting with "docker login.... Ending with amazonaws.com". You must get "Login Succeeded" message to proceed.

//Below command will create ECR Repository
frontEndRepoECRURI=$(aws ecr create-repository --repository-name ${EKS_CLUSTER_NAME,,}_front_end | jq -r  '.repository.repositoryUri') 
echo $frontEndRepoECRURI  

//The below command will create Container image from DockerFile. This takes 5-7 minutes. Red text message for gpg key is normal, and not errors. 

docker build -t front-end:v1 .      
docker images  | grep front-end    
frontEndImageId=$(docker images front-end:v1 | grep front-end | awk '{print $3}') ; echo $frontEndImageId   

docker tag $frontEndImageId $frontEndRepoECRURI 
docker push $frontEndRepoECRURI  
```
</br>

>#**Backend Service**</br>
```
cd ~/environment/container-tracing-app/backend-pi-array/       
 
//Below command will create ECR Repository
backEndRepoECRURI=$(aws ecr create-repository --repository-name ${EKS_CLUSTER_NAME,,}_back_end | jq -r  '.repository.repositoryUri') 
echo $backEndRepoECRURI  

echo "export backEndRepoECRURI=${backEndRepoECRURI}" >> ~/.bash_profile
echo "export frontEndRepoECRURI=${frontEndRepoECRURI}" >> ~/.bash_profile

//The below command will create Container image from DockerFile. This takes 5-7 minutes. Red text message for gpg key is normal, and not errors. 

docker build -t back-end:v1 .
docker images  | grep back-end    
backEndImageId=$(docker images back-end:v1 | grep back-end | awk '{print $3}') ; echo $backEndImageId   
docker tag $backEndImageId $backEndRepoECRURI  
docker push $backEndRepoECRURI 
```
</br>

* **Updating deployment files with ECR Link to Container Images**
```
//replacing the IMAGE_URL with appropriate ECR Repo Locations for Deployment.Yaml Files
cp ~/environment/container-tracing-app/front-end/deployment-front-end.yaml /tmp/deployment-front-end.yaml

sed -i "s|IMAGE_URL|$frontEndRepoECRURI|g" /tmp/deployment-front-end.yaml
cat /tmp/deployment-front-end.yaml

cp ~/environment/container-tracing-app/backend-pi-array/deployment-back-end-pi-array.yaml  /tmp/deployment-back-end-pi-array.yaml

sed -i "s|IMAGE_URL|$backEndRepoECRURI|g" /tmp/deployment-back-end-pi-array.yaml
cat /tmp/deployment-back-end-pi-array.yaml
```
</br>

* **To Deploy Containers and Service using kubectl**

```
kubectl apply -f /tmp/deployment-front-end.yaml
kubectl apply -f /tmp/deployment-back-end-pi-array.yaml



kubectl get svc,deploy,pods
```

</br>



* **check the frontEndDNSURLandPort URL in Browser Window, to check http access**


