# Jenkins + Terraform CI Setup

This README explains how to configure Jenkins to automatically run **Terraform Apply on every push** to your GitHub repository, and later run **Terraform Destroy manually**.

---

## 📌 Overview

You will set up:

* A GitHub repository containing Terraform code
* Jenkins pipeline job triggered on **every push**
* AWS credentials stored securely in **GitHub Secrets**
* Jenkinsfile that performs:

  * Clone repo
  * Terraform Init
  * Terraform Validate
  * Terraform Apply (auto on push)
  * Terraform Destroy (manual only)

---

## 🛠️ 1. Create AWS Access Key & Secret Key

1. Go to **AWS Console** → *IAM*
2. Choose the user that Jenkins will use
3. Go to **Security Credentials**
4. Create **Access key**
5. Copy the:

   * `AWS_ACCESS_KEY_ID`
   * `AWS_SECRET_ACCESS_KEY`

---

## 🔑 2. Add AWS Keys to GitHub Secrets

Go to your GitHub repo → **Settings → Secrets → Actions → New Repository Secret**

Add:

* `AWS_ACCESS_KEY_ID`
* `AWS_SECRET_ACCESS_KEY`

This keeps your credentials secure.

---

## 🔧 3. GitHub Webhook for Jenkins

Go to:
**GitHub → Repo → Settings → Webhooks → Add webhook**

Use:

* **Payload URL** → `http://<JENKINS_PUBLIC_IP>:8080/github-webhook/`
* **Content type** → `application/json`
* **Trigger** → Just the push event

---

## 🔧 4. Jenkins Plugin Requirements

Install these plugins:

* **GitHub Integration Plugin**
* **GitHub API Plugin**
* **Pipeline Plugin**

---

## 🧩 5. Jenkins Credentials Setup

Go to:
**Jenkins → Manage Jenkins → Credentials → Global → Add Credentials**

Add AWS credentials here:

* Kind: **Secret Text** → Access Key
* Kind: **Secret Text** → Secret Key

(or use environment variables inside pipeline)

---

## 📄 6. Jenkinsfile (Terraform Apply on Push)

Place this `Jenkinsfile` in your GitHub repo:

```groovy
pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        AWS_ACCESS_KEY_ID = credentials('aws-access-key')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
    }

    stages {

        stage('Clone Repo') {
            steps {
                git url: 'https://github.com/USERNAME/REPO.git', branch: 'main'
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }

        stage('Terraform Validate') {
            steps {
                sh 'terraform validate'
            }
        }

        stage('Terraform Apply') {
            steps {
                sh 'terraform apply -auto-approve'
            }
        }
    }
}
```

**This pipeline runs automatically for every push.**

---

## 🧨 7. Manual Terraform Destroy Job

Create another pipeline job called **terraform-destroy**.

Use this `Jenkinsfile-destroy`:

```groovy
pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID = credentials('aws-access-key')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
    }

    stages {
        stage('Clone Repo') {
            steps {
                git url: 'https://github.com/USERNAME/REPO.git', branch: 'main'
            }
        }

        stage('Terraform Destroy') {
            steps {
                sh 'terraform destroy -auto-approve'
            }
        }
    }
}
```

👉 **You will run this manually from the Jenkins UI.**

---

## ✔️ Summary

| Operation         | Trigger                 | Action                            |
| ----------------- | ----------------------- | --------------------------------- |
| Terraform Apply   | Automatic (GitHub Push) | Apply infra every push            |
| Terraform Destroy | Manual                  | Destroy infra whenever you choose |

---

## 🙌 Done!

You now have:

* Secure AWS key handling
* Automatic apply pipeline
* Manual destroy job
* Clean CI workflow with GitHub → Jenkins → AWS

Feel free to customize and improve!
