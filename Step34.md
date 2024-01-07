# Continous Integration Pipeline For Tooling Website

 ## STEP 1.  **Base infrastructure automation (VPC | Subnets)**
In my project file, I created a `main.tf` file, inserted the codes below, then ran the `terraform init` command the initialize terrfaorm in that directory.

1. I created an EC2 instance based on Ubuntun 22.04 and named it `Jenkins`, to install Jenkins, I ran the following commands;
 ```bash 
provider "aws" {
  region = "eu-west-2"
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = "172.16.0.0/16"
  enable_dns_support             = "true"
  enable_dns_hostnames           = "true"
}
```

![Screenshot](https://github.com/ardamz/my-demo/blob/main/Project16/TerraformInit.png)

2. I ran the `terraform plan` command to see a preview of what terraform intends to create, once I was happy with it, I ran the `terraform apply` command.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/Project16/TerraformPlan.png)


![Screenshot](https://github.com/ardamz/my-demo/blob/main/Project16/TerraformApply.png)

## STEP 2.  **Creating the Subnets**

1. Hardcoding values into codes is a general bad practice as it makes portability of codes difficult. 

```bash
# Create public subnets1
    resource "aws_subnet" "public1" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.0.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "eu-west-2a"

}

# Create public subnet2
    resource "aws_subnet" "public2" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.1.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "eu-west-2b"
}
```
I pasted the code above into the `main.tf` file and ran the `terraform plan` and the `terraform apply` command to create the two (2) subnets

![Screenshot](https://github.com/ardamz/my-demo/blob/main/Project16/Subnets.png)

In this scenario, the value of the  `cidr_block` and `availability_zone` were hardcoded, and this is not the best practice. This will be resolved by refactoring the coding and making use of variables.

2. I ran the `terraform destroy` command to delete the current infrastructure in preparation for the new one.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/Project16/TerraformDestroy.png)


## STEP 3.  **Refactoring the codes**

1. The variables can be declared either in the `main.tf` file or in a seperate `variables.tf` file, I decided to go with the latter as the `main.tf` can quickly appear clumsy.


2. I started by declaring all the required variables for the VPC in the `varibles.tf` file

![Screenshot](https://github.com/ardamz/my-demo/blob/main/Project16/VPCVariables.png)


I also updated the `main.tf` file accordingly and ran the `terraform apply` command.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/Project16/VPCMain.png)

![Screenshot](https://github.com/ardamz/my-demo/blob/main/Project16/TerraformApply2.png)


3. To prevent hardcoding the `cidr_block`, I had to make use of the `cidrsubnet()` function, I updated the `main.tf` file with the codes below.

```bash
# Get list of availability zones
data "aws_availability_zones" "available" {
  state = "available"
}

# Create public subnet1
resource "aws_subnet" "public" {
  count                   = 2
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]
}
```

4. I ran the `terraform plan` and the `terraform apply` command to preview and create the two (2) subnets specified in the count.

![Screenshot](https://github.com/ardamz/my-demo/blob/main/Project16/SubnetApply.png)


5. To avoid hardcoding the `count` value, I made use  of a variable called `preferred_number_of_public_subnets`.  I added the lines below to the `variables.tf` file.

```bash
  variable "preferred_number_of_public_subnets" {
      default = 7
}
```
I also updated the `count` argument with a condition that allows terraform to check the following conditions
1. first if the `preferred_number_of_public_subnets` in the variables is set to null, the number of AZs is used, this is dervived by the `length(data.aws_availability_zones.available.names)` function, else
2. The `preferred_number_of_public_subnets` defined in the varibales is used.

I ran the `terraform plan` and `terraform apply` commands to create the same infrastructure only this time there is no hardcoded values in my main code, ther ahve all been defined as variables and defined terrin the `variables.tf` file


![Screenshot](https://github.com/ardamz/my-demo/blob/main/Project16/TerraformPlan2.png)


![Screenshot](https://github.com/ardamz/my-demo/blob/main/Project16/TerraformApply3.png)


![Screenshot](https://github.com/ardamz/my-demo/blob/main/Project16/ConsoleView.png)