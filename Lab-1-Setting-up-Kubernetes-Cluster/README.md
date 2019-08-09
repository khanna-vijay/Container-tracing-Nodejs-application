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
>Type of trusted entity : "AWS Service" 
>Choose the service that will use this role : EC2, 'Click Next:permissions' <br/>
>Select from existing policies: "AdministratorAccess, 'Next:Tags'  <br/>
>Add-tags (optional) <br/>
>Role Name: 'Admin-Role_for_Cloud9_Instance' <br/>




* **Workshop Setup:**
### Launch Kubernetes Cluster:

