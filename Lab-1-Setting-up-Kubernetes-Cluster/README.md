# Lab # 1 : Installing the Kubernetes CLuster
We will be using AWS EKS for this demo. Login with an Administrator equivalent user to AWS console.

### Deploy Cloud9 IDE:
This lab documentation is made for N.Virginia region (us-east-1). Please make note of this, and change accordingly for your deployment.
Login to the AWS EC2 Console, go to Cloud9 Services. Create a new environment e.g. "Tracing Containerized Nodejs application" <br/>
>Select Environment type : "Create a new instance for environment (EC2)<br/>
>Instance type : t2.micro (1 GiB RAM + 1 vCPU)  <br/>
>Platform : Amazon Linux <br/>
>Cost-saving setting: after 30 minutes <br/>
>Network settings : Select an existing vpc and subnet, or create a new one <br/>
>Review, click Create <br/>

* **IAM AdministratorAccess Role for Cloud9 Instance :**
>Go to IAM Service, create a role <br/>
>Type of trusted entity : **AWS Service** <br/>
>Choose the service that will use this role : **EC2**, 'Click Next:permissions' <br/>
>Select from existing policies: **AdministratorAccess**, 'Next:Tags'  <br/>
>Add-tags (optional) <br/>
>Role Name: **Admin-Role_for_Cloud9_Instance** <br/>
>Open EC2 Service console, select the Cloud9 Instance <br/>
> Actions => Instance Settings => Attach/Replace IAM Role => Select **Admin-Role_for_Cloud9_Instance** => Apply<br/>

> In Cloud9 console => Preferences => AWS Settings => Disable "AWS Managed Temporary Credentials <br/>

> <br/>
> <br/>
> <br/>



* **Install kubectl and ancilliary tools:**
Inside the cloud9 console, 
```
sudo curl --silent --location -o /usr/local/bin/kubectl https://storage.googleapis.com/kubernetes-release/release/v1.13.7/bin/linux/amd64/kubectl

sudo chmod +x /usr/local/bin/kubectl

sudo yum -y install jq gettext

```
Verify the binaries are in path.
```
for command in kubectl jq envsubst
  do
    which $command &>/dev/null && echo "$command in path" || echo "$command NOT FOUND"
  done

```
Remove temporary credendials, and setup environment variables.

```
rm -vf ${HOME}/.aws/credentials

export ACCOUNT_ID=$(aws sts get-caller-identity --output text --query Account)
export AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')

echo "export ACCOUNT_ID=${ACCOUNT_ID}" >> ~/.bash_profile
echo "export AWS_REGION=${AWS_REGION}" >> ~/.bash_profile
aws configure set default.region ${AWS_REGION}
aws configure get default.region

```
Validate the IAM role. the Output Arn should contain the IAM role created and attached to instance, and the Instance ID
e.g.  "Arn": "arn:aws:sts::1122334455:assumed-role/Admin-Role_for_Cloud9_Instance/i-0abcd1122334455<br/>
```
aws sts get-caller-identity
```


* **Workshop Setup:**
* **Workshop Setup:**
### Launch Kubernetes Cluster:

