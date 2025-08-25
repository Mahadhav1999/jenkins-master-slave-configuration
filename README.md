
# 🚀 Jenkins Master-Slave Configuration

This repository demonstrates how to set up a **Jenkins Master-Slave (Distributed Build) architecture** using:

* **Ubuntu Master Node** (Jenkins installed here)
* **Ubuntu Slave Node** (build executor)
* **CentOS Slave Node** (build executor)

The setup includes installation steps, Java/agent configuration, and sample pipelines to test distributed CI builds across multiple nodes.

---

## 📌 Architecture Overview

```
+----------------+        +-----------------+
|  Ubuntu Master | <----> |  Ubuntu Slave   |
| (Jenkins UI)   |        |  (Agent Node)   |
+----------------+        +-----------------+

                       <---->
                     +-----------------+
                     |  CentOS Slave   |
                     |  (Agent Node)   |
                     +-----------------+
```

---

## 🛠️ Step-by-Step Setup

### **Step 1: Setup Master (Ubuntu VM in GCP)**

1. Create a VM in GCP (Ubuntu).
2. Install Jenkins → Follow this guide 👉 [Jenkins Setup and Installation](https://github.com/Mahadhav1999/jenkins-setup-and-installation).

---

### **Step 2: Prepare Ubuntu Slave**

On Ubuntu slave VM:

```bash
sudo apt update
sudo apt install openjdk-21-jdk -y
java -version   # verify Java installation
```

---

### **Step 3: Prepare CentOS Slave**

On CentOS slave VM:

```bash
sudo yum install java-17-openjdk -y
# If above doesn’t work
sudo dnf install java-21-openjdk java-21-openjdk-devel -y

java -version   # verify Java installation
```

---

### **Step 4: Add Nodes in Jenkins (Master UI)**

1. Login to Jenkins → `Manage Jenkins` → `Nodes & Clouds` → `New Node`.
2. Create nodes as follows:

#### 🔹 Ubuntu Slave

* Node name: `ubuntu-slave`
* Remote root dir: `/home/jenkins`
* Label: `ubuntu`
* Launch method: *Launch agent by connecting it to the master*

#### 🔹 CentOS Slave

* Node name: `centos-slave`
* Remote root dir: `/home/jenkins`
* Label: `centos`
* Launch method: *Launch agent by connecting it to the master*

---

### **Step 5: Connect Slaves to Master**

1. From Jenkins Dashboard → `Manage Jenkins` → `Nodes` → Click on a node (e.g., `ubuntu-slave`).
2. Jenkins will show **launch commands** like below (⚠️ These are just examples — copy the command shown in your Jenkins UI, since the secret will be different).

#### 🐧 Linux (Ubuntu/CentOS)

```bash
curl -sO http://<master-ip>:8080/jnlpJars/agent.jar
java -jar agent.jar -url http://<master-ip>:8080/ -secret <SECRET_KEY> -name "ubuntu-slave" -webSocket -workDir "/home/jenkins"
```

#### 🪟 Windows

```powershell
curl.exe -sO http://<master-ip>:8080/jnlpJars/agent.jar
java -jar agent.jar -url http://<master-ip>:8080/ -secret <SECRET_KEY> -name "windows-slave" -webSocket -workDir "C:\jenkins"
```

✅ After running the command → the slave will appear **online** in Jenkins.

---

### **Step 6: Create a Pipeline Job**


1. Go to Jenkins → `New Item` → Enter job name → Select **Pipeline** → Save.
2. In **Pipeline Script** section, use one of the examples below.

## Note :
# Create the two separate jobs with pipelines for two slaves(Ex: ubuntu-slave-job & centos-slave-job)

#### Example 1: Run on Ubuntu Slave

```groovy
pipeline {
    agent none
    stages {
        stage('Build on Ubuntu Slave') {
            agent { label 'ubuntu' }
            steps {
                echo "Running on Ubuntu Slave"
                sh 'uname -a'
            }
        }
    }
}
```

#### Example 2: Run on CentOS Slave

```groovy
pipeline {
    agent none
    stages {
        stage('Build on CentOS Slave') {
            agent { label 'centos' }
            steps {
                echo "Running on CentOS Slave"
                sh 'cat /etc/*release'
            }
        }
    }
}
```

3. Save → Click **Build Now** → Check console logs.

---

## ✅ Verification

* Jobs run on **slave nodes** (not on master).
* Logs confirm execution environment (`uname -a`, `cat /etc/*release`).
* Distributed CI works successfully.

---

## 📂 Repository Structure

```
.
├── README.md   # Documentation
├── pipelines/  # Example pipeline scripts
└── setup/      # Setup guides
```

---

## 🎯 Outcome

* Jenkins Master (Ubuntu) manages jobs.
* Slave nodes (Ubuntu + CentOS) execute builds.
* Distributed CI/CD ready for scaling.

---
