# 🚀 Set Up a Web App Using AWS and VS Code

> **Part 1 of a CI/CD Pipeline Series** — Building the foundation of a continuous integration pipeline by deploying a Java web application on AWS EC2.

---

## 📌 Project Overview

In this project, I set up a Java web application from scratch on an AWS EC2 instance and connected to it remotely using VS Code. This lays the groundwork for a full CI/CD pipeline in upcoming projects.

| Detail | Info |
|---|---|
| Cloud Provider | AWS (EC2 — ap-south-1) |
| Build Tool | Apache Maven |
| Language | Java (JSP) |
| IDE | VS Code + Remote-SSH |
| OS | Amazon Linux 2023 |
| Time Taken | ~2 hours |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────┐
│                  Your Local Machine              │
│                                                  │
│   VS Code  ──── Remote-SSH Extension ────────┐  │
│                                              │  │
└──────────────────────────────────────────────┼──┘
                                               │ SSH (port 22)
                                               │ (.pem key pair)
                                               ▼
┌─────────────────────────────────────────────────┐
│              AWS EC2 Instance                    │
│              (Amazon Linux 2023)                 │
│                                                  │
│   ┌──────────────────────────────────────────┐  │
│   │         Java Web App (Maven)             │  │
│   │                                          │  │
│   │  nextwork-web-project/                   │  │
│   │  ├── src/                                │  │
│   │  │   └── main/                           │  │
│   │  │       ├── webapp/                     │  │
│   │  │       │   ├── WEB-INF/                │  │
│   │  │       │   └── index.jsp  ◄── Entry    │  │
│   │  │       └── resources/                  │  │
│   │  ├── target/  ◄── .war build output      │  │
│   │  └── pom.xml  ◄── Maven config           │  │
│   └──────────────────────────────────────────┘  │
│                                                  │
└─────────────────────────────────────────────────┘
```

---

## 🛠️ Tools & Concepts Used

| Tool / Concept | Purpose |
|---|---|
| **AWS EC2** | Virtual server to host and run the web app |
| **Key Pair (.pem)** | Secure SSH authentication to EC2 |
| **SSH** | Remote access to EC2 from terminal |
| **VS Code Remote-SSH** | Edit files on EC2 directly from local VS Code |
| **Apache Maven** | Build automation — compile, test, package the Java app |
| **Java / JSP** | Core language powering the web application |
| **index.jsp** | Entry point / landing page of the web app |
| **pom.xml** | Maven project configuration and dependency management |

---

## 📋 Step-by-Step Setup Guide

### Step 1 — Launch an EC2 Instance

1. Log in to **AWS Console** → navigate to **EC2**
2. Click **Launch Instance**
3. Choose **Amazon Linux 2023** AMI
4. Select instance type: `t2.micro` (free tier eligible)
5. Under **Key Pair**, create a new key pair (e.g. `network-keypair`)
   - AWS automatically downloads `network-keypair.pem`
6. Enable **SSH (port 22)** in the security group
7. Launch the instance

### Step 2 — Secure Your Key Pair

On **Windows** (PowerShell), set correct permissions on the `.pem` file:

```powershell
icacls "network-keypair.pem" /reset
icacls "network-keypair.pem" /grant:r "YOUR_USERNAME:(R)"
icacls "network-keypair.pem" /inheritance:r
```

> This replicates Linux's `chmod 400` — restricting the key to read-only for the owner only, required for SSH to accept the key.

On **Linux / Mac**:
```bash
chmod 400 network-keypair.pem
```

### Step 3 — SSH into the EC2 Instance

```bash
ssh -i "path/to/network-keypair.pem" ec2-user@<your-ec2-public-dns>
```

Example:
```bash
ssh -i "network-keypair.pem" ec2-user@ec2-13-127-187-162.ap-south-1.compute.amazonaws.com
```

### Step 4 — Install Java and Maven on EC2

```bash
# Update packages
sudo yum update -y

# Install Java
sudo yum install java-11-amazon-corretto -y

# Install Maven
sudo yum install maven -y

# Verify installations
java -version
mvn -version
```

### Step 5 — Generate the Java Web App with Maven

```bash
mvn archetype:generate \
  -DgroupId=com.nextwork.app \
  -DartifactId=nextwork-web-project \
  -DarchetypeArtifactId=maven-archetype-webapp \
  -DinteractiveMode=false
```

Maven will generate the full project structure including `src/`, `webapp/`, `index.jsp`, and `pom.xml`.

### Step 6 — Connect VS Code via Remote-SSH

1. Install **Remote - SSH** extension in VS Code
2. Create/edit the SSH config file (`~/.ssh/config` on Mac/Linux, `C:\Users\<you>\.ssh\config` on Windows):

```
Host my-ec2
    HostName ec2-13-127-187-162.ap-south-1.compute.amazonaws.com
    User ec2-user
    IdentityFile ~/path/to/network-keypair.pem
```

3. In VS Code: press `Ctrl+Shift+P` → **Remote-SSH: Connect to Host** → select `my-ec2`
4. VS Code opens directly inside your EC2 instance — browse files, edit code, use the terminal, all from your local editor

### Step 7 — Edit the Web App

Navigate to `src/main/webapp/index.jsp` and customize it:

```html
<html>
  <body>
    <h2>Hello Revathi!</h2>
    <p>This is my NextWork web application working!</p>
  </body>
</html>
```

### Step 8 — Build the Application

```bash
cd nextwork-web-project
mvn package
```

This compiles the app and generates a `.war` file inside the `target/` folder — ready for deployment to a servlet container like Tomcat.

---

## 📄 Project Documentation

Full project walkthrough: [View PDF](docs/legendary-aws-devops-vscode.pdf)
---

## 💡 Key Learnings & Reflections

### What I learned

- **EC2 setup end-to-end** — from launching an instance to managing security groups, key pairs, and public DNS addresses
- **SSH key permissions matter** — without `chmod 400` (or the Windows `icacls` equivalent), SSH will refuse the connection. A small but critical detail
- **VS Code Remote-SSH is a game changer** — editing files directly on EC2 felt exactly like coding locally. No more `nano` or `vim` on the server!
- **Maven project structure** — understanding how `src/`, `webapp/`, `index.jsp`, and `pom.xml` fit together as the foundation of a Java web app
- **`.war` files** — Maven's `package` command bundles the entire app into a Web Application Archive ready for deployment

### What surprised me

The Remote-SSH extension in VS Code was far smoother than expected. It completely removed the friction of working on a remote server — no more copying files back and forth or relying on terminal-only editors.

### What's next

This is **Part 1** of a CI/CD pipeline series. Upcoming projects will cover:
- [ ] Part 2 — Setting up a Jenkins CI server
- [ ] Part 3 — Automating builds with a Jenkins pipeline
- [ ] Part 4 — Deploying to AWS with CD automation

---

## 🔗 Resources

- [NextWork.org](https://nextwork.org) — Project series source
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [Apache Maven](https://maven.apache.org/)
- [VS Code Remote-SSH](https://code.visualstudio.com/docs/remote/ssh)

---

## 👤 Author

**Revathi Annepu** — NextWork Student  
Part of a hands-on DevOps learning journey transitioning into cloud & DevOps engineering.

---

> ⭐ If you found this helpful, consider starring the repo!
