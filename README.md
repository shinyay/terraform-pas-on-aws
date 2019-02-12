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

## Terraform
### Variables File - `terraform.tfvars`

### Terrafom Apply

```
$ terraform init
$ terraform plan -out=plan
$ terraform apply plan
```

