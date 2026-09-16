# Task 1: Create Azure SSH Key Pair

##  Overview

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

##  Task Requirements

Create an SSH key pair with the following specifications:
- **Name:** `datacenter-kp`
- **Key Type:** `rsa`

##  Azure Credentials

> **Note:** Run `showcreds` command on the `azure-client` host to retrieve credentials

| Field | Value |
|-------|-------|
| Portal URL | https://portal.azure.com |
| Username | `kk_lab_user_main-b9ce1552758746cc@azurefreekmlprod.onmicrosoft.com` |
| Password | `&TkRSy@%` |


---

##  Execution Environment

All commands are executed in the **KodeKloud lab terminal** on the **azure-client host**.

---

##  Step-by-Step Solution

### Step 1: Login to Azure CLI

Authenticate to Azure using the provided credentials.

```bash
az login
```

**What happens:**
- Opens browser-based login or provides device login code
- Authenticate using the credentials above
- Returns JSON output with `tenantId`, `subscriptionId`, and `user` information

**Success indicator:** You'll see account details in JSON format.

---

### Step 2: Verify Azure Account (Optional)

Confirm you're logged into the correct Azure subscription.

```bash
az account show
```

**Purpose:** Displays active Azure subscription and confirms authentication.

**Expected output:**

<img width="670" height="420" alt="image" src="https://github.com/user-attachments/assets/ad716cb2-9590-4929-b233-068028593ec0" />

---

### Step 3: List Existing SSH Keys (Optional)

Check if `datacenter-kp` already exists to avoid conflicts.

```bash
az sshkey list
```

**Purpose:** Ensures no naming conflicts with existing SSH keys.

**Expected output:**

<img width="336" height="96" alt="git2" src="https://github.com/user-attachments/assets/9b91e85e-f7e8-400d-aea2-bbf7a1b2fd2d" />


---

### Step 4: Identify Resource Group

Find the resource group where the SSH key will be created.

```bash
az group list --output table
```

**Expected output:** List of resource groups (e.g., `kml_rg_main-b9ce1552758746cc`)

<img width="638" height="172" alt="git3" src="https://github.com/user-attachments/assets/b5b618f5-7436-4c4f-ad50-ff4c20c852f5" />
ng)

> **Note:** Use the resource group name from your lab environment.

---

### Step 5: Create SSH Key Pair 

Create the SSH key pair in Azure.

```bash
az sshkey create \
  --name datacenter-kp \
  --resource-group kml_rg_main-b9ce1552758746cc
```

> ⚠️ **Important:** Replace `kml_rg_main-b9ce1552758746cc` with your actual resource group name.

#### Why no `--key-type rsa` flag?

In some Azure CLI versions (including KodeKloud labs), the `--key-type` parameter is not supported. Azure automatically generates **RSA keys by default**, so this parameter is optional and the requirement is still fulfilled.

**Command Parameters Explained:**

| Parameter | Description |
|-----------|-------------|
| `az sshkey create` | Creates an SSH key resource in Azure |
| `--name datacenter-kp` | Sets the SSH key name |
| `--resource-group` | Specifies the Azure resource group |

**What Azure does:**
- ✅ Generates an RSA SSH key pair
- ✅ Stores the public key in Azure
- ✅ Saves the private key locally (typically in `~/.ssh/`)
- ✅ Returns output with `publicKey`, `privateKeyPath`, and `fingerprint`


---

### Step 6: Verify SSH Key Creation

Confirm the SSH key was created successfully.

```bash
az sshkey show \
  --name datacenter-kp \
  --resource-group kml_rg_main-b9ce1552758746cc
```

**Success indicators:**
- SSH key exists
- Name matches: `datacenter-kp`
- Key type is RSA



---

### Step 7: Locate Private Key Locally (Optional)

The private key is stored locally on the azure-client host.

```bash
ls -l ~/.ssh/
```

**Expected location:** `~/.ssh/datacenter-kp`

> 🔒 **Security Note:** Never share or expose your private key.

---

##  Task Validation Checklist

Before submitting, verify:

- [ ] SSH key name is `datacenter-kp`
- [ ] Key type is RSA
- [ ] Created using Azure CLI
- [ ] Key exists in Azure (verified with `az sshkey show`)

---

##  Why SSH Keys Matter

SSH keys provide several advantages over password-based authentication:

- **Enhanced Security:** Uses asymmetric cryptography
- **Brute-Force Protection:** Not vulnerable to password guessing attacks
- **Industry Standard:** Widely used in DevOps and cloud environments
- **Automation Friendly:** Enables secure, passwordless automation

---

##  Common Troubleshooting

### Error: `unrecognized arguments: --key-type rsa`

**Solution:** Remove the `--key-type rsa` parameter. Azure defaults to RSA automatically.

### Error: Resource group not found

**Solution:** Run `az group list --output table` and use the correct resource group name.

### Authentication failed

**Solution:** Re-run `az login` and ensure you're using the correct credentials.

---

## 📚 Additional Resources

- [Azure SSH Key Documentation](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/create-ssh-keys-detailed)
- [Azure CLI Reference - az sshkey](https://docs.microsoft.com/en-us/cli/azure/sshkey)

---

##  Notes

This SSH key pair will be used in subsequent migration tasks for secure VM access. Keep the private key secure and backed up appropriately.

---

