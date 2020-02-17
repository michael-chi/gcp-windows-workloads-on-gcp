##  Overview

Below diagram illustrated process flow of domain automated join.

```mermaid
sequenceDiagram

GCP ->> Windows VM: Create VM
Windows VM ->> AD Join API(GCE): download startup script
AD Join API(GCE) -->> Windows VM: download startup script
Windows VM ->> Windows VM: Execute script
Windows VM ->> Metadata server: Get Id token
Metadata server -->> Windows VM: Get Id token
Windows VM ->> AD Join API(GCE): Register computer
AD Join API(GCE) ->> Domain Controller: Validate OU
Domain Controller -->> AD Join API(GCE): Validate OU
AD Join API(GCE) ->> GCE API: Validate project access
GCE API -->> AD Join API(GCE): Validate project access
AD Join API(GCE) ->> Domain Controller: Create computer account and password
Domain Controller -->> AD Join API(GCE): Create computer account and password
AD Join API(GCE) ->> Windows VM: computer account credential
Windows VM ->> Domain Controller: Join domain
Windows VM ->> Windows VM: Reboot
```
##  Table of Content

[Domain Preparation](./domain-preparation.md)

[GCP Preparation](./gcp-preparation.md)

[Verify Domain auto join](./verify.md)

##  Reference

[GCP Automated AD Join](https://github.com/GoogleCloudPlatform/gce-automated-ad-join)