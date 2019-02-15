# Tarraform PAS for AWS
---

## Preparation
### AWS CLI

```
$ brew install awscli
```

### AWS CLI Configuration

```
$ aws configure
AWS Access Key ID [None]: AKIA......
AWS Secret Access Key [None]: 6wKCbAs........
Default region name [None]: ap-northeast-1
Default output format [None]: json
```

### Download PAS Terraform Template for AWS

```
$ pivnet login --api-token='27f8.........'
$ pivnet product-files -p elastic-runtime -r 2.4.2
$ pivnet download-product-files -p elastic-runtime -r 2.4.2 -i 258595
```

```
$ pivnet releases -p ops-manager
$ pivnet product-files -p ops-manager -r 2.4.3
$ pivnet download-product-files -p ops-manager -r 2.4.3 -i 302866
```

## Terraform
### Variables File - `terraform.tfvars`

```
env_name           = "YOUR-ENVIRONMENT-NAME"
access_key         = "YOUR-ACCESS-KEY"
secret_key         = "YOUR-SECRET-KEY"
region             = "ap-northeast-1"
availability_zones = ["Yap-northeast-1a", "ap-northeast-1c", "ap-northeast-1d"]
ops_manager_ami    = "YOUR-OPS-MAN-IMAGE-AMI"
dns_suffix         = "YOUR-DNS-SUFFIX"

ssl_cert = <<SSL_CERT
-----BEGIN CERTIFICATE-----
YOUR-CERTIFICATE
-----END CERTIFICATE-----
SSL_CERT

ssl_private_key = <<SSL_KEY
-----BEGIN EXAMPLE RSA PRIVATE KEY-----
YOUR-PRIVATE-KEY
-----END EXAMPLE RSA PRIVATE KEY-----
SSL_KEY
```

#### AWS Availability Zone

```
$ aws ec2 describe-availability-zones --region ap-northeast-1
```


### Terrafom Apply

```
$ terraform init
$ terraform plan -out=plan
$ terraform apply plan
```

### DNS Record

```
$ cat terraform.tfstate | jq -r '.modules[0].outputs.env_dns_zone_name_servers.value'
```

## OpsManager

### Access OpsManager

- https://$OPS_DOMAIN

```
$ OPS_DOMAIN = cat terraform.tfstate | jq -r '.modules[0].outputs.ops_manager_dns.value'
```

### AWS Config

|Input|Value|
|-----|-----|
|Access Key ID|cat terraform.tfstate \| jq -r '.modules[0].outputs.ops_manager_iam_user_access_key.value'|
|AWS Secret Key|cat terraform.tfstate \| jq -r '.modules[0].outputs.ops_manager_iam_user_secret_key.value'|
|Security Group ID|cat terraform.tfstate \| jq -r '.modules[0].outputs.vms_security_group_id.value'|
|Key Pair Name|cat terraform.tfstate \| jq -r '.modules[0].outputs.ops_manager_ssh_public_key_name.value'|
|SSH Private Key|cat terraform.tfstate \| jq -r '.modules[0].outputs.ops_manager_ssh_private_key.value'|
|Region|cat terraform.tfstate \| jq -r '.modules[0].outputs.region.value'|

### Director Config

|Input|Value|
|-----|-----|
|NTP Servers|0.amazon.pool.ntp.org,1.amazon.pool.ntp.org,2.amazon.pool.ntp.org,3.amazon.pool.ntp.org|
|JMX Provider IP Address|---|
|Bosh HM Forwarder IP Address|---|
|Enable VM Resurrector Plugin|TRUE|
|Enable Post Deploy Scripts|TRUE|
|Recreate all VMs|TRUE|
|Recreate All Persistent Disks|TRUE|
|Enable bosh deploy retries |TRUE|
|Skip Director Drain Lifecycle|TRUE|
|Allow Legacy Agents|TRUE|
|Keep Unreachable Director VMs|FALSE|
|HM Pager Duty Plugin|FALSE|
|HM Email Plugin|FALSE|
|CredHub Encryption Provider|Internal|
|Blobstore Location|Internal|
|Enable TLS|TRUE|
|Database Location|Internal|
|Director Workers|5|
|Max Threads|---|
|Director Hostname|---|
|Custom SSH Banner|---|
|Identification Tags|---|

### Create Availability Zones

|Input|Value|
|-----|-----|
|Amazon Availability Zone|cat terraform.tfstate \| jq -r '.modules[0].outputs.management_subnet_availability_zones.value'|

### Create Networks
#### Infrastructure

|Input|Value|
|-----|-----|
|Verification Settings|FALSE|
|Networks|Add Network|
|Name|infrastructure|
|1st subnet|---|
|VPC Subnet ID|cat terraform.tfstate | jq -r '.modules[0].outputs.infrastructure_subnet_ids.value[0]'|
|CIDR|cat terraform.tfstate | jq -r '.modules[0].outputs.infrastructure_subnet_cidrs.value[0]'<br>Ex. 10.0.16.0/28|
|Reserved IP Ranges|10.0.16.0-10.0.16.4|
|DNS|10.0.0.2|
|Gateway|cat terraform.tfstate | jq -r '.modules[0].outputs.infrastructure_subnet_gateways.value[0]'|
|Availability Zones|cat terraform.tfstate | jq -r '.modules[0].outputs.infrastructure_subnet_availability_zones.value[0]'|
|2nd subnet|---|
|VPC Subnet ID|cat terraform.tfstate | jq -r '.modules[0].outputs.infrastructure_subnet_ids.value[1]'|
|CIDR|cat terraform.tfstate | jq -r '.modules[0].outputs.infrastructure_subnet_cidrs.value[1]'<br>Ex. 10.0.16.16/28|
|Reserved IP Ranges|10.0.16.16-10.0.16.20|
|DNS|10.0.0.2|
|Gateway|cat terraform.tfstate | jq -r '.modules[0].outputs.infrastructure_subnet_gateways.value[1]'|
|Availability Zones|cat terraform.tfstate | jq -r '.modules[0].outputs.infrastructure_subnet_availability_zones.value[1]'|
|3rd subnet|---|
|VPC Subnet ID|cat terraform.tfstate | jq -r '.modules[0].outputs.infrastructure_subnet_ids.value[2]'|
|CIDR|cat terraform.tfstate | jq -r '.modules[0].outputs.infrastructure_subnet_cidrs.value[2]'<br>Ex. 10.0.16.32/28|
|Reserved IP Ranges|10.0.16.32-10.0.16.36|
|DNS|10.0.0.2|
|Gateway|cat terraform.tfstate | jq -r '.modules[0].outputs.infrastructure_subnet_gateways.value[2]'|
|Availability Zones|cat terraform.tfstate | jq -r '.modules[0].outputs.infrastructure_subnet_availability_zones.value[2]'|

#### PAS

|Input|Value|
|-----|-----|
|Name|pas|
|1st subnet|---|
|VPC Subnet ID|cat terraform.tfstate | jq -r '.modules[0].outputs.pas_subnet_ids.value[0]'|
|CIDR|cat terraform.tfstate | jq -r '.modules[0].outputs.pas_subnet_cidrs.value[0]'<br>Ex. 10.0.4.0/24|
|Reserved IP Ranges|10.0.4.0-10.0.4.4|
|DNS|10.0.0.2|
|Gateway|cat terraform.tfstate | jq -r '.modules[0].outputs.pas_subnet_gateways.value[0]'|
|Availability Zones|cat terraform.tfstate | jq -r '.modules[0].outputs.pas_subnet_availability_zones.value[0]'|
|2nd subnet|---|
|VPC Subnet ID|cat terraform.tfstate | jq -r '.modules[0].outputs.pas_subnet_ids.value[1]'|
|CIDR|cat terraform.tfstate | jq -r '.modules[0].outputs.pas_subnet_cidrs.value[1]'<br>Ex. 10.0.5.0/24|
|Reserved IP Ranges|10.0.5.0-10.0.5.4|
|DNS|10.0.0.2|
|Gateway|cat terraform.tfstate | jq -r '.modules[0].outputs.pas_subnet_gateways.value[1]'|
|Availability Zones|cat terraform.tfstate | jq -r '.modules[0].outputs.pas_subnet_availability_zones.value[1]'|
|3rd subnet|---|
|VPC Subnet ID|cat terraform.tfstate | jq -r '.modules[0].outputs.pas_subnet_ids.value[2]'|
|CIDR|cat terraform.tfstate | jq -r '.modules[0].outputs.pas_subnet_cidrs.value[2]'<br>Ex. 10.0.6.0/24|
|Reserved IP Ranges|10.0.6.0-10.0.6.4|
|DNS|10.0.0.2|
|Gateway|cat terraform.tfstate | jq -r '.modules[0].outputs.pas_subnet_gateways.value[2]'|
|Availability Zones|cat terraform.tfstate | jq -r '.modules[0].outputs.pas_subnet_availability_zones.value[2]'|

#### Services

|Input|Value|
|-----|-----|
|Name|services|
|1st subnet|---|
|VPC Subnet ID|cat terraform.tfstate | jq -r '.modules[0].outputs.services_subnet_ids.value[0]'|
|CIDR|cat terraform.tfstate | jq -r '.modules[0].outputs.services_subnet_cidrs.value[0]'<br>Ex. 10.0.8.0/24|
|Reserved IP Ranges|10.0.8.0-10.0.8.3|
|DNS|10.0.0.2|
|Gateway|cat terraform.tfstate | jq -r '.modules[0].outputs.services_subnet_gateways.value[0]'|
|Availability Zones|cat terraform.tfstate | jq -r '.modules[0].outputs.services_subnet_availability_zones.value[0]'|
|2nd subnet|---|
|VPC Subnet ID|cat terraform.tfstate | jq -r '.modules[0].outputs.services_subnet_ids.value[1]'|
|CIDR|cat terraform.tfstate | jq -r '.modules[0].outputs.services_subnet_cidrs.value[1]'<br>Ex. 10.0.9.0/24|
|Reserved IP Ranges|10.0.9.0-10.0.9.4|
|DNS|10.0.0.2|
|Gateway|cat terraform.tfstate | jq -r '.modules[0].outputs.services_subnet_gateways.value[1]'|
|Availability Zones|cat terraform.tfstate | jq -r '.modules[0].outputs.services_subnet_availability_zones.value[1]'|
|3rd subnet|---|
|VPC Subnet ID|cat terraform.tfstate | jq -r '.modules[0].outputs.services_subnet_ids.value[2]'|
|CIDR|cat terraform.tfstate | jq -r '.modules[0].outputs.services_subnet_cidrs.value[2]'<br>Ex. 10.0.10.0/24|
|Reserved IP Ranges|10.0.10.0-10.0.10.4|
|DNS|10.0.0.2|
|Gateway|cat terraform.tfstate | jq -r '.modules[0].outputs.services_subnet_gateways.value[2]'|
|Availability Zones|cat terraform.tfstate | jq -r '.modules[0].outputs.services_subnet_availability_zones.value[2]'|

### Assign AZs and Networks

|Input|Value|
|-----|-----|
|Singleton Availability Zone|ap-northeast-1a|
|Network|infrastructure|

### Security

|Input|Value|
|-----|-----|
|Trusted Certificates|---|
| Include OpsManager Root CA in Trusted Certs|FALSE|
|Generate VM passwords or use single password for all VMs|Generate passwords|

### BOSH DNS Config

|Input|Value|
|-----|-----|
|Excluded Recursors|---|
|Recursor Timeout|---|
|Handlers|[]|

### Syslog

|Input|Value|
|-----|-----|
|Do you want to configure Syslog for Bosh Director?|No|

### Resource Config

- Default

## SSH to OpsManager VM
### Private Key

```
$ cat terraform.tfstate | jq -r .modules[0].outputs.ops_manager_ssh_private_key.value > ops_mgr.pem
$ chmod 600 ops_mgr.pem
```

### SSH

```
$ ssh -i ops_mgr.pem ubuntu@OPSMAN_HOST_ADDRESS
```

## [OPSMGR] CLI for PCF
### OM CLI

```
$ wget https://github.com/pivotal-cf/om/releases/download/0.51.0/om-linux
$ sudo mv om-linux /usr/local/bin/om
$ sudo chmod +x /usr/local/bin/om
```

### Pivnet CLI

```
$ wget https://github.com/pivotal-cf/pivnet-cli/releases/download/v0.0.55/pivnet-linux-amd64-0.0.55
$ sudo mv pivnet-linux* /usr/local/bin/pivnet
$ sudo chmod +x /usr/local/bin/pivnet
```

## [OPSMGR] Download PAS

```
$ pivnet login --api-token='27f8.........'
$ pivnet product-files -p elastic-runtime -r 2.4.2
$ pivnet download-product-files -p elastic-runtime -r 2.4.2 -i 293808
```

## [OPSMGR] Download Stemcell

```
$ pivnet releases -p stemcells-ubuntu-xenial
$ pivnet product-files -p stemcells-ubuntu-xenial -r 170.25
$ pivnet download-product-files -p stemcells-ubuntu-xenial -r 170.25 -i 303825
```

## [OPSMGR] Upload PAS Install Image

- `om --target https://$OPS_MGR_DNS -k -u $OPS_MGR_USR -p $OPS_MGR_PWD --request-timeout 3600 upload-product -p ~/$FILENAME`

```
$ om --target https://localhost -k -u admin -p admin --request-timeout 3600 upload-product -p ~/cf-2.4.2-build.33.pivotal
```

## [OPSMGR] Stage PAS

- `om --target https://$OPS_MGR_DNS -k -u $OPS_MGR_USR -p $OPS_MGR_PWD stage-product -p $PRODUCT_NAME -v $PRODUCT_VERSION`

```
$ om --target https://localhost -k -u admin -p admin stage-product -p cf -v 2.4.2
```

## PAS on Azure

### Assign AZ and Networks

|Input|Value|
|-----|-----|
|Place singleton jobs in|どれか|
|Balance other jobs in|全て|
|Network|pas|
