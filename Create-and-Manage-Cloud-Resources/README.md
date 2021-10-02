## `Create and Manage Cloud Resources: Challenge Lab`

### `Task 1: Create a project jumphost instance`

- Click the **Navigation Menu** > **Computer Engine** > **VM Instances**
- Select **Create Instance**
- Give **nucleus-jumphost** as instance name
- Select **f1-micro** as machine type
- Finally, Click **Create Instance** and Check the progress in Qwiklabs

<hr>

### `Task 2: Create a Kubernetes service cluster`

1) First, authorize your console

```bash
gcloud auth list
```

2) Check your current **Project ID** is added in the configuration
```bash
gcloud config list project
```

3) Set the default zone to us-east1-b
```bash
gcloud config set compute/zone us-east1-b
```

4) Create a container cluster and get the credentials 
```bash
gcloud container clusters create manoj-hello-app
gcloud container clusters get-credentials manoj-hello-app
```

5) Next create a deployment contanier and expose it to port `8080`
```bash
kubectl create deployment hello-app --image=gcr.io/google-samples/hello-app:2.0
kubectl expose deployment hello-app --type=LoadBalancer --port 8080
```

<hr>

### `Task 3: Set up an HTTP load balancer`

1) Create a startup.sh to use it as template for startup-script

```bash
cat << EOF > startup.sh
    #! /bin/bash
    apt-get update
    apt-get install -y nginx
    service nginx start
    sed -i -- 's/nginx/Google Cloud Platform -'"\$HOSTNAME"'/' 
    /var/www/html/index.nginx-debian.html
    EOF
```


2) Create a instance template

```bash
gcloud compute instance-templates create nginx-template --metadata-from-file startup-script=startup.sh
```

3) Create a pool for connecting other instances
```bash
gcloud compute target-pools create nginx-pool
```
##### Press **N** and select `us-east1`

4) Creating 2 same ngnix instances 

```bash
gcloud compute instance-groups managed create nginx-group --base-instance-name nginx --size 2 --template nginx-template --target-pool nginx-pool
```

5) Now 2 instances is created, you can list that using below command
```bash
gcloud compute instances list
```

6) Creating firewal rules to allow tcp with port number `80` with the `www-firewall` as the rule name
```
gcloud compute firewall-rules create www-firewall --allow tcp:80
```

7) Creating forwarding rules with `ngnix-pool` in port `80`
```bash
gcloud compute forwarding-rules create nginx-lb --region us-east1 --ports=80 --target-pool nginx-pool
```

8) After this commmand you see the forwarding rules

```bash
gcloud compute forwarding-rules list
```

9) Creating health check for Google Cloud

```bash
gcloud compute http-health-checks create http-basic-check
```

10) Setting name port as `ngnix-group`

```bash
gcloud compute instance-groups managed set-named-ports nginx-group --named-ports http:80
```

11) Creating and adding backend services

```bash
gcloud compute backend-services create nginx-backend --protocol HTTP --http-health-checks http-basic-check --global
```

```bash
gcloud compute backend-services add-backend nginx-backend --instance-group nginx-group --instance-group-zone us-east1-b --global
```

12) Mapping the urls with the `nginx-backend` service

```bash
gcloud compute url-maps create web-map --default-service nginx-backend
```

13) Setting target proxies for instances

```bash
gcloud compute target-http-proxies create http-lb-proxy --url-map web-map
```

14) Creating forwarding rule

```bash
gcloud compute forwarding-rules create http-content-rule --global --target-http-proxy http-lb-proxy --ports 80
```

15) Now, after creating you can verify by listing it

```bash
gcloud compute forwarding-rules list
```

**`Kindly note that, it takes around 10-20 minutes for passing this test case`**
