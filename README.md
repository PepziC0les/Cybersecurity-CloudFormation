# CloudFormation: ELK + DVWA
<!-- markdownlint-configure-file { "MD004": { "style": "consistent" } } -->
<!-- markdownlint-disable MD033 -->
<p align="center">
    <img src="./image.png" width="685" height="300" alt="Pi-hole">
    <br>
</p>
<!-- markdownlint-enable MD033 -->

# Purpose of the CloudFormation
The CloudFormation built here demonstrates creating a basic network on AWS that utilizes DVWA and ELK servers for testing, practicing, and learning purposes. 
<p align="center">
    <img src="./NetworkDiagram2.png">
    <br>
</p>

Shown here is a diagram depicting how the network is configured where we have our instances that we can connect to over the Internet in the "Public1" subnet, and our instances that we don't want accessed in our "Private1" subnet (Our ELK server and DVWA server). 

We also

Our subnets are also shown to be encapsulated in the diagram by the Availbility Zone used to host our virtual network. All connections are faciliated via the Internet Gateway, which is found on VPC1 and is connected to our "Public" subnets.


# Understanding the Repository
This repository is for helping and practicing setting up a basic cloud network on AWS to host ELK and DVWA servers. Other things to setup will be MetricBeat and FileBeat to be used on our ELK server. The primary files to be used are daemon.json and the files located in the "./Configs+Playbooks" folder. You can follow the diagram above to get a visual understanding of how the network will look like in the end. 

To start setting up your own basic network with instances, start off on the section, **"Creating our Cloud Stack"**. Otherwise if you're already ahead, look for the specific section that you feel best fits where you're at.

# Creating our Cloud Stack
Login into your AWS console and search for "CloudFormation"
Select "create stack" and do the following:
1. Select "Upload a template" Upload "Basic_Network_Cloud_Formation.yaml" (not .yml)
2. Select next and name your stack
3. Select next until you can launch your instance

Before setting up our instances, we want to modify our VPC configurations
1. Search for VPC in the search bar and select VPC, then look for subnets and select subnets "Public1" and "Public2"
2. Select the dropdown called "Actions" and select "Modify auto-assign IP settings". Once enside select the checkbox to enable auto-assign public IPv4 address.

# Setting up our Amazon Instances
Search for EC2 and select "Instances", and then create the following instances by selecting "Launch Instances":
1. Amazon Linux 2 AMI (HVM), SSD Volume Type
- Only need 1 Amazon Linux.
- Click next and select t2.micro
- Click Next and modify network and subnet:
  - Network: VPC1
  - Subnet: Public1
- Select next until you have reached "Configure Security Group"
  - Modify "Security Group Name" and "Description" and give it something meaningful (e.g. a meaningful name and description)
  - Should only have SSH as a rule
- When prompted for key, create one if you do not have one. Creating keys allows you to download it onto Desktop for reuse on other AWS instances. Otherwise, choose existing.
- Select Review and Launch

2. Ubuntu Server 20.04 LTS (HVM), SSD Volume Type
- Need 3 Amazon Linux.
- For first 2 (Our DVWA servers):
  - Click next and select t2.micro
  - Click Next and modify network and subnet:
    - Set number of instances to 2
    - Network: VPC1
    - Subnet: Private1
  - Select next until you have reached "Configure Security Group"
    - Modify "Security Group Name" and "Description" and give it something meaningful (e.g. a meaningful name and description)
    - Add the following rules: "HTTP"
      - HTTP Port should default to 80. Change "Source" to be "Anywhere" instead of "Custom"
        - Note: This is bad practice to set source to anywhere. Only done here for testing purposes  
  - When prompted for key, create one if you do not have one. Creating keys allows you to download it onto Desktop for reuse on other AWS instances. Otherwise, choose existing.
  - Select Review and Launch
- For last 1 (Our ELK server):
  - Click next and select t3.medium
  - Click Next and modify network and subnet:
    - Network: VPC1
    - Subnet: Private1
  - Select next until you have reached "Configure Security Group"
    - Modify "Security Group Name" and "Description" and give it something meaningful (e.g. a meaningful name and description)
      - Note: We will be reusing this
    - Add the following rules: "HTTP", "TCP Custom" (need 5 TCP customs)
      - HTTP Port should default to 80. Change "Source" to be "Anywhere" instead of "Custom"
      - Custom TCP Ports should be 5044, 9600, 5601, 9300, and 9200. Change "Source" to be "Anywhere" instead of "Custom"
  - When prompted for key, create one if you do not have one. Creating keys allows you to download it onto Desktop for reuse on other AWS instances. Otherwise, choose existing.
  - Select Review and Launch

3. Microsoft Windows Server 2019 Base
- Only need 1 Windows instance.
- Click next and select t2.micro
- Click Next and modify network and subnet:
  - Network: VPC1
  - Subnet: Public1
- Select next until you have reached "Configure Security Group"
  - Modify "Security Group Name" and "Description" and give it something meaningful (e.g. a meaningful name and description)
  - Should only have RDP as a rule
- When prompted for key, create one if you do not have one. Creating keys allows you to download it onto Desktop for reuse on other AWS instances. Otherwise, choose existing.
- Select Review and Launch

# Creating the Load Balancer
On the EC2 page, look for load balancers and once found, select create load balancer.
1. Select "Create" for the "Application Load Balancer"
2. Give our load balancer a meaningful name and set to interal, 
3. Select VPC1 so that we have access to our availability zones (should be 2 that appears) and select them.
- Make sure that the availability zones are set to Private1 and Private2 (the location of our DVWA servers)
4. Select next until we reach configure security groups
- Here we want to create a new security group for our load balancer where we use HTTP and SSH (similar to secuirty configurations for our DVWA instances)
5. Select next and now we need to create a new target group if none have been made yet
- Target group should be new target group
- Set name to be something meaningful
6. Select next and now look for our DVWA servers under "Name".
- Select tthe checkboxes to the left for our DVWAs and select "add to registered"
- Note: If you already have a target group set for our DVWAs, skip step 6.
7. Select next until you see "Create" and then select create.

# Setting up our Instances for Deployment
We want to connect to our instances now. Go back to the instances page and select "Connect" for our Jumpbox and copy the ssh command.
- Open a Gitbash/Command Prompt terminal and change directories into the directory where your key(s) are/is stored.
- Paste the ssh command and hit enter. The command should fit in the outline of the following:
```bash
ssh -i <key> <user>@<destination>
```
- We want to transfer all our important configuration + key files (key files meaning keys) before moving on. To do that, it's recommended to open at least 1 more terminal (Gitbash or Command Prompt). The following command will allow you to transfer 1 or more files:
```bash
scp -i <key> <file name(s)> <user>@<destination>:/path/to/home/directory
Ex: scp -i Virginia.pem Virginia.pem ansible_config.yml thisuser@ec2.amazon.someinstance:/home/ec2-user
```
- ssh into each of your private Ubuntu instances using your keys and their private IPv4 addresses and update/upgrade them using the following commands:
```bash
sudo apt-get update
sudo apt-get upgrade
```
- To get out of a machine, type exit into the terminal.
- Once all Ubuntu instances have been updated, we want to be back inside our Jumpbox.
- Inside the Jumpbox, type the following command to install our "docker", which will be used to host and run our "Ansible" image:
```bash
sudo yum install docker -y
```
- Once the docker has been installed, we need to start the service. Before that, we should make a "daemon.json" so that our Ansible process defaults its address to our networks subnet. To do this, use the following command:
```bash
sudo nano /etc/docker/daemon.json
```
- Copy the content of daemon.json in this git repository and paste it into your daemon.json. Once done, save and exit.
- Now we must run our docker service which is done so by the following:
```bash
sudo service docker start
```
- To check if the service is running, you can do "sudo service docker status"
- Follow it with the next command which pulls the ansible image (in other words, downloads it into our Jumpbox):
```bash
sudo docker image pull cyberxsecurity/ansible
```
- Now we want to run the ansible image, which can be done by doing running the following command:
```bash
sudo docker run -ti cyberxsecurity/ansible bash
```
- If you had closed your other terminals, be sure to open at least 1 more and ssh into your Jumpbox once more (NOTE: do not close the terminal where you can see the running ansible process. If done, wait until we get to the part where we finish discussing getting the process ID).
- Inside the Jumpbox, we want to look for our process ID of the running Ansible process. To do that use the following command:
```bash
sudo docker ps
```
- Copy the value underneath process ID by highlighting it and keep that saved. If you had mistakingly exited or stopped the Ansible process, you can either reuse the "sudo docker run -ti cyberxsecurity/ansible bash" again if you did not see a process ID, or run "sudo docker attach <process ID>" to reenter the process.
- We want to use the process ID to transfer our files (keys + configuration files) to our Ansible process. We can only trasnfer files 1 at a time using the following command:
```bash
sudo docker cp <file> <process ID>:/root
Ex: sudo docker cp Key1.pem ad314a9d:/root
```
- Repeat the command above with each file in your Jumpbox.
- Go back to your Ansible process now. We have to modify our /etc/ansible/hosts and /etc/ansible/ansible.cfg files so that they can communicate with our Ubuntu instances. To do so nano both of those files (no need to add sudo at the beginning).
    - Inside the hosts file, look for '[webservers]' and remove the '#' symbols before it. Now add the private IPv4 addresses of your DVWA instances below it
    - Still inside the hosts file, add '[elkservers]' below the IPv4 addresses you had just added, and now add the IPv4 address of your ELK machine beneath '[elkservers]'
    - Save your changes and exit, and now nano /etc/ansible/ansible.cfg.
    - Inside ansible.cfg, look for "remote_user = root" and change the variable to equal "ubuntu".
    - Save and exit out of ansible.cfg.
- Back inside our Ansible process, we must ssh into our private instances and then exit out of them (this is so that our current Ansible can establish connections later on again).
- Once sshing into each of the private instances is done, run the following command to begin setting up our DVWA machines:
```bash
ansible-playbook ansible_config.yml --key-file=<key>
```
- We do the same to setup our ELK instance, where we switch out the ansible_config.yml with "install_elk.yml".

# Running our Servers
- Before running our servers, we to create a daemon.json for each of our servers. Do the same as we did when we created our first daemon.json by doing "sudo nano /etc/docker/daemon.json" in each of them, then copy the content of daemon.json in the git repository into daemon.jsons of the instances.
    - NOTE: If any of docker processes are running on any of our private instances, stop them since we will be restarting docker. You can check for a running process by doing "sudo docker ps" and if you do find one that is running, kill it by doing "sudo docker kill <process ID>"
- Restart the docker service once finished setting up our daemon.json file on each of the instances by doing the following:
```bash
sudo service docker restart
```
- You can confirm it is running by doing "sudo service docker status"
- Now we want to run our DVWA and ELK machines.
- On the DVWA machines, run the following command:
```bash
sudo docker run -ti cyberxsecurity/dvwa
```
- On the ELK machine, run the following command:
```bash
sudo docker run -p 5601:5601 -p 9200:9200 -p 9300:9300 -p 9600:9600 -p 5044:5044 sebp/elk
```

# Testing our Servers

# Setting up Metricbeat and Filebeat

# Possible Mishaps
