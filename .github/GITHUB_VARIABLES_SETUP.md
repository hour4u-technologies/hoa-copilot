# GitHub Variables and Secrets Setup Guide

This document lists all the variables and secrets you need to configure in GitHub for the deployment workflows.

## 📍 Where to Add These

1. **Secrets**: Repository → Settings → Secrets and variables → Actions → **Secrets** tab → New repository secret
2. **Variables**: Repository → Settings → Secrets and variables → Actions → **Variables** tab → New repository variable

---

## 🔐 Required Secrets (Sensitive Data)

### Development Environment Secrets

| Secret Name | Description | Example Value | Required |
|------------|-------------|---------------|----------|
| `EC2_DEV_HOST` | EC2 instance public IP or domain name for DEV | `ec2-12-34-56-78.compute-1.amazonaws.com` or `12.34.56.78` | ✅ Yes |
| `EC2_DEV_USER` | SSH username for DEV EC2 instance | `ec2-user` (Amazon Linux) or `ubuntu` (Ubuntu) | ✅ Yes |
| `EC2_DEV_SSH_PRIVATE_KEY` | Complete content of your SSH private key (.pem file) for DEV | Full content including `-----BEGIN RSA PRIVATE KEY-----` and `-----END RSA PRIVATE KEY-----` | ✅ Yes |

### Production Environment Secrets

| Secret Name | Description | Example Value | Required |
|------------|-------------|---------------|----------|
| `EC2_PROD_HOST` | EC2 instance public IP or domain name for PROD | `ec2-12-34-56-78.compute-1.amazonaws.com` or `12.34.56.78` | ✅ Yes |
| `EC2_PROD_USER` | SSH username for PROD EC2 instance | `ec2-user` (Amazon Linux) or `ubuntu` (Ubuntu) | ✅ Yes |
| `EC2_PROD_SSH_PRIVATE_KEY` | Complete content of your SSH private key (.pem file) for PROD | Full content including `-----BEGIN RSA PRIVATE KEY-----` and `-----END RSA PRIVATE KEY-----` | ✅ Yes |

---

## 📝 Optional Variables (Non-Sensitive Configuration)

### Development Environment Variables

| Variable Name | Description | Default Value | Required |
|--------------|-------------|---------------|----------|
| `EC2_DEV_PORT` | Port number to expose on DEV EC2 instance | `80` | ❌ No |

### Production Environment Variables

| Variable Name | Description | Default Value | Required |
|--------------|-------------|---------------|----------|
| `EC2_PROD_PORT` | Port number to expose on PROD EC2 instance | `80` | ❌ No |

---

## 📋 Step-by-Step Setup Instructions

### Step 1: Get Your EC2 Instance Details

1. **EC2 Host (IP/Domain)**:
   - Go to AWS Console → EC2 → Instances
   - Find your instance → Copy the "Public IPv4 address" or "Public IPv4 DNS"
   - Example: `ec2-12-34-56-78.us-east-1.compute.amazonaws.com` or `12.34.56.78`

2. **EC2 User**:
   - Amazon Linux: `ec2-user`
   - Ubuntu: `ubuntu`
   - RHEL: `ec2-user` or `root`
   - Check your AMI documentation if unsure

3. **SSH Private Key (.pem file)**:
   - The `.pem` file you downloaded when creating the EC2 instance
   - Or the private key you're using to SSH into the instance
   - You need the **complete file content**

### Step 2: Add Secrets to GitHub

1. Go to your repository on GitHub
2. Click **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. For each secret:
   - **Name**: Use exact name from the table above (case-sensitive)
   - **Secret**: Paste the value
   - Click **Add secret**

### Step 3: Add Variables to GitHub (Optional)

1. In the same page, click **Variables** tab
2. Click **New repository variable**
3. Add variables if you want to override defaults:
   - `EC2_DEV_PORT` (if not using default 8080)
   - `EC2_PROD_PORT` (if not using default 80)

---

## 🔑 How to Get SSH Private Key Content

### Option 1: From .pem File

```bash
# On your local machine
cat /path/to/your-key.pem
# Copy the entire output including BEGIN and END lines
```

### Option 2: From Existing SSH Key

```bash
# If you're using a different key
cat ~/.ssh/id_rsa
# Copy the entire output
```

### Example Format:

```
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA...
(many lines of encoded data)
...
-----END RSA PRIVATE KEY-----
```

**Important**: 
- Include the `-----BEGIN` and `-----END` lines
- Include all lines in between
- No extra spaces or line breaks

---

## 🧪 Testing Your Setup

After adding secrets, you can test by:

1. Go to **Actions** tab
2. Select **Deploy to AWS EC2** workflow
3. Click **Run workflow**
4. Choose environment (dev or prod)
5. Click **Run workflow**

The workflow will:
- ✅ Use your secrets to connect to EC2
- ✅ Deploy the container
- ✅ Show deployment status

---

## 🔒 Security Best Practices

1. **Never commit secrets to code** - Always use GitHub Secrets
2. **Use separate keys for dev/prod** - Different EC2 instances should have different keys
3. **Rotate keys regularly** - Update secrets periodically
4. **Limit access** - Only give repository access to trusted team members
5. **Use IAM roles** - For production, consider using AWS IAM roles instead of keys

---

## 📊 Summary Checklist

### Required Secrets (6 total):

- [ ] `EC2_DEV_HOST`
- [ ] `EC2_DEV_USER`
- [ ] `EC2_DEV_SSH_PRIVATE_KEY`
- [ ] `EC2_PROD_HOST`
- [ ] `EC2_PROD_USER`
- [ ] `EC2_PROD_SSH_PRIVATE_KEY`

### Optional Variables:

- [ ] `EC2_DEV_PORT` (default: 80)
- [ ] `EC2_PROD_PORT` (default: 80)

---

## 🆘 Troubleshooting

### "Permission denied (publickey)" Error
- Verify `EC2_SSH_PRIVATE_KEY` includes BEGIN/END lines
- Check that the public key is added to EC2 instance
- Verify `EC2_USER` is correct for your AMI

### "Connection timeout" Error
- Verify `EC2_HOST` is correct (IP or domain)
- Check EC2 security group allows SSH (port 22) from GitHub Actions IPs
- Verify instance is running

### "Container not starting" Error
- Check Docker is installed on EC2: `docker --version`
- Verify user has Docker permissions: `sudo usermod -aG docker $USER`
- Check port is available: `sudo netstat -tulpn | grep <PORT>`

---

## 📞 Need Help?

If you encounter issues:
1. Check GitHub Actions logs in the Actions tab
2. Verify all secrets are correctly set
3. Test SSH connection manually: `ssh -i key.pem user@host`
4. Check EC2 instance logs and Docker logs
