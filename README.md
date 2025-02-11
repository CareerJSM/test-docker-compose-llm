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
cd
wget https://us.download.nvidia.com/tesla/570.86.15/NVIDIA-Linux-x86_64-570.86.15.run
sudo sh NVIDIA-Linux-x86_64-570.86.15.run --silent --dkms

# Verify installation
nvidia-smi

# cd /home/ssm-user
# aws s3 cp --recursive s3://ec2-linux-nvidia-drivers/latest/ .
# chmod +x NVIDIA-Linux-x86_64*.run
```

4. clone the Thrive Ollama git repo
```
sudo yum install -y git


<!-- nvidia-smi

# Remove existing drivers if present

sudo yum remove nvidia\* -y

# Install prerequisites

sudo yum groupinstall "Development Tools" -y
sudo yum install kernel-devel-$(uname -r) kernel-headers-$(uname -r) -y

# Download and install Tesla driver

wget https://us.download.nvidia.com/tesla/535.104.05/NVIDIA-Linux-x86_64-535.104.05.run
sudo sh NVIDIA-Linux-x86_64-535.104.05.run --silent --dkms

# Verify installation

nvidia-smi -->
