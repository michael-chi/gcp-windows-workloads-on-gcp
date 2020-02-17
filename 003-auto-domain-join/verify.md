##  Overview

####    Create new VM with HTTP Domain Join API Endpoint

-   Create a new virutal machine

```shell
export REGISTER_URL=http://[GCP_DEFAULT_INTERNAL_FQDN]/register-computer
export VPC_REGION=[VPC_REGION]
export VPC_SUBNET=[SUBNET_NAME]
export ZONE=[ZONE]
export TEST_PROJECT_ID=$DOMAIN_PROJECT_ID

gcloud compute instances create verify-join-01 \
--image-family=windows-2016-core \
--image-project=windows-cloud \
--machine-type=n1-standard-2 \
--no-address \
--subnet=$VPC_SUBNET \
--zone=$ZONE \
"--metadata=sysprep-specialize-script-ps1=$text=(New-Object System.Net.WebClient).DownloadString('$REGISTER_URL'); iex($text)" \
--project=$TEST_PROJECT_ID
```


####    Create new VM with HTTPS Domain Join API Endpoint

-   Create a new virutal machine

    I experienced many issue while using HTTPS endpoint, all of those came from Powershell Invoke-WebRequest or Invoke-RestMethod could not properly handly TLS/SSL connection with domain join API endpoints. 

    Most of internet posts suggest to add Tls1.2 support via Powershell cmdlet, however, in my case, I still experience errors.

```shell
export REGISTER_URL=https://[GCP_DEFAULT_INTERNAL_FQDN]/register-computer
export VPC_REGION=[VPC_REGION]
export VPC_SUBNET=[SUBNET_NAME]
export ZONE=[ZONE]
export TEST_PROJECT_ID=$DOMAIN_PROJECT_ID
export true='$true'
export text='$text'

gcloud compute instances create verify-join-01 \
--image-family=windows-2016-core \
--image-project=windows-cloud \
--machine-type=n1-standard-2 \
--no-address \
--subnet=$VPC_SUBNET \
--zone=$ZONE \
"--metadata=sysprep-specialize-script-ps1=[System.Net.ServicePointManager]::ServerCertificateValidationCallback={$true};[Net.ServicePointManager]::SecurityProtocol=[Net.SecurityProtocolType]::Tls12 -bor [Net.SecurityProtocolType]::Tls11 -bor [Net.SecurityProtocolType]::Tls -bor [Net.SecurityProtocolType]::Ssl3;$text=(New-Object System.Net.WebClient).DownloadString('$REGISTER_URL'); iex($text)" \
--project=$TEST_PROJECT_ID
```