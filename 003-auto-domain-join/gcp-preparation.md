##  Overview

We need to setup our GCP project in order to access domain resources. In our case, we need Cloud Function to access domain resource. Hence we need Serverless VPC access here.


####    Setup Serverless VPC Access

-   Go to Google Cloud Console

-   Open up Cloud shell

-   Execute below commands, where

    -   your_gcp_project is your GCP project id
    -   vpc-name is your vpn name which will be using for Serverless VPC access
    -   serverless-region is the region you used for serverless VPC access
    -   serverless-ip-range is an unused /28 IP range for Serverless VPN access

```shell
export DOMAIN_PROJECT_ID=your_gcp_project
export VPC_NAME=vpc-name
export SERVERLESS_REGION=serverless-region
export SERVERLESS_IP_RANGE=serverless-ip-range
```

-   Clone sample domain join applications from [here](https://github.com/GoogleCloudPlatform/gce-automated-ad-join/tree/master/register-computer).

    Also created a [Dockerfile](./src//auto-domain-join/register-computer/Dockerfile) so that we can deploy it to servives such as GKE.

-   Build docker image and push to Google Container Registry