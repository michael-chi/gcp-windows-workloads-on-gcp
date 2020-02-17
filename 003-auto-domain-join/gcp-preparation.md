##  Overview

We need to setup our GCP project in order to access domain resources. There are several options to host our domain join API

-   Cloud Function
    
    In order to be able access domain resource we need setup Serverless VPC access here.

-   GKE

    GKE is natively integrated with VPC

-   Compute Enginer with Container

    GCE is natively integrated with VPC

-   AppEngine Flexible Environment

-   Compute Engine

###    Deploy Domain Join API to Container hosted in Compute Engine instance

    Note that for simplicity, I am using HTTP instead of HTTPS endpoint. There are many options in GCP which natively supports HTTPS, such as Cloud Function, Cloud Run...etc. However, in order to access your AD, these services need to integrated with VPC via Serverless VPC Access or VPC integration which can add complixity.

    Hence in my case I deploy a container image to Compute Instance and running HTTP.

-   Clone sample domain join applications from [here](https://github.com/GoogleCloudPlatform/gce-automated-ad-join/tree/master/register-computer).

    Also created a [Dockerfile](./src//auto-domain-join/register-computer/Dockerfile) so that we can deploy it to servives such as GKE.

#####  (Optional)HTTPS Support.

-   To support HTTPS, I have below openssl to generate self-signed SSL for HTTPS where CN set to GCE's default internal FQDN (INSTANCE_NAME.c.PROJECT_ID.internal) as in the codes it requires HTTPS protocol.

```Dockerfile
### ...
RUN openssl req \
    -new \
    -newkey rsa:4096 \
    -days 365 \
    -nodes \
    -x509 \
    -subj "/C=TW/ST=Taiwan/L=Taipei/O=GCP/CN=domainjoinapi.c.PROJECT_ID.internal" \
    -keyout server.key \
    -out server.crt
```

-   Also in main.py, change to use SSL certificate when required

```python
    app.run('0.0.0.0', debug=True, port=443, ssl_context=('./server.crt', './server.key'))  
    # app.run(host='0.0.0.0', port=80)
```
-   Finally, update join.py, use HTTP instead of HTTPS

```python
$JoinInfo = (Invoke-RestMethod `
    -Headers @{"Authorization" = "Bearer $IdToken"} `
    -Method POST `
    -Uri "http://%domain%/register-computer")
```

#####  Deploy Container on GCE

-   Compile ksetpwd tool

    Switch to parent folder of register-computer folder where ksetpwd folder resides. Run below command to compile ksetpwd. This command reads Makefile and output compiled binary to register-computer\\ksetpwd\\bin

```shell
make
```

-   Run below command to build and push container image to GCR

```shell
docker build . -t gcr.io/$DOMAIN_PROJECT_ID/register-computer:latest
docker push gcr.io/$DOMAIN_PROJECT_ID/register-computer:latest
```
-   Now we are ready to deploy API server for newly provisioned instance to bootstrap itself. To execute below command, replace below parameters with proper values.

    - [PROJECT_ID] : replace with your top OU name
    - [PASSWORD] : domain join user password
    - [DOMAIN] : AD Domain name
    - [AD_USER] : domain join user name

```shell
export PASSWORD=[PASSWORD]
export INSTANCE_NAME=[INSTANCE_NAME]
export DOMAIN_PROJECT_ID=[PROJECT_ID]
export VPC_NAME=[VPC_NAME]

gcloud beta compute --project=$DOMAIN_PROJECT_ID instances create-with-container $INSTANCE_NAME --zone=asia-east1-b \
    --machine-type=n1-standard-1 --subnet=$VPC_NAME \
    --metadata=google-logging-enabled=true,google-monitoring-enabled=true \
    --scopes=https://www.googleapis.com/auth/cloud-platform \
    --tags=http-server,https-server --image=cos-stable-80-12739-68-0 \
    --image-project=cos-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard \
    --boot-disk-device-name=$INSTANCE_NAME \
    --container-image=gcr.io/$DOMAIN_PROJECT_ID/register-computer:20200217-001 \
    --container-restart-policy=always \
    --container-env=^,@^PROJECTS_DN=[PROJECT_OU_DN],@AD_USERNAME=[DOMAIN]\\[AD_USER],@AD_PASSWORD=$PASSWORD,@AD_DOMAIN=[DOMAIN]

##  Create firewall rules when required
# gcloud compute --project=$DOMAIN_PROJECT_ID firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server

# gcloud compute --project=$DOMAIN_PROJECT_ID firewall-rules create default-allow-https --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:443 --source-ranges=0.0.0.0/0 --target-tags=https-server

#   Update container image
# gcloud compute instances update-container $INSTANCE_NAME  \
# --container-image gcr.io/$DOMAIN_PROJECT_ID/register-computer:latest
# --container-env=^,@^PROJECTS_DN=[PROJECT_OU_DN],@AD_USERNAME=[DOMAIN]\\[AD_USER],@AD_PASSWORD=$PASSWORD,@AD_DOMAIN=[DOMAIN]
```

-   Once deployed, wait for a few minutes until container is up and running and browse to the public IP of the compute instance.

![Image](../docs/img/2020-02-15-16-41-38.png)

#####    Create Cloud KMS (Optional)

We use Cloud KMS to securely store password.

-   Enable Cloud KMS API and create a Cloud KMS instance

```shell
gcloud services enable cloudkms.googleapis.com --project=$DOMAIN_PROJECT_ID

gcloud kms keyrings create computer-registrar-keyring \
 --location global \
 --project=$DOMAIN_PROJECT_ID

gcloud kms keys create computer-registrar-key \
 --location=global \
 --purpose=encryption \
 --keyring=computer-registrar-keyring \
 --project=$DOMAIN_PROJECT_ID
```

-   Grant Domain Join API's Compute instance Service Account permission to access Cloud KMS

```shell
export REGISTRAR_EMAIL=[SERVOCE_ACCOUNT_ID]@$DOMAIN_PROJECT_ID.iam.gserviceaccount.com
gcloud kms keys add-iam-policy-binding computer-registrar-key \
 --location=global \
 --keyring=computer-registrar-keyring \
 --member=serviceAccount:$REGISTRAR_EMAIL \
 --role=roles/cloudkms.cryptoKeyEncrypterDecrypter \
 --project=$DOMAIN_PROJECT_ID
```

-   Grant compite/viewer role to Service Account

```shell
export TEST_PROJECT_ID=$DOMAIN_PROJECT_ID
gcloud projects add-iam-policy-binding $TEST_PROJECT_ID \
 --member "serviceAccount:$REGISTRAR_EMAIL" \
 --role "roles/compute.viewer"
```
