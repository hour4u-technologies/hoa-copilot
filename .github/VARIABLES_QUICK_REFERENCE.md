# 🚀 Quick Reference: GitHub Variables & Secrets

## ✅ Required Secrets (Add in: Settings → Secrets and variables → Actions → Secrets)

### Development Environment
```
EC2_DEV_HOST          → Your DEV EC2 instance IP/domain
EC2_DEV_USER          → SSH username (usually "ec2-user" or "ubuntu")
EC2_DEV_SSH_PRIVATE_KEY → Complete .pem file content (with BEGIN/END lines)
```

### Production Environment
```
EC2_PROD_HOST          → Your PROD EC2 instance IP/domain
EC2_PROD_USER          → SSH username (usually "ec2-user" or "ubuntu")
EC2_PROD_SSH_PRIVATE_KEY → Complete .pem file content (with BEGIN/END lines)
```

---

## 📝 Optional Variables (Add in: Settings → Secrets and variables → Actions → Variables)

```
EC2_DEV_PORT  → Port for DEV (default: 80)
EC2_PROD_PORT → Port for PROD (default: 80)
```

---

## 🎯 How It Works

- Push to `main` branch → Deploys to **PRODUCTION**
- Push to `develop` branch → Deploys to **DEVELOPMENT**
- Manual trigger → Choose environment (dev/prod)

---

## 📋 Copy-Paste Checklist

### Secrets to Add:
- [ ] `EC2_DEV_HOST`
- [ ] `EC2_DEV_USER`
- [ ] `EC2_DEV_SSH_PRIVATE_KEY`
- [ ] `EC2_PROD_HOST`
- [ ] `EC2_PROD_USER`
- [ ] `EC2_PROD_SSH_PRIVATE_KEY`

### Variables to Add (Optional):
- [ ] `EC2_DEV_PORT` (default: 8080)
- [ ] `EC2_PROD_PORT` (default: 80)

---

## 💡 Example Values

**EC2_DEV_HOST**: `ec2-12-34-56-78.us-east-1.compute.amazonaws.com`  
**EC2_DEV_USER**: `ec2-user`  
**EC2_DEV_SSH_PRIVATE_KEY**: 
```
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA...
(entire key content)
...
-----END RSA PRIVATE KEY-----
```

---

See `GITHUB_VARIABLES_SETUP.md` for detailed instructions.
