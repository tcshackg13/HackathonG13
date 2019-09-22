# Group 13 - DevOps Project

09.22.2019

**─**

- Sarath Kunala
- Vishal Aswani
- Rahul Vanmali
- Ankita Chhabra
- Isabel Hernandez

# Overview

This Project is to Execute End to End CICD Pipeline on provided Eureka Micro Service provided as part of TCS DevOps Hackathon 2019

# Goals

- Challenge:1         -  All 3 Linux Centos 7 Servers (Jenkins, Nexus &amp; Sonarqube) are provisioned with respective Ansible Playbooks. Also all dependencies required for all projects are part of this Ansible Playbooks
- Challenge:2          - Configure Jenkins &amp; Docker services
- Challenge:3        - Execute end to end Jenkins Pipeline for CI &amp; CD through Groovy DSL / Jenkinsfile
- Challenge:4        - Automate All above challenges wherever applicable.
- Challenge:5        - Setup Stackdriver dashboard for GCP Project.

# Pre-requisites:

- Created GCP Account, Project
- Created 4 VMs and made sure SSH keys are transferred for ansible to do ssh.
- Created 1 Kubernetes cluster.
- Updated Firewall Rules to allow ports so that we can access applications externally.
  - tcp:8080; tcp:9000; tcp:8081; tcp:8761; tcp:8082; tcp:8083; tcp:9990; tcp:9090

| **Instance Name** | **Type** | **IP** |
| --- | --- | --- |
| client-instance | VM | Internal : 10.142.0.6 External: 34.74.233.242 |
| ccid-instance | VM | 10.128.0.5 35.232.99.117 |
| artifactory-instance | VM | 10.128.0.635.232.19.37 |
| testing-instance | VM | 10.142.0.4 (nic0) 35.243.222.63 |
| g13-k8s-cluster | Kubernetes Cluster (3 Node) | g13-k8s-cluster |
| Github - Ansible Code | Github | [https://github.com/tcshackg13/tcshackg13](https://github.com/tcshackg13/tcshackg13)  tcshackg13 / Tcs@hackg13 |
| Github - Application Code | Github | [https://github.com/tcshackg13/eureka-devops-g13](https://github.com/tcshackg13/eureka-devops-g13) |

# Solution Approach

![project.jpeg](https://github.com/tcshackg13/tcshackg13/blob/master/images/project.jpeg)

# Steps - Challenge 1

Install Ansible &amp; Git on Client Instance :
sudo yum install ansible
Sudo yum install git

Created Playbook Roles &amp; corresponding yml files as per below configuration :

- Dependencies
- Docker
- Jenkins
- Nexus
- Sonarqube

Jenkins Playbook will call Dependencies (maven, java,git,unzip,wget ), Docker (docker, docker-compose) , Jenkins Roles (sonar scanner , kubectl)
Nexus Playbook will call Dependencies, Docker, Nexus Roles
Sonarqube Playbook will call Dependencies, Docker, Sonarqube Roles

Refer to Zip file provided or below Git Repo for Playbook Roles :
https://github.com/tcshackg13/tcshackg13

**On Client Instance :**

Step1: Added below host entry  in /etc/ansible/hosts
[jenkins]
10.128.0.5
[sonarqube]
10.142.0.4
[nexus]
10.128.0.6

Step2 : Execute ansible playbooks with verbose command
ansible-playbook jenkins-playbook.yml -vv
ansible-playbook nexus-playbook.yml -vv
ansible-playbook sonar-playbook.yml -vv

As part of ansible execution note Jenkins Admin password &amp; Nexus Admin password which will be available in standard output printed.

Step3 : Access all 3 instances through respective URLs

Jenkins : [http://35.232.99.117:8080](http://35.232.99.117:8080/)
Sonarqube: [http://35.243.222.63:9000](http://35.243.222.63:9000/about)
Nexus: [http://35.232.19.37:8081/](http://35.232.19.37:8081/)

Step4 : Login to Respective application and configure.

# Steps - Challenge 2

Configuring Nexus , jenkins for Pipeline build
**Nexus :**

- Create a user &quot;jenkins/jenkins&quot; with nx-admin role
- Create a Maven2 - hosted Repository, &quot;g13project&quot;
- Create a Docker - hosted repository, &quot;g13docker&quot;. Add 8083 port as HTTP connector.
- Create a Docker - Proxy , &quot;g13dockerhub&quot; add it to Docker Group repo.
  - Note: Add url as &quot;https://registry-1.docker.io&quot;
- Create a Docker - Group Repo, &quot;g13dockergroup&quot; and add above docker hosted repo and docker proxy repo to this group. Add 8082 port as HTTP connector.

**Jenkins :**

- Login with Admin password and setup Jenkins with all basic plugins
- Create first time admin user (jenkins/jenkins)
- Create Credentials for Nexus
- Install Nexus Artifact Uploader - Plugin
- Create Pipeline Job as required to pull eureka-sample project provided.



# Challenge 3

Setup &quot;Jenkinsfile&quot; Pipeline Script, Kubernetes Manifest files, Dockerfile in eureka sample project.

[https://github.com/tcshackg13/eureka-devops-g13](https://github.com/tcshackg13/eureka-devops-g13)

Note: These files are also available under &quot;references&quot; folder in the zip file or git repo [https://github.com/tcshackg13/tcshackg13](https://github.com/tcshackg13/tcshackg13)

Jenkins Configuration :

- Create Jenkins Pipeline job &quot;tcshackg13-pipeline-job&quot; pointing to Jenkinsfile in [https://github.com/tcshackg13/eureka-devops-g13](https://github.com/tcshackg13/eureka-devops-g13)
- Select Github hook trigger for GITscm polling
- Create webhook on Github pointing to jenkins server for push events http://35.232.99.117:8080/github-webhook/ (push)

Execute a commit on Github for the sample project and automatically Jenkins Pipeline should start executing, we can monitor popeline status and also respective Application URLs.

Pipeline Execution :

Nexus Repo :

Sonar Scanner :

Dockerized Application (for QA/Test): [http://35.232.99.117:8761/](http://35.232.99.117:8761/)

Eureka Server (Microservice) Project : [http://35.233.132.69:8761/](http://35.233.132.69:8761/) (On Kubernetes Cluster)

# Challenge 4

Execute master playbook to provision all the softwares in single click.
ansible-playbook master-playbook.yml -vv

Solution Approaches :

1. Use Ansible to install &amp; configure jenkins based on below :
  - XML files for Jenkins Config, Jobs, Credentials.
  - Copy the core XML files along with other dependent XML files from the source system folder  &quot;/var/lib/jenkins/&quot; to target system.

1. Using Jenkins CLI to create credentials, build /create jobs/plugins etc...
  - Automate Ansible execution of scripts and then subsequently Jenkins CLI commands can be executed to perform end to end environment.

# Challenge 5

Created a Simple Stackdriver Monitoring Dashboard to monitor Project / related servers &amp; clusters.
[https://app.google.stackdriver.com/dashboards/5851650109277982146?project=g13-devops-project](https://app.google.stackdriver.com/dashboards/5851650109277982146?project=g13-devops-project)
