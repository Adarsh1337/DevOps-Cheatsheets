# Jenkins Cheatsheet

## Table of Contents

- [Overview](#overview)
- [Installation](#installation)
- [Basic Configuration](#basic-configuration)
- [Pipeline Syntax](#pipeline-syntax)
- [Common Commands](#common-commands)
- [Plugins](#plugins)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## Overview

Jenkins is an open-source automation server that enables developers to build,
test, and deploy their software reliably. It is a leading CI/CD tool that
supports building and testing projects continuously.

**Key Features:**

- Extensible plugin architecture (1800+ plugins)
- Distributed builds across multiple machines
- Pipeline as Code (Jenkinsfile)
- Easy installation and configuration
- Support for all major version control systems

## Installation

### Docker Installation

```bash
# Pull Jenkins LTS image
docker pull jenkins/jenkins:lts

# Run Jenkins container
docker run -d -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  --name jenkins jenkins/jenkins:lts

# Get initial admin password
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### Ubuntu/Debian Installation

```bash
# Add Jenkins repository
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | \
  sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

# Install Jenkins
sudo apt-get update
sudo apt-get install jenkins

# Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

## Basic Configuration

### Configure Global Security

1. Navigate to **Manage Jenkins > Configure Global Security**
2. Enable security realm (e.g., Jenkins own user database)
3. Set authorization strategy (Matrix-based security recommended)
4. Configure CSRF protection

### Manage Credentials

```groovy
// Add credentials programmatically
import jenkins.model.Jenkins
import com.cloudbees.plugins.credentials.*
import com.cloudbees.plugins.credentials.domains.Domain

def credentials = new UsernamePasswordCredentialsImpl(
  CredentialsScope.GLOBAL,
  "my-credential-id",
  "Description",
  "username",
  "password"
)

SystemCredentialsProvider.getInstance().getStore()
  .addCredentials(Domain.global(), credentials)
```

## Pipeline Syntax

### Declarative Pipeline

```groovy
pipeline {
    agent any

    environment {
        APP_NAME = 'my-application'
        VERSION = '1.0.0'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/user/repo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh './deploy.sh'
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
            mail to: 'team@example.com',
                 subject: "Failed: ${currentBuild.fullDisplayName}",
                 body: "Build failed: ${env.BUILD_URL}"
        }
    }
}
```

### Scripted Pipeline

```groovy
node {
    try {
        stage('Checkout') {
            git 'https://github.com/user/repo.git'
        }

        stage('Build') {
            sh 'mvn clean package'
        }

        stage('Test') {
            sh 'mvn test'
        }

        stage('Deploy') {
            if (env.BRANCH_NAME == 'main') {
                sh './deploy.sh'
            }
        }
    } catch (e) {
        currentBuild.result = 'FAILED'
        throw e
    } finally {
        // Cleanup steps
        cleanWs()
    }
}
```

## Common Commands

### Jenkins CLI

```bash
# Download Jenkins CLI
wget http://localhost:8080/jnlpJars/jenkins-cli.jar

# List jobs
java -jar jenkins-cli.jar -s http://localhost:8080/ \
  -auth admin:password list-jobs

# Build a job
java -jar jenkins-cli.jar -s http://localhost:8080/ \
  -auth admin:password build job-name

# Get job status
java -jar jenkins-cli.jar -s http://localhost:8080/ \
  -auth admin:password get-job job-name

# Safe restart
java -jar jenkins-cli.jar -s http://localhost:8080/ \
  -auth admin:password safe-restart
```

### Groovy Script Console

```groovy
// List all jobs
Jenkins.instance.getAllItems(AbstractItem.class).each {
    println it.fullName
}

// Disable all jobs
Jenkins.instance.getAllItems(AbstractProject.class).each {
    it.disabled = true
    it.save()
}

// Delete old builds
Jenkins.instance.getAllItems(Job.class).each { job ->
    job.builds.findAll { build ->
        build.number < 100
    }.each { build ->
        build.delete()
    }
}
```

## Plugins

### Essential Plugins

| Plugin | Purpose | Installation |
|--------|---------|-------------|
| Pipeline | Pipeline as Code support | Pre-installed |
| Git | Git SCM integration | Pre-installed |
| Docker Pipeline | Docker commands in Pipeline | `docker-workflow` |
| Blue Ocean | Modern UI for Pipelines | `blueocean` |
| Credentials Binding | Bind credentials to variables | `credentials-binding` |
| Kubernetes | Deploy to Kubernetes | `kubernetes` |

### Install Plugins via CLI

```bash
# Install a plugin
java -jar jenkins-cli.jar -s http://localhost:8080/ \
  -auth admin:password install-plugin plugin-name

# Restart after installation
java -jar jenkins-cli.jar -s http://localhost:8080/ \
  -auth admin:password safe-restart
```

## Best Practices

### Pipeline Best Practices

1. **Use Declarative Pipeline**: Easier to read and maintain
2. **Version Control Jenkinsfile**: Store with application code
3. **Use Shared Libraries**: Reuse common pipeline code
4. **Implement Proper Error Handling**: Use try-catch blocks
5. **Keep Pipelines Fast**: Parallelize when possible

```groovy
// Parallel execution example
stage('Parallel Tests') {
    parallel {
        stage('Unit Tests') {
            steps {
                sh 'npm run test:unit'
            }
        }
        stage('Integration Tests') {
            steps {
                sh 'npm run test:integration'
            }
        }
    }
}
```

### Security Best Practices

1. **Enable CSRF Protection**: Prevent cross-site request forgery
2. **Use Role-Based Access Control**: Limit permissions appropriately
3. **Secure Credentials**: Never hardcode secrets
4. **Enable Audit Logging**: Track all user actions
5. **Regular Updates**: Keep Jenkins and plugins updated

### Performance Optimization

```groovy
// Use lightweight checkout for large repositories
checkout([
    $class: 'GitSCM',
    branches: [[name: 'main']],
    extensions: [[$class: 'CloneOption', shallow: true, depth: 1]],
    userRemoteConfigs: [[url: 'https://github.com/user/repo.git']]
])
```

## Troubleshooting

### Common Issues and Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| Build hangs | Resource exhaustion | Increase executor count |
| Plugin conflicts | Incompatible versions | Check plugin compatibility |
| Out of memory | Insufficient heap | Increase `-Xmx` in startup |
| Slow performance | Too many builds retained | Configure build discarder |
| Authentication fails | Incorrect credentials | Reset admin password |

### Reset Admin Password

```bash
# Stop Jenkins
sudo systemctl stop jenkins

# Edit config.xml
sudo vim /var/lib/jenkins/config.xml

# Set useSecurity to false
<useSecurity>false</useSecurity>

# Restart Jenkins
sudo systemctl start jenkins

# Login without password, then reconfigure security
```

### Check System Logs

```bash
# View Jenkins logs
sudo tail -f /var/log/jenkins/jenkins.log

# Docker logs
docker logs -f jenkins

# Check system info
curl -u admin:password http://localhost:8080/systemInfo
```

### Debug Pipeline

```groovy
// Enable debug output
pipeline {
    options {
        timestamps()
        ansiColor('xterm')
    }
    stages {
        stage('Debug') {
            steps {
                script {
                    echo "BUILD_ID: ${env.BUILD_ID}"
                    echo "JOB_NAME: ${env.JOB_NAME}"
                    echo "WORKSPACE: ${env.WORKSPACE}"
                    sh 'env | sort'
                }
            }
        }
    }
}
```

## Additional Resources

- [Official Documentation](https://www.jenkins.io/doc/)
- [Pipeline Syntax Reference](https://www.jenkins.io/doc/book/pipeline/syntax/)
- [Plugin Index](https://plugins.jenkins.io/)
- [Jenkins Community](https://www.jenkins.io/participate/)
- [Best Practices Guide](https://www.jenkins.io/doc/book/pipeline/best-practices/)

---

**Last Updated**: October 2025
**Version**: Jenkins 2.x LTS
