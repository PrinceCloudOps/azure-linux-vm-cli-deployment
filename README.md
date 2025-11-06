# Deploy & Harden an Azure Linux VM Using Only Azure CLI

[![Watch Me Build It!](./thumbnail.png)](https://www.loom.com/share/e374822388fb45258a30d64934c8e55e)
<img width="1398" height="728" alt="Thumbnail" src="https://github.com/user-attachments/assets/6ccff329-bb22-4f1c-b55e-7debe6c40a84" />





This lab demonstrates deploying and securing an Ubuntu Linux VM in Microsoft Azure using **100% CLI** — no portal clicking.  
This reflects real-world cloud engineering workflows where infrastructure is created through automation.

---

## **What This Lab Covers**
- Creating Azure networking resources from scratch using CLI
- Deploying an Ubuntu 22.04 LTS VM (using a stable image URN)
- Enforcing **key-based SSH authentication**
- Hardening the VM using UFW + Fail2Ban
- Cleaning up resources to prevent cost

---

## **🔧 Prerequisites**
- Azure Subscription
- Azure CLI Installed
- SSH Keys present (`~/.ssh/id_rsa`) — CLI will generate if missing

---

## 🚀 Deployment Steps (All CLI)

### 1. Set Variables
```powershell
$LOCATION="eastus"
$RG="rg-azurecli-lab"
$VNET="vnet-azurecli-lab"
$SUBNET="subnet-azurecli-lab"
$VNET_CIDR="10.10.0.0/16"
$SUBNET_CIDR="10.10.1.0/24"
$PIP="pip-linux-cli-lab"
$NSG="nsg-linux-cli-lab"
$NIC="nic-linux-cli-lab"
$VMNAME="linux-cli-lab"
$ADMINUSER="azureuser"
$IMAGENAME="Canonical:0001-com-ubuntu-server-jammy:22_04-lts:latest"
```

### 2. Create Resource Group
```powershell
az group create --name $RG --location $LOCATION
```

### 3. Create Virtual Network + Subnet
```powershell
az network vnet create `
  --resource-group $RG `
  --name $VNET `
  --address-prefixes $VNET_CIDR `
  --subnet-name $SUBNET `
  --subnet-prefixes $SUBNET_CIDR
```

### 4. Create Public IP (Standard SKU, Static IPv4)
```powershell
az network public-ip create `
  --resource-group $RG `
  --name $PIP `
  --sku Standard `
  --allocation-method Static `
  --version IPv4
```

### 5. Create NSG + Allow SSH
```powershell
az network nsg create --resource-group $RG --name $NSG
az network nsg rule create --resource-group $RG --nsg-name $NSG --name allow-ssh --protocol Tcp --priority 1000 --destination-port-ranges 22 --access Allow --direction Inbound --source-address-prefixes "*"
```

### 6. Create NIC
```powershell
az network nic create `
  --resource-group $RG `
  --name $NIC `
  --vnet-name $VNET `
  --subnet $SUBNET `
  --network-security-group $NSG `
  --public-ip-address $PIP
```

### 7. Deploy VM (SSH Key Auth)
```powershell
az vm create `
  --resource-group $RG `
  --name $VMNAME `
  --nics $NIC `
  --image $IMAGENAME `
  --admin-username $ADMINUSER `
  --generate-ssh-keys
```
🔐 Post-Deployment Hardening (Security Best Practices)

Once connected via SSH:
# Update system packages
sudo apt update && sudo apt upgrade -y

# Enable firewall (UFW) and allow only SSH
sudo ufw allow ssh
sudo ufw enable

# Install and enable fail2ban to block SSH brute-force attacks
sudo apt install fail2ban -y
sudo systemctl enable --now fail2ban

# Disable password SSH login (key-based auth only)
sudo sed -i 's/^#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart ssh

Purpose of these steps:

| Control               | What It Does                        | Security Benefit                     |
| --------------------- | ----------------------------------- | ------------------------------------ |
| System Updates        | Patches software packages           | Reduces vulnerability exposure       |
| UFW Firewall          | Restricts inbound access            | Limits attack surface                |
| Fail2Ban              | Auto-blocks aggressive SSH attempts | Mitigates brute-force attacks        |
| Disable Password Auth | Enforces SSH key login only         | Prevents credential guessing attacks |


## 🧹 Cleanup (Avoid Unwanted Charges)

When finished, remove all deployed resources with one command:

```powershell
az group delete --name $RG --yes --no-wait
```

This removes:
- Virtual Machine
- Virtual Network + Subnet
- Network Security Group
- Public IP
- Network Interface
- Managed Disk

**One command → zero cost left behind.**


## 🎯 Key Takeaways

This project demonstrates the ability to:

- Deploy Azure resources entirely through **Azure CLI** (no portal clicking)
- Use **Virtual Networks + Subnets** for private network segmentation
- Secure access using **SSH key-based authentication** (no passwords exposed)
- Create **Standard SKU Public IP** (production-grade, zone-resilient)
- Configure **Network Security Groups** to enforce least-privilege inbound rules
- Harden a Linux VM with:
  - System updates & patching
  - UFW firewall enforcement (deny-all except SSH)
  - Fail2Ban intrusion prevention
  - Password login disabled → **Key-only SSH**
- Follow real enterprise cloud practices aligned with the **Azure Well-Architected Security Model**
- Cleanly delete cloud resources to prevent unnecessary spend


## 📌 Growth Path / Next Projects

This project establishes core Azure provisioning and secure administration fundamentals.  
The natural progression is to expand into identity security, endpoint policy enforcement, and cloud monitoring.

| Next Project | Focus | Real-World Application |
|-------------|--------|------------------------|
| Azure IAM + Conditional Access | Identity, MFA, Zero Trust | Secure user access across cloud workloads |
| Intune Device Compliance & BitLocker Escrow | Endpoint governance | Protect corporate devices + enforce encryption |
| Microsoft Sentinel SIEM Rule Engineering | Threat detection & response | Monitor and respond to real security incidents |
