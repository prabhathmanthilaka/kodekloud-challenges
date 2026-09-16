# Task 2: Create Azure Virtual Machine

##  Overview

The Nautilus DevOps team is planning to migrate a portion of their infrastructure to the Azure cloud incrementally. As part of this migration, you are tasked with creating an Azure Virtual Machine (VM).

The requirements are:

1) Use the existing resource group.

2) The VM name must be `devops-vm`, it should be in `West US` region.

3) Use the `Ubuntu 22.04 LTS` image for the VM.

4) The VM size must be `Standard_B1s`.

5) Attach a default Network Security Group (NSG) that allows inbound SSH (port 22).

6) Attach a 30 GB storage disk of type `Standard HDD`.

7) The rest of the configurations should remain as default.

After completing these steps, make sure you can SSH into the virtual machine.
Create an Azure Virtual Machine with the following specifications:

| Requirement | Value |
|-------------|-------|
| **VM Name** | `devops-vm` |
| **Region** | `West US` |
| **OS Image** | Ubuntu 22.04 LTS |
| **VM Size** | `Standard_B1s` |
| **Storage Disk** | 30 GB Standard HDD |
| **Network** | NSG with SSH (port 22) allowed |
| **Resource Group** | Use existing |

##  Azure Credentials

> **Note:** Run `showcreds` command on the `azure-client` host to retrieve credentials

| Field | Value |
|-------|-------|
| Portal URL | https://portal.azure.com |
| Username | `kk_lab_user_main-4cdca7496c4f4938@azurefreekmlprod.onmicrosoft.com` |
| Password | `vJw5&p#g` |
| Start Time | Fri Jan 16 06:03:16 UTC 2026 |
| End Time | Fri Jan 16 07:03:16 UTC 2026 |

---

##  Execution Environment

All commands are executed in the **KodeKloud lab terminal** on the **azure-client host**.

---

##  Step-by-Step Solution

### Step 1: Login to Azure CLI

Authenticate to Azure using device login flow.

```bash
az login
```

**What happens:**
- Displays a device login URL: `https://microsoft.com/devicelogin`
- Provides a unique code (e.g., `ABCD-EFGH`)

**Authentication steps:**
1. Open https://microsoft.com/devicelogin in your browser
2. Enter the code displayed in terminal
3. Login with the provided credentials
4. Return to terminal - authentication completes automatically

---

### Step 2: Verify Azure Login ✅

Confirm successful authentication with any of these methods:

#### Method 1: Show Account Details (Recommended)

```bash
az account show
```



**Success indicators:**
- `"state": "Enabled"`
- `"user"` matches your login credentials

#### Method 2: Table Format (Human-Readable)

```bash
az account show --output table
```

**Expected output:**

```
Name              CloudName    SubscriptionId                         State
----------------  -----------  -------------------------------------  -------
KodeKloud-Sub     AzureCloud   xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx   Enabled
```

#### Method 3: List Resource Groups (Practical Test)

```bash
az group list --output table
```

If you see resource groups listed, authentication is successful.

> [!WARNING]
> If you see `Please run 'az login' to setup account`, your login failed or expired.

---

### Step 3: Set Default Subscription

List available subscriptions and set the default.

```bash
az account list --output table
```


Set the subscription (if multiple exist):

```bash
az account set --subscription "<SUBSCRIPTION_ID>"
```

> [!NOTE]
> KodeKloud labs typically have only one subscription, but this step ensures consistency.

---

### Step 4: Identify Existing Resource Group

List all resource groups to find the existing one.

```bash
az group list --output table
```



**Important:** Note the resource group name (e.g., `kodekloud-rg`, `kml_rg_main-xxxxx`)

> [!IMPORTANT]
> Use the **existing** resource group. Do NOT create a new one.

---

### Step 5: Create Network Security Group

Create an NSG that allows SSH traffic on port 22.

#### Create the NSG

```bash
az network nsg create \
  --resource-group <RESOURCE_GROUP> \
  --name devops-nsg \
  --location westus
```

Replace `<RESOURCE_GROUP>` with your actual resource group name.

#### Add SSH Rule

```bash
az network nsg rule create \
  --resource-group <RESOURCE_GROUP> \
  --nsg-name devops-nsg \
  --name AllowSSH \
  --priority 1000 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-ranges 22
```

**Rule Parameters Explained:**

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `--name` | `AllowSSH` | Rule name |
| `--priority` | `1000` | Rule precedence (lower = higher priority) |
| `--direction` | `Inbound` | Traffic direction |
| `--access` | `Allow` | Permit traffic |
| `--protocol` | `Tcp` | Protocol type |
| `--destination-port-ranges` | `22` | SSH port |

✅ **Requirement satisfied:** NSG with inbound SSH access configured

---

### Step 6: Create the Virtual Machine 

Execute the main command to create the VM with all specifications.

```bash
az vm create \
  --resource-group <RESOURCE_GROUP> \
  --name devops-vm \
  --location westus \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --nsg devops-nsg \
  --os-disk-size-gb 30 \
  --storage-sku Standard_LRS
```

**Command Parameters Breakdown:**

| Parameter | Value | Requirement Met |
|-----------|-------|-----------------|
| `--resource-group` | Existing RG | Uses existing resource group ✅ |
| `--name` | `devops-vm` | VM name requirement ✅ |
| `--location` | `westus` | West US region ✅ |
| `--image` | `Ubuntu2204` | Ubuntu 22.04 LTS ✅ |
| `--size` | `Standard_B1s` | VM size specification ✅ |
| `--admin-username` | `azureuser` | Default admin user |
| `--generate-ssh-keys` | Auto-generate | Creates SSH key pair |
| `--nsg` | `devops-nsg` | Attaches NSG with SSH ✅ |
| `--os-disk-size-gb` | `30` | 30 GB disk size ✅ |
| `--storage-sku` | `Standard_LRS` | Standard HDD ✅ |



**What Azure does:**
- Creates the virtual machine
- Generates and stores SSH keys locally (`~/.ssh/id_rsa`)
- Assigns a public IP address
- Attaches the NSG
- Creates a 30GB Standard HDD disk
- Returns VM details including public IP

> [!TIP]
> The VM creation process takes 2-3 minutes. Wait for the command to complete.

---

### Step 7: Retrieve VM Public IP Address

Get the public IP address for SSH connection.

```bash
az vm list-ip-addresses \
  --resource-group <RESOURCE_GROUP> \
  --name devops-vm \
  --output table
```

**Expected output:**

```
VirtualMachine    PublicIPAddresses    PrivateIPAddresses
----------------  -------------------  --------------------
devops-vm         20.123.45.67         10.0.0.4
```

 **Copy the Public IP Address** - you'll need it for SSH access

**Alternative method (get JSON output):**

```bash
az vm show \
  --resource-group <RESOURCE_GROUP> \
  --name devops-vm \
  --show-details \
  --query publicIps \
  --output tsv
```

---

### Step 8: SSH into the Virtual Machine

Verify SSH access to confirm the VM is properly configured.

#### Locate the SSH Private Key

Azure CLI automatically saved the SSH key:

```bash
ls -l ~/.ssh/id_rsa
```

#### Connect via SSH

```bash
ssh azureuser@<PUBLIC_IP>
```

Replace `<PUBLIC_IP>` with the IP address from Step 7.

**First-time connection prompt:**

```
The authenticity of host '20.123.45.67 (20.123.45.67)' can't be established.
ECDSA key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Type `yes` and press Enter.

**Expected result:**

```
Welcome to Ubuntu 22.04.x LTS (GNU/Linux 5.15.x-xxx-azure x86_64)
```

✅ **SSH connection successful!**

---

### Step 9: Verify Ubuntu Version

Inside the VM, confirm the operating system version.

```bash
lsb_release -a
```

**Expected output:**

```
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.x LTS
Release:        22.04
Codename:       jammy
```

**Alternative verification:**

```bash
cat /etc/os-release
```

✅ **Confirms Ubuntu 22.04 LTS is installed**

---

### Step 10: Test VM Functionality (Optional)

Run basic commands to verify the VM is working properly.

```bash
# Check system resources
free -h

# Check disk space
df -h

# Check network interfaces
ip addr show

# Test internet connectivity
ping -c 4 google.com

# Exit the VM
exit
```

---


---

## 🎯 Final Verification Summary

| Requirement | Status | Verification Command |
|-------------|--------|---------------------|
| Existing Resource Group | ✅ | `az group list --output table` |
| VM Name: devops-vm | ✅ | `az vm show -n devops-vm -g <RG>` |
| Region: West US | ✅ | Check `location` in VM details |
| OS: Ubuntu 22.04 LTS | ✅ | `lsb_release -a` (inside VM) |
| VM Size: Standard_B1s | ✅ | Check `hardwareProfile.vmSize` |
| SSH Allowed (Port 22) | ✅ | `az network nsg rule list --nsg-name devops-nsg` |
| Disk: 30GB Standard HDD | ✅ | Check `storageProfile.osDisk` |
| SSH Login Successful | ✅ | `ssh azureuser@<PUBLIC_IP>` |

---

##  Common Troubleshooting

### Error: "Location 'westus' is not valid"

**Solution:** Use lowercase: `westus` not `WestUS` or `West US`

### Error: "The subscription is not registered to use namespace 'Microsoft.Compute'"

**Solution:** Register the provider:

```bash
az provider register --namespace Microsoft.Compute
```

### Error: SSH connection refused

**Possible causes:**
1. NSG rule not applied correctly
2. VM still starting up (wait 2-3 minutes)
3. Wrong public IP address

**Solution:** Verify NSG rules:

```bash
az network nsg rule list \
  --resource-group <RESOURCE_GROUP> \
  --nsg-name devops-nsg \
  --output table
```

### Error: Image 'Ubuntu2204' not found

**Solution:** List available images:

```bash
az vm image list --offer Ubuntu --all --output table
```

Use exact image URN or alias.

### Cannot SSH - Permission denied (publickey)

**Solution:** Verify SSH key exists:

```bash
ls -l ~/.ssh/id_rsa
```

If missing, manually specify key:

```bash
ssh -i ~/.ssh/id_rsa azureuser@<PUBLIC_IP>
```

---

## Security Best Practices

> [!WARNING]
> **Never share or commit your private SSH key (`~/.ssh/id_rsa`)**

**Recommended practices:**
- Keep private keys secure with proper permissions: `chmod 600 ~/.ssh/id_rsa`
- Use key-based authentication instead of passwords
- Regularly rotate SSH keys
- Limit NSG rules to specific IP ranges in production
- Enable Azure Security Center for threat detection

---

## 🧹 Cleanup (Optional)

To delete the VM and associated resources after testing:

```bash
# Delete the VM (keeps disk and network resources)
az vm delete \
  --resource-group <RESOURCE_GROUP> \
  --name devops-vm \
  --yes

# Delete the NSG
az network nsg delete \
  --resource-group <RESOURCE_GROUP> \
  --name devops-nsg

# Delete all associated resources
az vm delete \
  --resource-group <RESOURCE_GROUP> \
  --name devops-vm \
  --yes \
  --force-deletion yes
```

> [!CAUTION]
> Only run cleanup commands if instructed or after task completion.

---

## 📚 Additional Resources

- [Azure VM Documentation](https://docs.microsoft.com/azure/virtual-machines/)
- [Azure CLI VM Commands](https://docs.microsoft.com/cli/azure/vm)
- [Ubuntu on Azure](https://ubuntu.com/azure)
- [Network Security Groups](https://docs.microsoft.com/azure/virtual-network/network-security-groups-overview)

