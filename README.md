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