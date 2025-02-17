# AZURE + TERRAFORM

## Creating a 2-tier web app 

1. Install azure cli:
https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows?tabs=azure-cli
 
2. open a terminal and run:
az login
  - Make sure you restart the terminal (git bash) other az env variable might not show.
 
This should open a browser window for you to log in using active directory.
 
3. Confirm the install is set up with the command:
az

Create a new terraform folder in your github folders.
- Terraform.

## My .tf files:

Using these 2 files, I created an App VM, a DB VM, along with a vnet and 2 subnets (pub+priv).
- I used my custom images previously created to ensure dependencies were already installed and I used a userdata script for the app vm to run automatically without my input. 
- Make sure the db is in the private subnet and the export command passed in the userdata for app vm included the correct private address for the db (for the connection).
- Provided ssh key to be able to ssh into the instances too.


### .tf file to create db vm + private subnet and the NSG for the db
```
resource "azurerm_subnet" "private-subnet" {
  name                 = "my-private-subnet"
  resource_group_name  = "tech501"
  virtual_network_name = azurerm_virtual_network.zainab-vnet.name
  address_prefixes     = ["10.0.3.0/24"]
}

# Define a new Network Security Group (NSG) for DB VM
resource "azurerm_network_security_group" "db-nsg" {
  name                = "db-nsg"
  location            = "UK South"
  resource_group_name = "tech501"
}

# Inbound SSH Rule for DB VM (Port 22)
resource "azurerm_network_security_rule" "db_ssh_rule" {
    resource_group_name = "tech501"
  name                        = "Allow-SSH"
  priority                    = 100
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "22"
  source_address_prefix       = "*"
  destination_address_prefix  = "*"
  network_security_group_name = azurerm_network_security_group.db-nsg.name
}

# Inbound MongoDB Rule for DB VM (Port 27017)
resource "azurerm_network_security_rule" "db_mongodb_rule" {
    resource_group_name = "tech501"
  name                        = "Allow-MongoDB"
  priority                    = 101
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "27017"
  source_address_prefix       = "*"
  destination_address_prefix  = "*"
  network_security_group_name = azurerm_network_security_group.db-nsg.name
}

# Associate NSG with Private Subnet (for DB VM)
resource "azurerm_subnet_network_security_group_association" "nsg_association_db" {
  subnet_id                 = azurerm_subnet.private-subnet.id
  network_security_group_id = azurerm_network_security_group.db-nsg.id
}

# Define Network Interface for DB VM in Private Subnet (no Public IP)
resource "azurerm_network_interface" "db-nic" {
  name                = "zainab-db-nic"
  location            = "UK South"
  resource_group_name = "tech501"

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.private-subnet.id
    private_ip_address_allocation = "Dynamic"
    # No public IP for DB VM
  }
}

# Define Database VM in Private Subnet
resource "azurerm_virtual_machine" "tech501-zainab-db-vm" {
  name                = "tech501-zainab-db-vm"
  resource_group_name = "tech501"
  location            = "UK South"
  vm_size             = "Standard_B1s"

  # Network Interface association
  network_interface_ids = [azurerm_network_interface.db-nic.id]

  os_profile {
    computer_name  = "tech501-zainab-db-vm"
    admin_username = "adminuser"
    custom_data    = base64encode("#!/bin/bash\napt-get update && apt-get install -y mongodb\nsystemctl start mongodb")  # Example script to install MongoDB
  }

  os_profile_linux_config {
    disable_password_authentication = true
    ssh_keys {
      path     = "/home/adminuser/.ssh/authorized_keys"
      key_data = file("~/.ssh/zainab-test-ssh-2.pub")
    }
  }

  # OS Disk configuration
  storage_os_disk {
    name              = "zainab-db-os-disk"
    caching           = "ReadWrite"
    create_option     = "FromImage"
    managed_disk_type = "Standard_LRS"
  }

  # Image reference (your custom image for DB)
   storage_image_reference {
    id = "/subscriptions/cd36dfff-6e85-4164-b64e-b4078a773259/resourceGroups/tech501/providers/Microsoft.Compute/images/tech501-zainab-sparta-db-ready-to-run-img"
  }
  # Tags
  tags = {
    environment = "dev"
  }


}
```


### **Main.tf:**
- Creates an app vm after db vm is created.
```
provider "azurerm" {
  features {}
  resource_provider_registrations = "none"
  subscription_id                 = "cd36dfff-6e85-4164-b64e-b4078a773259"
}


# Virtual Network definition
resource "azurerm_virtual_network" "zainab-vnet" {
  name                = "zainab-my-vnet"
  location            = "UK South"
  resource_group_name = "tech501"
  address_space       = ["10.0.0.0/16"]
}

# Public Subnet definition
resource "azurerm_subnet" "public-subnet" {
  name                 = "my-public-subnet"
  resource_group_name  = "tech501"
  virtual_network_name = azurerm_virtual_network.zainab-vnet.name
  address_prefixes     = ["10.0.2.0/24"]
}

# Network Security Group (NSG)
resource "azurerm_network_security_group" "zainab-nsg" {
  name                = "zainab-nsg"
  location            = "UK South"
  resource_group_name = "tech501"
}

# Inbound SSH Rule (Port 22)
resource "azurerm_network_security_rule" "ssh_rule" {
  resource_group_name         = "tech501"
  name                        = "Allow-SSH"
  priority                    = 100
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "22"
  source_address_prefix       = "*"
  destination_address_prefix  = "*"
  network_security_group_name = azurerm_network_security_group.zainab-nsg.name
}

# Inbound HTTP Rule (Port 80)
resource "azurerm_network_security_rule" "http_rule" {
  resource_group_name         = "tech501"
  name                        = "Allow-HTTP"
  priority                    = 102
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "3000"
  source_address_prefix       = "*"
  destination_address_prefix  = "*"
  network_security_group_name = azurerm_network_security_group.zainab-nsg.name
}

resource "azurerm_network_security_rule" "db_rule" {
  resource_group_name         = "tech501"
  name                        = "Allow-3000"
  priority                    = 101
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "80"
  source_address_prefix       = "*"
  destination_address_prefix  = "*"
  network_security_group_name = azurerm_network_security_group.zainab-nsg.name
}


# Associate NSG with Public Subnet
resource "azurerm_subnet_network_security_group_association" "nsg_association" {
  subnet_id                 = azurerm_subnet.public-subnet.id
  network_security_group_id = azurerm_network_security_group.zainab-nsg.id
}

# Create Public IP
resource "azurerm_public_ip" "my_public_ip" {
  name                = "my-public-ip"
  location            = "UK South"
  resource_group_name = "tech501"
  allocation_method   = "Dynamic" # If you need a static IP, change this to "Static"
  sku                 = "Basic"
}

#locals {
#ssh_public_key = file("~/.ssh/zainab-test-ssh-2.pub")  # Update this path to your public key file
#}


# Network Interface definition for VM in Public Subnet
resource "azurerm_network_interface" "nic" {
  name                = "zainab-nic"
  location            = "UK South"
  resource_group_name = "tech501"

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.public-subnet.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.my_public_ip.id
  }
}

# Virtual Machine definition in Public Subnet
resource "azurerm_virtual_machine" "my-app-vm" {
  name                = "zainab-app-vm"
  resource_group_name = "tech501"
  location            = "UK South"
  vm_size             = "Standard_B1s"

  # Network Interface association
  network_interface_ids = [azurerm_network_interface.nic.id]

  os_profile {
    computer_name  = "zainab-app-vm"
    admin_username = "adminuser"
    custom_data    = base64encode("#!/bin/bash\ncd /repo/app\nexport DB_HOST=mongodb://10.0.3.4:27017/posts\npm2 start app.js") # User data script
  }                                                                                                                             # User data script


  os_profile_linux_config {
    disable_password_authentication = true

    ssh_keys {
      path     = "/home/adminuser/.ssh/authorized_keys"
      key_data = file("~/.ssh/zainab-test-ssh-2.pub")
    }
  }
  # OS Disk configuration
  storage_os_disk {
    name              = "zainab-os-disk"
    caching           = "ReadWrite"
    create_option     = "FromImage"
    managed_disk_type = "Standard_LRS"
  }


  # VM Tags
  tags = {
    environment = "dev"

  }
  storage_image_reference {
    id = "/subscriptions/cd36dfff-6e85-4164-b64e-b4078a773259/resourceGroups/tech501/providers/Microsoft.Compute/images/tech501-zainab-sparta-app-ready-to-run-img"
  }

    # Ensure DB VM is created after App VM
  depends_on = [
    azurerm_virtual_machine.tech501-zainab-db-vm
  ]
}

```



![alt text](<Images/Screenshot 2025-02-13 120941.png>)

![alt text](<Images/Screenshot 2025-02-13 120751.png>)

## State files:
- Need to put them in remote backends to be able to collaborate with other people. 
- AWS - S3+dynamodb (s3 doesnt have file locking by default so have to use dynamodb = more expensive).
- Azure blob storage as a backend - more cheaper and easier as has built in file locking
- file locking - only one person can work on the code at a time.