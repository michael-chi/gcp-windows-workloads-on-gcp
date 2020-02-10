##  Overview

Ansible Windows ASP.Net deployment notes

-   [Step by Step](https://codingnote.cc/zh-tw/p/20574)
-   [Install Ansible Server on Ubuntu](https://www.techrepublic.com/article/how-to-install-ansible-on-ubuntu-server-18-04/)
-   [IT鐵人賽](https://ithelp.ithome.com.tw/articles/10184670)
-   [How to manage Windows Server 2016 with Ansible](http://nokitel.im/index.php/2016/11/09/how-to-manage-windows-server-2016-with-ansible/)
-   [How to manage Windows Server 2016 with Ansible and Active Directory](http://nokitel.im/index.php/2016/11/10/how-to-manage-windows-server-2016-with-ansible-active-directory/)

###    Step by Step

####    Ansible Server Installation

-   Create a Compute Instance on GCP

-   Install Ansible Server

```shell
##  Update
sudo apt-get update
sudo apt-get upgrade -y

##  Add repository required
sudo apt-get install software-properties-common
sudo apt install dirmngr
sudo apt-get update

##  Solve [gpg: no valid OpenPGP data found] error
##  I still get same error after below steps, but than I can still successfully install ansible
curl -O https://packages.cloud.google.com/apt/doc/apt-key.gpg
sudo apt-key add apt-key.gpg
curl -s https://updates.signal.org/desktop/apt/keys.asc | sudo apt-key add -


sudo apt-add-repository ppa:ansible/ansible

##  Update
sudo apt-get update

##  Install Ansible
sudo apt-get install ansible -y

##  Install Python
sudo apt-get install python -y

##  Install Pip
sudo apt-get install python-pip -y
sudo pip install --upgrade pip

##  Install PyWinRM
pip install pywinrm

```

-   Edit hosts file to add hosts

```shell
sudo nano /etc/ansible/hosts
```

```ini
##  Add below lines
[win]
172.16.2.5 
172.16.2.6 

[win:vars]
ansible_user=<local machine or ad user>
ansible_password=<password>
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore
```

-   [Configure managed Windows Nodes](./managed-node-setup.md)

-   Test connection

```shell
ansible win -i /etc/ansible/hosts -m win_ping
# 10.140.15.233 | SUCCESS => {
#     "changed": false, 
#     "ping": "pong"
# }
```