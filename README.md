# Jenkins_test
Jenkins_test
Username :- sai
password :- 1

# 🚀 Jenkins CI/CD with AWS EC2 and GitHub (Beginner Friendly Guide)

This is a step-by-step guide for setting up a **CI/CD pipeline** using:

* **AWS EC2 (Ubuntu)** 🖥️
* **Jenkins** 🔧
* **GitHub Webhooks** 🐙
* **Apache Web Server** 🌐

When finished → every time you change `index.html` (or other files) in GitHub → Jenkins will auto-deploy them to your EC2 → and you’ll see it live in your browser.

---

## 🟢 Step 1: Launch EC2 Instance

1. Go to **AWS Console → EC2 → Launch Instance**

   * AMI: **Ubuntu Server 22.04 LTS**
   * Instance type: **t2.micro (Free Tier)**
   * Key pair: **Download .pem file** (example: `new_account.pem`).
   * Security group inbound rules:

     * **SSH (22)** → My IP only
     * **HTTP (80)** → 0.0.0.0/0
     * **Custom TCP (8080)** → 0.0.0.0/0

2. Save your `.pem` key safely. You’ll need it for **SSH** and **Jenkins SSH plugin**.

---

## 🟢 Step 2: Install Dependencies on EC2

SSH into your EC2 (use Git Bash, MobaXterm, or terminal):

```bash
ssh -i new_account.pem ubuntu@<your-ec2-public-ip>
```

Now run these commands:

```bash
# Update system
sudo apt update -y
sudo apt upgrade -y

# Install Apache & Git
sudo apt install -y apache2 git

# Install Java (needed for Jenkins)
sudo apt install -y fontconfig openjdk-21-jre

# Install Jenkins
sudo mkdir -p /etc/apt/keyrings
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install -y jenkins

# Start Jenkins
sudo systemctl enable --now jenkins
sudo systemctl status jenkins
```

Check Jenkins in your browser:
👉 `http://<your-ec2-public-ip>:8080`

---

## 🟢 Step 3: Configure Jenkins

1. Get the initial password:

   ```bash
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```
2. Open Jenkins in browser → enter password → create user (`sai / 1 / sai@gmail.com`).
3. Install suggested plugins.

---

## 🟢 Step 4: Setup Web Directory for Deployment

```bash
sudo mkdir -p /var/www/html/webdirectory
sudo chown -R jenkins:jenkins /var/www/html/webdirectory
sudo chmod -R 755 /var/www/html/webdirectory
```

---

## 🟢 Step 5: Install Jenkins Plugin (Publish Over SSH)

1. In Jenkins → **Manage Jenkins → Plugins → Available Plugins** → install **Publish Over SSH**.
2. Go to **Manage Jenkins → System → Publish over SSH**.
3. Add new server:

   * **Name**: `DemoServer`
   * **Hostname**: `<your-ec2-public-ip>`
   * **Username**: `ubuntu` (for Ubuntu AMI)
   * **Remote Directory**: `/var/www/html/webdirectory`
   * **Key**: paste contents of `new_account.pem`

Click **Test Configuration** → must say **Success** ✅.
Save changes.

---

## 🟢 Step 6: Create a GitHub Repo with HTML Page

1. On GitHub → create new public repo (example: `Jenkins_test`).
2. Add a simple file `index.html`:

```html
<!DOCTYPE html>
<html>
  <head><title>Hi Page</title></head>
  <body>
    <h1>👋 Hi, Welcome!</h1>
    <p>This is a simple HTML page created by me.</p>
  </body>
</html>
```

---

## 🟢 Step 7: Create Jenkins Job

1. Dashboard → **New Item → Freestyle Project** (`demo_project`).

2. Enable **GitHub Project** → paste repo URL.

3. In **Source Code Management** → Git:

   * Repo URL: `https://github.com/<username>/Jenkins_test.git`
   * Branch: `*/main`

4. In **Build Triggers** → check **GitHub hook trigger for GITScm polling**.

5. In **Post-build Actions → Send build artifacts over SSH**:

   * **Name**: `DemoServer`
   * **Source files**: `**/*.html, **/*.css` (or `**/*` for all files)
   * **Remote directory**: `/`

Save ✅

---

## 🟢 Step 8: Configure GitHub Webhook

1. Go to your GitHub repo → **Settings → Webhooks → Add Webhook**.
2. Payload URL:

   ```
   http://<your-ec2-public-ip>:8080/github-webhook/
   ```
3. Content type: `application/json`.
4. Select **Just the push event**.
5. Save.

---

## 🟢 Step 9: Fix Apache to Show Your Site

Change Apache default directory:

```bash
sudo vim /etc/apache2/sites-enabled/000-default.conf
```

Change:

```
DocumentRoot /var/www/html
```

to:

```
DocumentRoot /var/www/html/webdirectory
```

if it is not working then execute this in ec2 connect
```
sudo tee /etc/apache2/sites-enabled/000-default.conf > /dev/null <<EOL
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/webdirectory

    <Directory /var/www/html/webdirectory>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog \${APACHE_LOG_DIR}/error.log
    CustomLog \${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
EOL
```

Save → exit → restart Apache:

```bash
sudo systemctl restart apache2
```

Now open:
👉 `http://<your-ec2-public-ip>/`

---


## 🟢 Step 10: Test the Pipeline 🎉

1. Edit `index.html` in GitHub → commit changes.
2. Jenkins will auto-build → deploy to EC2.
3. Refresh `http://<your-ec2-public-ip>/` → changes live 🚀.

---

## 🔁 Important Note

If you **Stop/Start EC2 instance**, public IP changes.
When that happens, update IP in:

1. GitHub Webhook → Payload URL change IP:8080 with new IP 
2. Go in Manage Jenkins -> Seeting -> last you will see Publish over SSH → change  Hostname(old IP) to new one .

---

## ✅ Hurray!

You’ve built a full **CI/CD pipeline** using **Jenkins + GitHub + AWS EC2 + Apache**.

Whenever you push code → it goes live automatically. 🎯

---
****
