### How to create ssh key through terraform

<!-- ####### ======== creating private key -->
resource "tls_private_key" "linuxkey" {
  algorithm = "RSA"
  rsa_bits  = 4096
}

<!-- ####### ======== Getting the content of the private key -->
resource "local_file" "linuxpemkey" {
  filename   = "linuxkey.pem"
  content    = tls_private_key.linuxkey.private_key_pem
  depends_on = [tls_private_key.linuxkey, ]

}

<!-- ####### ======== Creating Linux VM -->
resource "azurerm_linux_virtual_machine" "linuxvm" {
  name                = "linuxvm"
  resource_group_name = data.azurerm_resource_group.appgrp.name
  location            = data.azurerm_resource_group.appgrp.location
  size                = "Standard_D2s_v3"
  admin_username      = "linuxuser"
  network_interface_ids = [
    azurerm_network_interface.appinterface.id,
  ]
  admin_ssh_key {
    username   = "linuxuser"
    public_key = tls_private_key.linuxkey.public_key_openssh

  }

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-focal"
    sku       = "20_04-lts"
    version   = "latest"
  }
  depends_on = [azurerm_network_interface.appinterface, data.azurerm_resource_group.appgrp, tls_private_key.linuxkey]
}

#### Steps to follow once the machine is created
1. right click on the generated ssh key
2. security 
3. advanced
4. remove all inheritance
5. add new user
6. give full access
7. connect to the machine
   1. ssh -i .\linuxkey.pem linuxuser@public_ip address