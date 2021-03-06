## My personal notes for creating the KevinCrook.com 101 AWS AMI

**Purpose:**  AMI to provide Ubuntu with Docker so students can launch an AWI image of any size and run Docker images into containers or clusters.  

### Ubuntu Server

As of August 2018, when this was made, AWS was only supporting Ubuntu Server 16.04 LTS "Xenial".  
(note: 18.04 LTS "Bionic" has been out for 4 months, but not yet certified for AWS.)

Here is the image used:

Ubuntu Server 16.04 LTS (HVM), SSD Volume Type - ami-759bc50a

Launched details:
* N. Virginia Region
* t2.large - 2 CPUs, 8 GiB RAM, 9 cents an hour current pricing
* 100 GiB storage, $10 a month current pricing
* security Group I named 101, open ports: 22 (ssh), TBD

### Update packages and upgrade packages and reboot

```
sudo apt-get update

sudo apt-get -y upgrade

sudo reboot
```

### Docker CE (community edition) install

The Docker website provides the following web page of instructions.  I followed the Section "Install Docker CE" subsection "Install using the repository"

https://docs.docker.com/install/linux/docker-ce/ubuntu/

These are the specific commands I ran from the above link:
```
sudo apt-get remove docker docker-engine docker.io

sudo apt-get update

sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
    
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
(be sure and include the dash at the end)

sudo apt-key fingerprint 0EBFCD88

verify it looks like this:

pub   4096R/0EBFCD88 2017-02-22
      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid                  Docker Release (CE deb) <docker@docker.com>
sub   4096R/F273FCD8 2017-02-22

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
   
sudo apt-get update

sudo apt-get install docker-ce
```

The instructions above did not include the instruction to install docker-compose:

```
sudo apt-get install -y docker-compose
```

After the install instructions, they have the post install instructions, which are mainly to add the user ubuntu to the docker group so we don't have to run every docker command as sudo:

https://docs.docker.com/install/linux/linux-postinstall/

These are the specific commands I ran from the above link:
```
sudo groupadd docker

sudo usermod -aG docker $USER

If you get errors on permissions from running docker with sudo before adding it to the group, you may need to do these:

sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
sudo chmod g+rwx "/home/$USER/.docker" -R

Docker was automatically enabled, but if for some reason it isn't enabled, you can enable it with this:

sudo systemctl enable docker
```

### Creating the user "science" used in MIDS W205 and add to sudo and docker groups

```
sudo adduser science

sudo usermod -aG sudo science

sudo usermod -aG docker science
```

### Change the sudo setup to not require password for members of sudo group

```
sudo visudo
```

Change the following to add the NOPASSWD: 

```
# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) NOPASSWD: ALL
```

### Enable ssh without login for science

You will need to do this after you create the instance in AWS:

```
sudo mkdir /home/science/.ssh

sudo cp /home/ubuntu/.ssh/authorized_keys /home/science/.ssh/authorized_keys

sudo chown -R science:science /home/science/.ssh
```


