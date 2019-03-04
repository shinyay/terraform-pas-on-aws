# Tarraform PAS for AWS
---

## Preparation
### AWS Instance Limits

- https://ap-northeast-1.console.aws.amazon.com/ec2/v2/home?region=ap-northeast-1#Limits:
  - t2.micro
    - 50
  - c4.large
    - 20

### Terraform

```
$ brew install terraform
```
### jq

```
$ brew install jq
```

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

### DNS Record Configuration
- Check your Name Server confired by Terraform

```
$ cat terraform.tfstate | jq -r '.modules[0].outputs.env_dns_zone_name_servers.value'
```

- Manage your DNS service to configure your name service above


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
|**1ST SUBNET**|---|
|VPC Subnet ID|cat terraform.tfstate \| jq -r '.modules[0].outputs.infrastructure_subnet_ids.value[0]'|
|CIDR|cat terraform.tfstate \| jq -r '.modules[0].outputs.infrastructure_subnet_cidrs.value[0]'<br>Ex. 10.0.16.0/28|
|Reserved IP Ranges|10.0.16.0-10.0.16.4|
|DNS|10.0.0.2|
|Gateway|cat terraform.tfstate \| jq -r '.modules[0].outputs.infrastructure_subnet_gateways.value[0]'|
|Availability Zones|cat terraform.tfstate \| jq -r '.modules[0].outputs.infrastructure_subnet_availability_zones.value[0]'|
|2nd subnet|---|
|VPC Subnet ID|cat terraform.tfstate \| jq -r '.modules[0].outputs.infrastructure_subnet_ids.value[1]'|
|CIDR|cat terraform.tfstate \| jq -r '.modules[0].outputs.infrastructure_subnet_cidrs.value[1]'<br>Ex. 10.0.16.16/28|
|Reserved IP Ranges|10.0.16.16-10.0.16.20|
|DNS|10.0.0.2|
|Gateway|cat terraform.tfstate \| jq -r '.modules[0].outputs.infrastructure_subnet_gateways.value[1]'|
|Availability Zones|cat terraform.tfstate \| jq -r '.modules[0].outputs.infrastructure_subnet_availability_zones.value[1]'|
|3rd subnet|---|
|VPC Subnet ID|cat terraform.tfstate \| jq -r '.modules[0].outputs.infrastructure_subnet_ids.value[2]'|
|CIDR|cat terraform.tfstate \| jq -r '.modules[0].outputs.infrastructure_subnet_cidrs.value[2]'<br>Ex. 10.0.16.32/28|
|Reserved IP Ranges|10.0.16.32-10.0.16.36|
|DNS|10.0.0.2|
|Gateway|cat terraform.tfstate \| jq -r '.modules[0].outputs.infrastructure_subnet_gateways.value[2]'|
|Availability Zones|cat terraform.tfstate \| jq -r '.modules[0].outputs.infrastructure_subnet_availability_zones.value[2]'|

#### PAS

|Input|Value|
|-----|-----|
|Name|pas|
|1st subnet|---|
|VPC Subnet ID|cat terraform.tfstate \| jq -r '.modules[0].outputs.pas_subnet_ids.value[0]'|
|CIDR|cat terraform.tfstate \| jq -r '.modules[0].outputs.pas_subnet_cidrs.value[0]'<br>Ex. 10.0.4.0/24|
|Reserved IP Ranges|10.0.4.0-10.0.4.4|
|DNS|10.0.0.2|
|Gateway|cat terraform.tfstate \| jq -r '.modules[0].outputs.pas_subnet_gateways.value[0]'|
|Availability Zones|cat terraform.tfstate \| jq -r '.modules[0].outputs.pas_subnet_availability_zones.value[0]'|
|2nd subnet|---|
|VPC Subnet ID|cat terraform.tfstate \| jq -r '.modules[0].outputs.pas_subnet_ids.value[1]'|
|CIDR|cat terraform.tfstate \| jq -r '.modules[0].outputs.pas_subnet_cidrs.value[1]'<br>Ex. 10.0.5.0/24|
|Reserved IP Ranges|10.0.5.0-10.0.5.4|
|DNS|10.0.0.2|
|Gateway|cat terraform.tfstate \| jq -r '.modules[0].outputs.pas_subnet_gateways.value[1]'|
|Availability Zones|cat terraform.tfstate \| jq -r '.modules[0].outputs.pas_subnet_availability_zones.value[1]'|
|3rd subnet|---|
|VPC Subnet ID|cat terraform.tfstate \| jq -r '.modules[0].outputs.pas_subnet_ids.value[2]'|
|CIDR|cat terraform.tfstate \| jq -r '.modules[0].outputs.pas_subnet_cidrs.value[2]'<br>Ex. 10.0.6.0/24|
|Reserved IP Ranges|10.0.6.0-10.0.6.4|
|DNS|10.0.0.2|
|Gateway|cat terraform.tfstate \| jq -r '.modules[0].outputs.pas_subnet_gateways.value[2]'|
|Availability Zones|cat terraform.tfstate \| jq -r '.modules[0].outputs.pas_subnet_availability_zones.value[2]'|

#### Services

|Input|Value|
|-----|-----|
|Name|services|
|1st subnet|---|
|VPC Subnet ID|cat terraform.tfstate \| jq -r '.modules[0].outputs.services_subnet_ids.value[0]'|
|CIDR|cat terraform.tfstate \| jq -r '.modules[0].outputs.services_subnet_cidrs.value[0]'<br>Ex. 10.0.8.0/24|
|Reserved IP Ranges|10.0.8.0-10.0.8.3|
|DNS|10.0.0.2|
|Gateway|cat terraform.tfstate \| jq -r '.modules[0].outputs.services_subnet_gateways.value[0]'|
|Availability Zones|cat terraform.tfstate \| jq -r '.modules[0].outputs.services_subnet_availability_zones.value[0]'|
|2nd subnet|---|
|VPC Subnet ID|cat terraform.tfstate \| jq -r '.modules[0].outputs.services_subnet_ids.value[1]'|
|CIDR|cat terraform.tfstate \| jq -r '.modules[0].outputs.services_subnet_cidrs.value[1]'<br>Ex. 10.0.9.0/24|
|Reserved IP Ranges|10.0.9.0-10.0.9.3|
|DNS|10.0.0.2|
|Gateway|cat terraform.tfstate \| jq -r '.modules[0].outputs.services_subnet_gateways.value[1]'|
|Availability Zones|cat terraform.tfstate \| jq -r '.modules[0].outputs.services_subnet_availability_zones.value[1]'|
|3rd subnet|---|
|VPC Subnet ID|cat terraform.tfstate \| jq -r '.modules[0].outputs.services_subnet_ids.value[2]'|
|CIDR|cat terraform.tfstate \| jq -r '.modules[0].outputs.services_subnet_cidrs.value[2]'<br>Ex. 10.0.10.0/24|
|Reserved IP Ranges|10.0.10.0-10.0.10.3|
|DNS|10.0.0.2|
|Gateway|cat terraform.tfstate \| jq -r '.modules[0].outputs.services_subnet_gateways.value[2]'|
|Availability Zones|cat terraform.tfstate \| jq -r '.modules[0].outputs.services_subnet_availability_zones.value[2]'|

### Assign AZs and Networks

|Input|Value|
|-----|-----|
|Singleton Availability Zone|ap-northeast-1a|
|Network|infrastructure|

### Security

- Default

|Input|Value|
|-----|-----|
|Trusted Certificates|---|
| Include OpsManager Root CA in Trusted Certs|FALSE|
|Generate VM passwords or use single password for all VMs|Generate passwords|

### BOSH DNS Config

- Default

|Input|Value|
|-----|-----|
|Excluded Recursors|---|
|Recursor Timeout|---|
|Handlers|[]|

### Syslog

- Default

|Input|Value|
|-----|-----|
|Do you want to configure Syslog for Bosh Director?|No|

### Resource Config

- Default

## Apply Change

- Apply Change

## SSH to OpsManager VM
### Private Key

```
$ cat terraform.tfstate | jq -r .modules[0].outputs.ops_manager_ssh_private_key.value > ops_mgr.pem
$ chmod 600 ops_mgr.pem
```

### SSH

```
$ OPSMAN_HOST_ADDRESS = aws ec2 describe-instances --filter "Name=tag:Name,Values=pcf-ops-manager"|jq -r .Reservations[].Instances[].PublicIpAddress
```

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
$ pivnet product-files -p elastic-runtime -r 2.4.3
$ pivnet download-product-files -p elastic-runtime -r 2.4.3 -i 310780
```

## [OPSMGR] Download Stemcell

```
$ pivnet releases -p stemcells-ubuntu-xenial
$ pivnet product-files -p stemcells-ubuntu-xenial -r 170.30
$ pivnet download-product-files -p stemcells-ubuntu-xenial -r 170.30 -i 313919
```

## [OPSMGR] Upload PAS Install Image

- `om --target https://$OPS_MGR_DNS -k -u $OPS_MGR_USR -p $OPS_MGR_PWD --request-timeout 3600 upload-product -p ~/$FILENAME`

```
$ om --target https://localhost -k -u admin -p admin --request-timeout 3600 upload-product -p ~/cf-2.4.3-build.18.pivotal
```

## [OPSMGR] Upload Stemcell Image

- `om --target https://$OPS_MGR_DNS -k -u $OPS_MGR_USR -p $OPS_MGR_PWD --request-timeout 3600 upload-stemcell -s ~/$STEMCELL`

```
$ om --target https://localhost -k -u admin -p admin --request-timeout 3600 upload-stemcell -s ~/light-bosh-stemcell-170.30-aws-xen-hvm-ubuntu-xenial-go_agent.tgz
```

## [OPSMGR] Stage PAS

- `om --target https://$OPS_MGR_DNS -k -u $OPS_MGR_USR -p $OPS_MGR_PWD stage-product -p $PRODUCT_NAME -v $PRODUCT_VERSION`

```
$ om --target https://localhost -k -u admin -p admin stage-product -p cf -v 2.4.3
```

## PAS on AWS

### Assign AZ and Networks

|Input|Value|
|-----|-----|
|Place singleton jobs in|どれか|
|Balance other jobs in|全て|
|Network|pas|

### Domains

|Input|Value|
|-----|-----|
|System Domain|cat terraform.tfstate \| jq -r '.modules[0].outputs.sys_domain.value'|
|Apps Domain|cat terraform.tfstate \| jq -r '.modules[0].outputs.apps_domain.value'|

### Networking

|Input|Value|
|-----|-----|
|Router IPs|---|
|SSH Proxy IPs|---|
|HAProxy IPs|---|
|TCP Router IPs|---|
|Certificates and Private Keys for HAProxy and Router|Add|
|Name|pas-cert|
|Generate RSA Certificate|MY_DOMAIN = YOUR-ENVIRONMENT-NAME.YOUR-DOMAIN<br>\*.$MY_DOMAIN,\*.sys.$MY_DOMAIN,\*.apps.$MY_DOMAIN,login.sys.$MY_DOMAIN,uaa.sys.$MY_DOMAIN,doppler.sys.$MY_DOMAIN,loggregator.sys.$MY_DOMAIN,ssh.sys.$MY_DOMAIN,tcp.$MY_DOMAIN,opsman.$MY_DOMAIN<br><br>[SAMPLE]<br>\*.mypcf.syanagihara.cf,\*.sys.mypcf.syanagihara.cf,\*.apps.mypcf.syanagihara.cf,login.sys.mypcf.syanagihara.cf,uaa.sys.mypcf.syanagihara.cf,doppler.sys.mypcf.syanagihara.cf,loggregator.sys.mypcf.syanagihara.cf,ssh.sys.mypcf.syanagihara.cf,tcp.mypcf.syanagihara.cf,opsman.mypcf.syanagihara.cf|
|Certificate Authorities Trusted by Router and HAProxy|---|
|Minimum version of TLS supported by HAProxy and Router|<DEFAULT><br>TLSv1.2|
|Logging of Client IPs in CF Router|<DEFAULT><br>Log client IPs|
|Configure support for the X-Forwarded-Client-Cert header|<DEFAULT><br>TLS terminated for the first time at infrastructure load balancer|
|HAProxy behavior for Client Certificate Validation|<DEFAULT><br>HAProxy does not request client certificates|
|Router behavior for Client Certificate Validation|<DEFAULT><br>Router requests but does not require client certificates|
|TLS Cipher Suites for Router|<DEFAULT>|
|TLS Cipher Suites for HAProxy|<DEFAULT>|
|HAProxy forwards requests to Router over TLS|Disable|
|HAProxy support for HSTS|Disable|
|Disable SSL certificate verification for this environment|TRUE|
|Disable HTTP on HAProxy and Router|<DEFAULT><br>FALSE|
|Disable insecure cookies on the Router|<DEFAULT><br>FALSE|
|Enable Zipkin tracing headers on the Router|<DEFAULT><br>TRUE|
|Enable Router to write access logs locally|<DEFAULT><br>TRUE|
|Routers reject requests for Isolation Segments|<DEFAULT><br>FALSE|
|Enable support for PROXY protocol in CF Router|<DEFAULT><br>FALSE|
|Choose whether to enable route services|<DEFAULT><br>Enable route services|
|Max Connections Per Backend|<DEFAULT><br>500|
|Enable Keepalive Connections for Router|<DEFAULT><br>Enable|
|Router Timeout to Backends|<DEFAULT><br>900|
|Frontend Idle Timeout for Router and HAProxy|<DEFAULT><br>900|
|Load Balancer Unhealthy Threshold|<DEFAULT><br>20|
|Load Balancer Healthy Threshold|<DEFAULT><br>20|
|HTTP Headers to Log|---|
|HAProxy Request Max Buffer Size|<DEFAULT><br>16384|
|HAProxy Protected Domains|---|
|HAProxy Trusted CIDRs|---|
|Loggregator Port|---|
|Container Network Interface Plugin|Silk|
|DNS Search Domains|---|
|Database Connection Timeout|120|
|Enable TCP requests to your apps via specific ports on the TCP Router|Select this option if you prefer to enable TCP Routing at a later time|

### Application Containers

- DEFAULT

### Application Developer Controls

- DEFAULT

### Application Security Groups

- Type `X`

### Authentication and Enterprise SSO

- Internal UAA
  - DEFAULT

### UAA

|Input|Value|
|-----|-----|
|Choose the location of your UAA database|PAS database|
|JWT Issuer URI|---|
|SAML Service Provider Credentials|Generate RSA Certificate|
|SAML Service Provider Key Password|---|
|SAML Entity ID Override|---|

### CredHub

|Input|Value|
|-----|-----|
|Choose the location of your UAA database|PAS database|
|Encryption Keys|Add|
|Name|pas-encrypt|
|Key|<20字以上>|
|Primary|TRUE|

### Databases

- Internal Databases - MySQL - Percona XtraDB Cluster

### Internal MySQL

- E-mail address (required)

### File Storage

- DEFAULT

### System Logging

- DEFAULT

### Custom Branding

- DEFAULT

### Apps Manager

- DEFAULT

### Email Notifications

- DEFAULT

### App Autoscaler

- DEFAULT

### Cloud Controller

- DEFAULT

### Smoke Tests

- DEFAULT

### Advanced Features

- DEFAULT

### Metric Registrar

- DEFAULT

### Errands

- DEFAULT

### Resource Config

|Input|Value|
|-----|-----|
|Router - LoadBalancers|cat terraform.tfstate \| jq -r .modules[0].outputs.web_elb_name.value|
|Diego Brain - LoadBalancers|cat terraform.tfstate \| jq -r .modules[0].outputs.ssh_elb_name.value|

## PAS VMs Stop and Start

### bosh login

```
$ bosh alias-env aws -e $DIRECTOR-IP-ADDRESS --ca-cert /var/tempest/workspaces/default/root_ca_certificate
```

```
$ bosh -e aws log-in
Using environment '$DIRECTOR-IP-ADDRESS'

Email (): director
Password (): $DIRECTOR_CREDENTIALS

Successfully authenticated with UAA

Succeeded
```

### VMS list

```
$ bosh -e aws vms
```

### PAS VMs Stop

```
$ bosh -e aws -d $DEPLOYMENT stop --hard
```

### PAS VMs Start

```
$ find /var/tempest/workspaces/default/deployments -name cf-*.yml
$ bosh -e aws -d $DEPLOYMENT start
```
