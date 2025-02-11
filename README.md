1. Provision AWS instance with

- amazon linux 2
- 100GB gp3
- use existing keys
- use security group that has 80/443 access
- use role with ssm and s3 access (added s3 read-only to ecs-instance-profile)

2. provision route53 dns or use existing for both traefik (CNAME) and llm (A Record) access or use existing

3. confirm connect to instance terminal through aws console ssm

# On instance

## NB all setup is being done for ssm-user. So, `sudo su -l ssm-user` to switch from ec2-user if you're ssh'ing in.

1. confirm s3 access for NVidia driver download
   `aws s3 ls`

2. install gcc and reboot - needed for compiling nvidia drivers

```
sudo yum update -y && sudo yum install -y gcc make
sudo reboot
```

3. download the nvidia drivers
   this is why we needed the s3 read-only access

```
# Remove existing drivers if present
sudo yum remove nvidia* -y

# Install prerequisites
sudo yum groupinstall "Development Tools" -y
sudo yum install kernel-devel-$(uname -r) kernel-headers-$(uname -r)  kernel-modules-extra.x86_64 vulkan-loader -y

# Download and install Tesla driver
# can check for latest version here:  https://www.nvidia.com/en-us/drivers/
cd
wget https://us.download.nvidia.com/tesla/570.86.15/NVIDIA-Linux-x86_64-570.86.15.run
sudo sh NVIDIA-Linux-x86_64-570.86.15.run --silent --dkms

# Verify installation
nvidia-smi
```

4. clone the Thrive Ollama git repo
```
sudo yum install -y git
mkdir -p /home/ssm-user/dev
cd /home/ssm-user/dev
git clone https://github.com/CareerJSM/test-docker-compose-llm.git
```

5. Install Docker and Docker Compose
```
sudo yum install docker -y
sudo service docker start
sudo chkconfig docker on
# Install docker-compose
sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
# Verify success
docker-compose version
# Add users to docker group
sudo usermod -aG docker ssm-user
sudo usermod -aG docker ec2-user
# add the networks
docker network create ollama-network
docker network create traefik-network

# add nvidia container toolkit
curl -s -L https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo | \
    sudo tee /etc/yum.repos.d/nvidia-container-toolkit.repo

# Install NVIDIA Container Toolkit
sudo dnf clean expire-cache
sudo dnf install -y nvidia-container-toolkit
```

```
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```
reboot to get things going
```
# run to confirm docker is up and spinning
docker ps
```

6. Assign an elastic IP address to the instance
In the AWS EC2 console, assign an Elasic IP Address to the instance.  Update the dns records to point to the new IP address.

7. Lanunch the docker-compose file and reboot
You may want to review/edit the `.env` file to ensure you don't want to make changes to the hostname or LLM models loaded.
```
cd /home/ssm-user/dev/test-docker-compose-llm
docker-compose up -d
```
wait for it to finish.

8. Reboot
- Confirm docker-compose automatically restarts after reboot
- Confirm you can reach the server with HTTPS at the hostname in the `.env`
