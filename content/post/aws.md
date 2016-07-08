+++
date = "2016-05-28T11:52:55-05:00"
tags = []
title = "aws"

+++

- Sign up for Amazon AWS:
https://portal.aws.amazon.com/gp/aws/developer/registration/index.html
 Fill out the basic info
 On the next page, be sure to click the Personal Account option:
 ![aws1](/images/aws1.png)
- Go to Services > EC2 > Launch Instance
 ![launch](/images/launch.png)
- Click Community AMIs on the left
- Search for the latest FreeBSD "RELEASE" version, currently FreeBSD 10.3-RELEASE-amd64 - ami-dcd239bc
- Choose the Free tier elligible t2.micro instance
 ![t2-micro](/images/t2-micro.png)
- Hit Next: Configure Instance Details
 ![next](/images/next.png)
- Scroll down and expand the Advanced Details. Paste this user-data to have your vm configured for root login, update all packages on first boot, and install Python which will be a requirement for Ansible:
  ```
  #!/bin/sh
  cat << EOF > etc/rc.conf
  ec2_configinit_enable="YES"
  ec2_fetchkey_enable="YES"
  ec2_fetchkey_user="root"
  ec2_bootmail_enable="NO"
  ec2_ephemeralswap_enable="YES"
  ec2_loghostkey_enable="YES"
  ifconfig_xn0="DHCP"
  firstboot_freebsd_update_enable="YES"
  firstboot_pkgs_enable="YES"
  firstboot_pkgs_list="python27"
  dumpdev="AUTO"
  panicmail_enable="NO"
  panicmail_autosubmit="NO"
  sshd_enable="YES"
  EOF

  cat << EOF > /etc/ssh/sshd_config
  Port 22
  ListenAddress 0.0.0.0
  Subsystem sftp /usr/libexec/sftp-server
  PermitRootLogin without-password
  UseDNS no
  EOF
  ```
- Next: Add Storage
- You get 30GB with the free tier so might as well use it. Change the Size(GB) to 30
 ![size](/images/size.png)
- Hit Next: Tag Instance, which you can skip.
- Next: Configure Security Group
- Your vm will be behind an aws firewall which by default will not let any traffic in but SSH. This is great, and secure by default but we need to allow traffic for the server to be useful. Allow HTTP, HTTPS, IMAP, and SMTP.
 ![security](/images/security.png)
- Click Review and Launch to make sure everything looks good. Don't worry about the warning up top, your server is as secure as it needs to be.
- Click Launch. Now you'll need to setup SSH access to your vm. Click create a new key pair and name your key "aws" without quotes of course, then click Download Key Pair. Save the key to your ~/.ssh folder (create it if it does not exist yet). You should now have an aws.pem file in your .ssh folder. ![key](/images/key.png)
- Now aws should be setting up your instance which can take around 5 minutes. In the mean time, edit your local ssh config so it will be easy to log into the new server. Replace the HostName info below with your Public DNS info aws provides. If you need to find this, go back to Services > Compute > EC2 > Instances on the left.
 ```
 nano ~/.ssh/config

 Host aws
   HostName ec2-127-0-0-1.us-west-2.compute.amazonaws.com
   IdentityFile ~/.ssh/aws.pem
   User root
 ```
- Once your instance says 2/2 Passed in the Status Checks Column, you should be able to login now.
 ```
 ssh aws
 ```
