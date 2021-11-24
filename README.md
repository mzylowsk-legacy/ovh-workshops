# Workshops 25.11.2021
![logo](./img/logo.png)

#### 1. Walk through manager
- Introduction
- Payment methods
- Public cloud project creation

#### 2. First instance
- SSH key generation
- Spawning first instance (S1-2, public IP only)
- Docker installation by cloud-init
```
#include https://get.docker.com/
```
- Logging to VM via SSH
- Starting a httpd docker container with custom text:
```
sudo docker run -d -p 80:80 --name www-server httpd && sudo docker exec www-server sh -c "echo 'Hello from OVH workshops :)' > htdocs/index.html"
``` 
- Check that connectivity via internet is working

#### 3. Create and attach volume
- Create 10GB volume in manager
- Attach the volume to VM spawned in previous exercise
- Check that volume is accessible as block device:
```
lsblk
```
- For new volumes, creating a file system is needed:
```
mkfs.ext4 /dev/sdX
```
- When filesystem is created, volume can be mount somewhere:
```
sudo mkdir /mnt/demo-volume
sudo mount /dev/sdX /mnt/demo-volume
sudo echo "<h1>Hi!</h1>" > /mnt/demo-volume/index.html
```

#### 4. Multiple instances in different regions connected by one private network
- Configure private network (vRack) in manager
- Create a set of instances (VM's) in WAW1 region
- Create a set of instances (VM's) in GRA9 region
- Ensure that there is connectivity over private network between them (ping)
- Check routes (and number of hops) between public IP and private IP (mtr)

#### 5. Horizon and security group change
- Create a user for Public Cloud in manger
- Use created credentials to login into horizon
- Check security group rules
- Disable connectivity by public IP for VM's in WAW1 (except SSH):
    * Remove general IPv4 ingress rule that allows everything (at this point SSH connectivity will be lost)
    * Add a rule that allows SSH traffic (Ingress IPv4, TCP, port 22)
    * Check connectivity

#### 6. Resize a VM (change model (flavor) of a instance) by openstack CLI:
- Install openstack cli client:
```
pip3 install python-openstackclient
```
- Download and source openrc file from ovh manager
```
source openrc.sh
```
- Perform resize on chosen VM:
```
openstack server resize [VM_id] --flavor [FLAVOR NAME]
```
(Flavors can be listed by):
```
openstack flavor list
```
