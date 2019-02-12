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

