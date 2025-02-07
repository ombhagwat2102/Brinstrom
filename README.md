Setup Instructions

Prerequisites
Ensure you have the following:
- AWS EC2 Instance (Ubuntu or Amazon Linux recommended)
- GitHub Repository (where your application code is stored)
- SSH Key Pair (to securely connect to your EC2 instance)

Generate SSH Key (if not already created)
Run the following command on your local machine:
```bash
ssh-keygen -t rsa -b 4096 -C "github-action"
```
Press Enter for all prompts. This will generate:
- Private Key: `~/.ssh/id_rsa`
- Public Key: `~/.ssh/id_rsa.pub`

Add Public Key to EC2 Instance
1. Copy the public key:
   ```bash
   cat ~/.ssh/id_rsa.pub
   ```
2. SSH into your EC2 instance:
   ```bash
   ssh ubuntu@YOUR_EC2_PUBLIC_IP
   ```
3. Add the public key to the `authorized_keys` file:
   ```bash
   echo "PASTE_YOUR_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
   chmod 600 ~/.ssh/authorized_keys
   ```
4. Verify SSH access:
   ```bash
   ssh -i ~/.ssh/id_rsa ubuntu@YOUR_EC2_PUBLIC_IP
   ```

---

## 🔑 GitHub Secrets Configuration
Go to GitHub → Your Repository → Settings → Secrets and Variables → Actions and add these secrets:

| Secret Name         | Example Value             | Description                        |
|---------------------|--------------------------|------------------------------------|
| `SERVER_IP`        | `65.2.174.145`            | Your EC2 instance's public IP.    |
| `SERVER_USER`      | `ubuntu`                  | SSH username (e.g., `ec2-user`).  |
| `SSH_PRIVATE_KEY`  | (Your private SSH key)    | The private key to access EC2.    |

---

## ⚡ GitHub Actions Workflow (`.github/workflows/deploy.yml`)
Create a GitHub Actions workflow file at `.github/workflows/deploy.yml`:

```yaml
name: Deploy to EC2

on:
  push:
    branches:
      - main  # Change if using a different branch

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Copy files to EC2 via SCP
        uses: appleboy/scp-action@master
        with:
          host: ${{ secrets.SERVER_IP }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          source: "./*"  # Change this if needed
          target: "/home/ubuntu/app"  # Target directory on EC2
          strip_components: 1
          debug: true  # Enable debug logs
```

---

## 🛠️ Troubleshooting

### ❌ "Permission denied (publickey)" Error
- Ensure your GitHub Secret `SSH_PRIVATE_KEY` matches the public key on EC2 (`~/.ssh/authorized_keys`).
- Check file permissions:
  ```bash
  chmod 600 ~/.ssh/authorized_keys
  ```

### ❌ GitHub Actions Workflow Fails
- Check GitHub Actions logs under Actions → Your Workflow.
- Run `ssh -i ~/.ssh/id_rsa ubuntu@YOUR_PUBLIC_IP` manually to verify connectivity.
- Ensure EC2 Security Group allows inbound SSH (port 22).

### ❌ Deployment Successful but Application Not Running
- Ensure the application is started after deployment (e.g., using `pm2`, `systemctl`, or `docker`).
- SSH into the server and run:
  ```bash
  ls /home/ubuntu/app  # Check if files are copied
  ```

---

Next Steps
- Set up automatic service restarts (`pm2`, `systemctl`, or `docker`)
- Secure EC2 with firewall rules
- Implement CI/CD with rolling updates



