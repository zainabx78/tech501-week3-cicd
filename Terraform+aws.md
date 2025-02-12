# Terraform

### 2 types of IAC:

1. Configuration management (ansible).
2. Orchestration tool (Terraform).
   - Creates infrastructure in order (e.g. vm gets created last due to other dependencies like sg, vpc being created first).

Benefit - prevents configuration drift (ansible).

## What is Terraform? 
- Terraform is an Infrastructure as Code (IaC) tool that uses HashiCorp Configuration Language (HCL). 
- It is used to manage cloud resources across multiple providers, including AWS, Azure, Google Cloud, Kubernetes, and on-prem environments.
- Orchestration - full lifecycle management of infrastructure.
- Sees infrastructure as disposable (immutable).


ANSIBLE - Best as a config management tool for existing infrastructure not creating it.
- tf - declarative
- ansible - imperative
- Idempotent = doesn't matter how many time you run it, makes sure the result is in your desired state.


### Why use Terraform? The benefits?
- Infrastructure as Code (IaC) – Allows infrastructure to be defined in code, making it version-controlled and repeatable.
- Multi-cloud Support – Works with multiple cloud providers through plugins called providers.
- State Management – Tracks resource states using a state file, enabling efficient updates and avoiding unintended changes.
- Declarative Syntax – Users define the desired state of infrastructure, and Terraform ensures it is achieved.
- Plan and Apply Workflow – The terraform plan command helps preview changes before applying them, reducing errors.
- Modularity and Reusability – Encourages reusable configurations through modules.
- Efficient Resource Orchestration – Automates dependency resolution and parallel execution of resources.

### Alternatives to Terraform
- AWS CloudFormation – AWS-native IaC tool; limited to AWS.
- Pulumi – Uses general-purpose languages (Python, TypeScript, etc.), unlike Terraform’s HCL.
- Ansible – Configuration management and provisioning tool; not stateful like Terraform.
- Chef/Puppet – Focuses on configuration management rather than infrastructure provisioning.
- Google Deployment Manager (GDM) – - Google Cloud-specific IaC tool.

### Who is using Terraform in the industry?
Terraform is widely used by companies for cloud automation, including:

- Netflix – Automates multi-cloud infrastructure.
- Uber – Manages AWS and Kubernetes resources.
- Airbnb – Implements scalable cloud deployments.
- Microsoft, Google, AWS – Terraform is supported as a primary IaC tool for their cloud platforms.

### In IaC, what is orchestration? How does Terraform act as an "orchestrator"?

Orchestration in IaC refers to managing and automating complex infrastructure deployments, ensuring that dependencies and resources are provisioned in the correct order.

Terraform acts as an orchestrator by:

- Dependency Management – Analyzing resource dependencies and determining the correct provisioning order.
- Stateful Management – Tracking resource states and applying changes incrementally.
- Parallel Execution – Creating independent resources concurrently for efficiency.

### Best Practice: Supplying AWS Credentials to Terraform

Terraform requires AWS credentials to manage AWS resources. Best practices include:

- Use IAM Roles for EC2 (preferred) – Avoids storing credentials in local files.
- Use AWS SSO or IAM Roles with OIDC – More secure than static credentials.
- Use environment variables (AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY) – Secure but still stored on the machine.
- Use AWS profiles in ~/.aws/credentials and ~/.aws/config – Organizes credentials securely.
- Use Terraform Cloud or Vault for Secrets Management – Enhances security.
Order of Precedence for AWS Credentials in Terraform

### Terraform looks up AWS credentials in the following order:

1. Environment variables (AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_SESSION_TOKEN).
2. Shared credentials file (~/.aws/credentials).
3. AWS config file (~/.aws/config).
4. IAM role attached to an EC2 instance (via instance profile).
5. IAM roles for Service Accounts (IRSA) in Kubernetes (OIDC authentication).
6. Terraform Cloud variables (if using Terraform Cloud).
The method highest in the list takes precedence.

### How Should AWS Credentials Never Be Passed to Terraform?

- Never Hardcode AWS credentials in Terraform files (.tf files) – Avoid committing secrets to version control.
- Never store credentials in Terraform state files (terraform.tfstate) – State files should be encrypted or stored securely (e.g., in Terraform Cloud or AWS S3 with encryption).
- Never pass credentials as command-line arguments – They can be exposed in logs.

### Why Use Terraform for Different Environments (e.g., Production, Testing, etc.)?
- Consistency – Ensures that infrastructure is identical across development, testing, and production.
- Isolation – Different environments can have separate state files to avoid conflicts.
- Scalability – Easily spin up and tear down environments as needed.
- Security – Different AWS IAM roles and - permissions per environment improve security.
- Versioning – Each environment can have different configurations tracked in version control.


# To install the latest version: 
- First ran `where terraform` to see where my old version was installed.
- Deleted old terraform files. 
- Installed new version from google.
- Added path variable in git bash terminal:
`export PATH="/c/Users/zaina/Terraform:$PATH"`
- `terraform version`
  - `Terraform v1.10.5
    on windows_amd64`

Need to store private key for terraform in environment variables (in memory so terraform can access aws).
Also save them in .ssh folder

- Moved my keys from downloads to my .ssh folder ` mv /c/Users/zaina/Downloads/tech501-zainab_accessKeys.csv /c/Users/zaina/.ssh` 
- Add the key access ID and secret access key to the system variables:

![alt text](<Images/Screenshot 2025-02-11 121319.png>)

- If you open a new git bash terminal and run `printenv` you should see the new env variables.

# LAB

![alt text](<Images/Screenshot 2025-02-11 150549.png>)

- Create a new repo in git bash terminal
   - cd Documents/github
   - mkdir tech501-terraform
   - cd tech501-terraform.
   - Open in vscode - code . 
- Create a new file `main.tf`
   - Uses HCL language.
- Don't push state files to public github repo - sensitive info.
- Lock file created when `terraform init` 
  - This command creates the setup/dependencies needed for terraform in that folder.
  - Open bash terminal in vscode.
  - Can share when collaborating.
  - Uses same version of plugins/providers.

**Script to create an ec2 instance**

```
# Create an ec2 instance  

# where to create it - provide cloud name
provider "aws" {

  # which region to use
  region = "eu-west-1"
}

# which services/resources
resource "aws_instance" "app_instance" {

  # what AMI ID
  ami = "ami-0c1c30571d2dae5c9"

  # which type of instance
  instance_type = "t3.micro"

  # public Ip
  associate_public_ip_address = true

  # name the isntance
  tags = {
    Name = "tech501-zainab-terraform-app"
  }
}
```
- `terraform fmt` command can help with formatting.
- `terraform plan`
  - Can use -out to save plan to a file and use that plan as a reference.
- `terraform apply`

![alt text](<Images/Screenshot 2025-02-11 150926.png>)

- Don't need credentials because already stored in environment variables.
  
- The files + hidden files after terraform apply.

![alt text](<Images/Screenshot 2025-02-11 151719.png>)

- Run `terraform destroy` so you can go back and add a key to ssh into the instance.
  
![alt text](<Images/Screenshot 2025-02-11 152034.png>)

## Initialise your git repo and commit without sensitive files
- git init
- git branch -M main
- Don't wanna push sensitive info so put those files in a gitignore file:
  - Create a file in vscode in the tf folder for .gitignore
  - Enter this in the .gitignore file:
```
# doesnt contain sensitive files
#contains provider plugins, module cache
#excluded to avoid bloating the repo
.terraform/

#the most important files to secure as they contain credentials
terraform.tfstate
terraform.tfstate.backup
```
- In the terminal:
  - `git add .`
  - `git status`
    - Shouldnt see the tf state files.
  - `git commit -m ""`

## Adding a security group and key pair (existing) to the terraform script:

```
# Create an ec2 instance  

# where to create it - provide cloud name
provider "aws" {

  # which region to use
  region = "eu-west-1"
}

# which services/resources
resource "aws_instance" "app_instance" {

  # what AMI ID
  ami = "ami-0c1c30571d2dae5c9"

  # which type of instance
  instance_type = "t3.micro"

  # public Ip
  associate_public_ip_address = true

  # security group
  vpc_security_group_ids = [aws_security_group.tech501-zainab-tf-allow-port-22-3000-80.id]
  
  # key pair
  key_name = aws_key_pair.zainab-key-tf.id

  # name the isntance
  tags = {
    Name = "tech501-zainab-terraform-app"
  }

}

# Creating the security group

resource "aws_security_group" "tech501-zainab-tf-allow-port-22-3000-80" {
    name        = "tech501-zainab-tf-allow-port-22-3000-80"
    description = "security group"

    tags = {
        Name = "tech501-zainab-tf-allow-port-22-3000-80"
    }
}

resource "aws_vpc_security_group_ingress_rule" "allow-ssh" {
  security_group_id = aws_security_group.tech501-zainab-tf-allow-port-22-3000-80.id
  from_port         = 22
  ip_protocol       = "tcp"
  to_port           = 22
  cidr_ipv4  = "88.97.161.118/32"
}

resource "aws_vpc_security_group_ingress_rule" "allow-http" {
  security_group_id = aws_security_group.tech501-zainab-tf-allow-port-22-3000-80.id
  from_port         = 80
  ip_protocol       = "tcp"
  to_port           = 80
  cidr_ipv4 =   "0.0.0.0/0"


}

resource "aws_vpc_security_group_ingress_rule" "allow-3000" {
  security_group_id = aws_security_group.tech501-zainab-tf-allow-port-22-3000-80.id
  from_port         = 3000
  ip_protocol       = "tcp"
  to_port           = 3000
  cidr_ipv4 =   "0.0.0.0/0"

}

# Creating a key pair for ec2
resource "aws_key_pair" "zainab-key-tf" {
  key_name   = "zainab-tf-my-existing-keypair-"
  public_key = file("~/.ssh/zainab-test-ssh-2.pub")  # Replace with your actual public key path
}

```

- Run `terraform apply`.
- Also SSH in using the public IP of the instance:
  - `ssh -i /c/Users/zaina/.ssh/zainab-test-ssh-2 ubuntu@34.246.163.39`


![alt text](<Images/Screenshot 2025-02-11 164417.png>)

## Variables

- Don't repeat yourself in the terraform code - variables are good.
- Easier to change values when they're set in variables.
- Easy to put into the variables in .gitignore file so the values don't get pushed onto a github repo.
- Abstraction - abstracts some complexity from the code - easier to read and use and understand.

1. Create a file called variable.tf.
- Add the AMI ID into the file as a variable and then input that variable into the main.tf file.

![alt text](<Images/Screenshot 2025-02-12 105911.png>)

In the main.tf file:

![alt text](<Images/Screenshot 2025-02-12 105944.png>)

- Add the variable.tf file to the .gitignore file so it doesn't get pushed to any git repo. 
- Set the rest of the hardcoded values as variables too.

- Id's aren't allowed in variables.
  
![alt text](<Images/Screenshot 2025-02-12 111617.png>)

### Variables in main.tf:

```
# Create an ec2 instance  

# where to create it - provide cloud name
provider "aws" {

  # which region to use
  region = var.region
}

# which services/resources
resource "aws_instance" "app_instance" {

    # what AMI ID
  ami = var.ami_id

  # which type of instance
  instance_type = var.instance_type

  # public Ip
  associate_public_ip_address = true

  # security group
  vpc_security_group_ids = [aws_security_group.tech501-zainab-tf-allow-port-22-3000-80.id]
  
  # key pair
  key_name = aws_key_pair.zainab-key-tf.id

  # name the isntance
  tags = {
    Name = var.instance_name
  }

}

# Creating the security group

resource "aws_security_group" "tech501-zainab-tf-allow-port-22-3000-80" {
    name        = var.security_group_name
    description = "security group"

    tags = {
        Name = var.security_group_name
    }
}

resource "aws_vpc_security_group_ingress_rule" "allow-ssh" {
  security_group_id = aws_security_group.tech501-zainab-tf-allow-port-22-3000-80.id
  from_port         = 22
  ip_protocol       = "tcp"
  to_port           = 22
  cidr_ipv4  = var.my-ip
}

resource "aws_vpc_security_group_ingress_rule" "allow-http" {
  security_group_id = aws_security_group.tech501-zainab-tf-allow-port-22-3000-80.id
  from_port         = 80
  ip_protocol       = "tcp"
  to_port           = 80
  cidr_ipv4 =  var.open-access-ip


}

resource "aws_vpc_security_group_ingress_rule" "allow-3000" {
  security_group_id = aws_security_group.tech501-zainab-tf-allow-port-22-3000-80.id
  from_port         = 3000
  ip_protocol       = "tcp"
  to_port           = 3000
  cidr_ipv4 =   var.open-access-ip

}

# Creating a key pair for ec2
resource "aws_key_pair" "zainab-key-tf" {
  key_name   = var.key_name
  public_key = file(var.key_file)  # Replace with your actual public key path
}

```

### Variables set in the variable.tf file:

```
# Create a variable for the ami-id

variable "ami_id" {
  default = "ami-0c1c30571d2dae5c9"

}

variable "region" {
  default = "eu-west-1"
}

variable "instance_type" {
  default = "t3.micro"
}

variable "instance_name" {
  default = "tech501-zainab-terraform-app"
}

variable "key_name" {
  default = "zainab-tf-my-existing-keypair"
}

variable "key_file" {
  default = "~/.ssh/zainab-test-ssh-2.pub"
}

variable "open-access-ip" {
  default = "0.0.0.0/0"
}

variable "my-ip" {
  default = "88.97.161.118/32"
}

variable "security_group_name" {
  default = "tech501-zainab-tf-allow-port-22-3000-80"
}

```
Push the repo to new github repo called terraform