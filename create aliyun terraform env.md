# site aliyun

This repository contains templates and scripts for managing Infrastructure as Code (IaC) for the Web2Box project on Alibaba Cloud using [Terraform](https://www.terraform.io/).

## Requirements

* Python 2.7 (https://www.python.org/downloads/)
* Run `pip install -r requirements.txt` to install required python packages.
* Golang (https://golang.org/dl/)
* Terraform ([installation guide](https://www.terraform.io/intro/getting-started/install.html)).
* aliyun go cli - install according to read.me (https://github.com/aliyun/aliyun-cli) and copy aliyun.exe to Python script folder

## manjaro installation
* make sure, you are in the folder containing this README.md
* `sudo pacman -Syyu`
* `sudo pacman -S python-pip go jq`
* Python packages: `sudo pip install -r requirements.txt`
* Terraform 0.11.14: `sudo echo ; zcat <( curl -q "https://releases.hashicorp.com/terraform/0.11.14/terraform_0.11.14_linux_amd64.zip" ) | sudo tee /usr/local/bin/terraform > /dev/null ; sudo chmod +x /usr/local/bin/terraform`
* Terraform >= 0.12: `sudo echo ; zcat <( CURRR_VER=$(curl -s https://checkpoint-api.hashicorp.com/v1/check/terraform | jq -r -M '.current_version') ; curl -q "https://releases.hashicorp.com/terraform/$CURRR_VER/terraform_${CURRR_VER}_linux_amd64.zip" ) | sudo tee /usr/local/bin/terraform > /dev/null ; sudo chmod +x /usr/local/bin/terraform`
* aliyun go cli: `sudo echo ; curl --silent "https://api.github.com/repos/aliyun/aliyun-cli/releases/latest" | jq -r '.assets[] | select(.name | contains("linux")).browser_download_url' | xargs sudo curl -L --url | sudo tar xvz -C /usr/local/bin ; sudo chmod +x /usr/local/bin/aliyun`
* aliyunlog cli: `sudo pip install -U aliyun-log-cli`

## Configure credentials

Before managing the terraform stacks, you will need to configure your CLI credentials for API access:

1. Configure aliyun go cli by running `aliyun configure` and entering your API credentials. (Default Region Id [None]: `cn-shenzhen`, Default output format [None]: `json`)
2. Export the credentials environment variables needed by terraform. To do this, run the script `auth.sh` using the command `. auth.sh` or `source auth.sh` in a shell on Linux or `auth.bat` on Windows.

## Usage

To create/update/delete terraform stacks:

1. Navigate to the correct stack configuration directory under the `envs/` directory. For example, the VPC stack for the dev environment in Frankfurt region, the directory would be `envs/dev/cn-shenzhen/vpc/`.
2. Run the command `terraform init` to initialize the stack config and download required terraform plugins.
3. Run the command `terraform apply` which will apply any differences (i.e. create new resources or update existing ones) between the stack as defined in the terraform templates and what is currently the state in your Aliyun account.
4. To delete a stack, run the command `terraform destroy`.

## SSH keys

Different SSH keys are used to access instances created by Terraform. We use 3 keys per enviornment for 'Prinect/HDM', 'Packgate' and 'Magento' groups of instances. The keys should be named as `<env>-shenzhen`, `<env>-packgate` and `<env>-magento` respectively.

To create new keys for a new enviornment, follow these steps:
* Login to Alibaba web console.
* Go to `Elastic Compute Service` service page.
* From the left menu, choose the `SSH Key Pair` tab.
* Click on `Create SSH Key Pair` button in the top right corner.
* After entering the key name and clicking `OK`, the private key will be downloaded automatically. The private key can not be download again after this point so make sure its stored safely.

## Directory structure

Terraform files are structured in 2 main directories:

### modules

[Terraform modules](https://www.terraform.io/docs/modules/index.html) are a way of creating re-usable and parameterised templates. We group resources with a single purpose under a single module and place all modules in the `modules/` directory.

For example, all resources related to the VPC (vpc, vswitches, nat, etc) are grouped in a VPC module in the directory `modules/vpc`. This module can be used to create these resources in multiple regions/environments.

### envs

The directory `envs/` contain environment-specific configuration passed on to the appropriate modules. The directory is further structured in the format `<environment>/<region>/<module>/`.
