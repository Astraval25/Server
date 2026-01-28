
# CoreFlow Frontend - Automatic Deployment to Linux VPS using GitHub Actions

---

## A. ONE-TIME VPS SETUP

### 1. Login to VPS

```bash
ssh -p 2222 astraval@astraval.com
```

---

### 2. Create project directories

```bash
sudo mkdir -p /var/www/domains/coreflow.astraval.com/{public,source}
sudo chown -R astraval:astraval /var/www/domains/coreflow.astraval.com
```

---

### 3. Allow password-less sudo for deploy commands

```bash
sudo visudo
```

Add **at bottom**:

```bash
astraval ALL=(ALL) NOPASSWD: /bin/rm, /bin/cp, /bin/chown, /usr/bin/find
```

Save & exit.

---

### 4. Verify sudo (must NOT ask password)

```bash
sudo rm --version
sudo cp --version
```

---

## B. SSH KEY SETUP (WINDOWS)

### 5. Generate SSH key

```bat
ssh-keygen -t ed25519 -C "github-coreflow-deploy"
```

Press **Enter** for all prompts.

---

### 6. Copy public key

```bat
type C:\Users\DELL\.ssh\id_ed25519.pub
```

---

### 7. Add key to VPS

```bash
ssh -p 2222 astraval@astraval.com
```

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
```

Paste public key → Save → Exit

```bash
chmod 600 ~/.ssh/authorized_keys
exit
```

---

### 8. Test SSH (no password)

```bat
ssh -p 2222 astraval@astraval.com
```

---

## C. GITHUB SECRETS

GitHub → **Repo → Settings → Secrets → Actions**

Add:

```
VPS_HOST = astraval.com
VPS_USER = astraval
VPS_PORT = 2222
```

Private key:

```bat
type C:\Users\DELL\.ssh\id_ed25519
```

Add as:

```
VPS_SSH_KEY = (full private key)
```

---

## D. GITHUB ACTION WORKFLOW

### 9. Create file

```text
.github/workflows/deploy.yml
```

### 10. Paste this EXACT content

```yaml
name: Deploy CoreFlow Frontend

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - run: npm ci
      - run: npm run build

      - name: Upload build
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          port: ${{ secrets.VPS_PORT }}
          key: ${{ secrets.VPS_SSH_KEY }}
          source: "dist/*"
          target: "/var/www/domains/coreflow.astraval.com/source"
          overwrite: true

      - name: Atomic deploy
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          port: ${{ secrets.VPS_PORT }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            cd /var/www/domains/coreflow.astraval.com
            sudo rm -rf public/*
            sudo cp -r source/dist/* public/
            sudo chown -R www-data:www-data public
            sudo find public -type d -exec chmod 755 {} \;
            sudo find public -type f -exec chmod 644 {} \;
            rm -rf source/*
```

---

## E. DEPLOY

### 11. Commit & push

```bash
git add .github/workflows/deploy.yml
git commit -m "ci: auto deploy frontend"
git push origin main
```

---

## F. VERIFY

### 12. Check status

GitHub → **Actions** → Green ✔

### 13. Open site

```
https://coreflow.astraval.com
```

---

## ✅ DONE

* Secure SSH
* Zero-downtime deploy
* Industry-standard CI/CD
* Fully automated 🚀
