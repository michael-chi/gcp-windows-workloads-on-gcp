##  Overview

####    Create new VM

-   Create a new virutal machine

```shell
export REGISTER_URL=https://[DOMAIN_JOIN_API_INTERNAL_IP]/register-computer
export VPC_REGION=[VPC_REGION]
export VPC_SUBNET=[SUBNET_NAME]
export ZONE=[ZONE]

gcloud compute instances create verify-join-01 \
 --image-family=windows-2016-core \
 --image-project=windows-cloud \
 --machine-type=n1-standard-2 \
 --no-address \
 --subnet=$VPC_SUBNET \
 --zone=$ZONE \
 "--metadata=sysprep-specialize-script-ps1=iex((New-Object System.Net.WebClient).DownloadString('$REGISTER_URL'))" \
 --project=$TEST_PROJECT_ID \
 --container-env AD_DOMAIN=demo.local,AD_USERNAME=kalschi,PROJECTS_DN=OU="xxx,OU=GCP,OU=Cloud,DC=demo,DC=local",AD_PASSWORD=$PASSWORD
```