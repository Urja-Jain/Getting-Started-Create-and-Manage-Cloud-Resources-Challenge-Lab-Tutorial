# Getting-Started-Create-and-Manage-Cloud-Resources-Challenge-Lab-Tutorial
This medium article focusses on the detailed walk through of the steps I took to solve the challenge lab of the Create and Manage Cloud Resources Skill Badge on the Google Cloud Platform.


### Detailed Tutorial of Task — 1
In task 1, it requires the user to create a Jumphost instance with the following parameters:
Naming of the instance should be nucleus-jumphost
The machine type should be f1-micro.
Using the default image type (Debian Linux).
Go to navigation menu > Compute Engine > VM Instance.
Select the above parameters and click create.
#### Wait for a second and then check your progress in the lab. You should see a green mark. Now lets go to task-2.

### Detailed Tutorial of Task — 2
In task 2, the lab requires the user to create a Kubernetes Service Cluster with the following parameters:
Creating the cluster in the us-east1 region.
Using the Docker container hello-app (`gcr.io/google-samples/hello-app:2.0`) as a place holder.
Exposing the app on port 8080.

#### Open the Cloud Shell and wait for it to be configured. Then run the following set of commands to create a kubernetes cluster:


     gcloud config set compute/zone us-east1-b

     gcloud container clusters create nucleus-jumphost-webserver1

     gcloud container clusters get-credentials nucleus-jumphost-webserver1

     kubectl create deployment hello-app --image=gcr.io/google-samples/hello-app:2.0

     kubectl expose deployment hello-app --type=LoadBalancer --port 8080

     kubectl get service


### Detailed Tutorial of Task — 3
Step 3 requires to setup an HTTP Load Balancer with a managed instance group of two nginx web servers. We need to do a set of 8 small tasks in order to score full in this task. 
##### The following set of configurations is given in advance:

cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
#### Press enter after this. After that, follow the below steps:

#### 1- Creating an instance template:
    gcloud compute instance-templates create nginx-template \
    --metadata-from-file startup-script=startup.sh

#### 2-Creating a target pool:
gcloud compute target-pools create nginx-pool

#### This command will ask you the location where you want to create the target pool. If it’s not us-east1, type “n”.
#### Select the number corresponding to us-east1 in the list.

#### 3-  Creating a managed instance group:
gcloud compute instance-groups managed create nginx-group \
--base-instance-name nginx \
--size 2 \
--template nginx-template \
--target-pool nginx-pool

gcloud compute instances list

#### 4-Creating a firewall rule to allow traffic (80/tcp):

gcloud compute firewall-rules create www-firewall --allow tcp:80

gcloud compute forwarding-rules create nginx-lb \
--region us-east1 \
--ports=80 \
--target-pool nginx-pool

gcloud compute forwarding-rules list

#### 5-Creating a health check:
gcloud compute http-health-checks create http-basic-check

gcloud compute instance-groups managed \
set-named-ports nginx-group \
--named-ports http:80

#### 6- Creating a backend service and attach the manged instance group:

gcloud compute backend-services create nginx-backend \
--protocol HTTP --http-health-checks http-basic-check --global

gcloud compute backend-services add-backend nginx-backend \
--instance-group nginx-group \
--instance-group-zone us-east1-b \
--global

#### 7- Creating a URL map and target HTTP proxy to route requests to your URL map:

gcloud compute url-maps create web-map \
--default-service nginx-backend

gcloud compute target-http-proxies create http-lb-proxy \
--url-map web-map

#### 8- Creating a forwarding rule:

gcloud compute forwarding-rules create http-content-rule \
--global \
--target-http-proxy http-lb-proxy \
--ports 80

gcloud compute forwarding-rules list


#### After this gcloud compute forwarding-rules list and then open the ip addresses listed in the output of the above command 
#### in browser. There will be three values in the list. Ignore the first one. Open the ip adreesses of the rest two in your browser 
For example : 

              http://<IP_Address>:8080   ----> for second one

              http://<IP_ADDRESS>        -----> for the third one



