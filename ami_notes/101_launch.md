## My personal notes for launching the KevinCrook.com 101 AWS AMI

**Purpose:**  AMI to provide Ubuntu with Docker so students can launch an AWI image of any size and run Docker images into containers or clusters.  

### AMI Details

Please make sure you are in the right region.  AMIs are stored in a region.  You can search on the AMI name, but always verify the AMI ID before launching.  Anyone can make an image and call it anything, so someone else might create a KevinCrook.com AMI with malicious content.

**Region:** N. Virgina 

**Name:** KevinCrook.com 101

**AMI ID:** ami-019c497a921bc29bd

**Storage:** 100 GiB, approximately $10 a month with August 2018 pricing to store

### Sizing the instance

Recommended Minimum: 

t2.large - 2 CPUs, 8 GiB RAM, **9 cents an hour** current pricing when running, you are only charged for storage when it's not running

Remember one of the advanges of AWS is that you can launch much larger images if you need more CPUs and more memory such as 16 GiB, 32 GiB, 64 GiB, etc.

### Users

**root** - the linux superuser.  Ubuntu prefers the method of using login named ubuntu (see below) and doing anything that requires root privileges with the sudo command.

**ubuntu** - the default user.  This is the account that AWS will create the ~/.ssh/authorized_keys file for.  You will need to use your key to login to this account.

**science** - an account I created to match the W205 droplet science account.  If you want to use this account, you will need the do the following that I was unable to do in the AMI for security reasons:

* Set a password for the science account from the ubuntu account using sudo.  After this, you can login to the science account using the password you just set.

```
sudo passwd science
```

* Copy the ~/.ssh/authorized_keys file from the ubuntu account to the science account and set ownership and permissions.  After this, you can login using your same private key that AWS gave you for the ubuntu account.  

```
sudo mkdir /home/science/.ssh

sudo cp /home/ubuntu/.ssh/authorized_keys /home/science/.ssh/authorized_keys

sudo chown -R science:science /home/science/.ssh
```

