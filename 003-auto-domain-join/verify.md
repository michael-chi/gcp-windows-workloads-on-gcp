##  Overview

####    Create new VM

-   Create a new virutal machine

```shell
export REGISTER_URL=https://[GCP_DEFAULT_INTERNAL_FQDN]/register-computer
export VPC_REGION=[VPC_REGION]
export VPC_SUBNET=[SUBNET_NAME]
export ZONE=[ZONE]

export true='$true'
export text='$text'
export METADATA="sysprep-specialize-script-ps1=[System.Net.ServicePointManager]::ServerCertificateValidationCallback={$true};[Net.ServicePointManager]::SecurityProtocol=[Net.SecurityProtocolType]::Tls12 -bor [Net.SecurityProtocolType]::Tls12 -bor [Net.SecurityProtocolType]::Tls -bor [Net.SecurityProtocolType]::Ssl3;$text=(New-Object System.Net.WebClient).DownloadString('$REGISTER_URL'); iex($text)"


gcloud compute instances create verify-join-04 \
--image-family=windows-2016-core \
--image-project=windows-cloud \
--machine-type=n1-standard-2 \
--no-address \
--subnet=$VPC_SUBNET \
--zone=$ZONE \
--metadata=$METADATA \
--project=$TEST_PROJECT_ID
```