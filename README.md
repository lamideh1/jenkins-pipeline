# jenkins-pipeline
1. Create a Pipeline Job in Jenkins
Open your Jenkins Dashboard.

Click New Item (left-hand side).

Give it a name:
➔ My pipeline job

Select Pipeline type.

Click OK.

2. Configure GitHub Webhook Trigger
After creating the job:

Click Configure.

Scroll down to Build Triggers section.

Tick this option:
➔ GitHub hook trigger for GITScm polling

This means whenever you push to GitHub, Jenkins will automatically build.

3. Write a Jenkins Pipeline Script
Inside the same "Configure" page, under Pipeline section:

Choose Pipeline script.

In the text box, paste this basic script (corrected for you):

groovy
Copy
Edit
pipeline {
    agent any

    stages {
        stage('Connect To Github') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/RidwanAz/jenkins-scm.git']]
                ])
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t dockerfile .'
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    sh 'docker run -itd --name nginx -p 8081:80 dockerfile'
                }
            }
        }
    }
}
✅ This does:

Connect to your GitHub repo

Build a Docker image from your project

Run a Docker container that shows the website

4. Install Docker on Jenkins Server
Jenkins cannot run Docker commands yet.
You have to install Docker on the same machine where Jenkins is running.

Steps:

Create a file called docker.sh:

bash
Copy
Edit
nano docker.sh
Paste this inside:

bash
Copy
Edit
sudo apt-get update -y
sudo apt-get install ca-certificates curl gnupg -y
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
Save and exit (Ctrl + O, then Enter, then Ctrl + X).

Make the file executable:

bash
Copy
Edit
chmod u+x docker.sh
Run the file:

bash
Copy
Edit
./docker.sh
(Note: This script is only installing the Docker GPG keys. You may still need to install the full Docker engine after this. I can show you if you want.)

5. Create a Dockerfile and index.html inside your GitHub repo
Dockerfile (you already know):

dockerfile
Copy
Edit
# Use the official NGINX base image
FROM nginx:latest
WORKDIR /usr/share/nginx/html/
COPY index.html /usr/share/nginx/html/
index.html (basic web page):

html
Copy
Edit
<!DOCTYPE html>
<html>
<head>
    <title>Hello from Jenkins + Docker</title>
</head>
<body>
    <h1>It worked! Jenkins is awesome!</h1>
</body>
</html>
✅ Push these files into your GitHub repo.

6. Open Your Website
After Jenkins runs the pipeline:

It will build the image

Run the Docker container

The container will expose port 8081.

You can visit:

arduino
Copy
Edit
http://your-jenkins-ip-address:8081
And you’ll see the website.

✅ Example:
If your server IP address is 52.90.103.244, visit:

cpp
Copy
Edit
http://52.90.103.244:8081
7. Security Group Rule (If you're using AWS EC2)
If your Jenkins server is an EC2 instance (AWS):

Go to your EC2 instance ➔ Security Groups

Add a new Inbound Rule:

Type: Custom TCP

Protocol: TCP

Port Range: 8081

Source: 0.0.0.0/0 (anywhere)

✅ This allows you to access the container publicly.

✍️ Summary Checklist:

Step	What to Do
Create Jenkins Pipeline	Named "My pipeline job"
Set Build Trigger	GitHub webhook
Install Docker	On Jenkins server
Create Project Files	Dockerfile + index.html
Push to GitHub	All project files
Run Pipeline	Jenkins will Build + Run
Open Browser	http://jenkins-ip:8081
✅ You’re done!
⚡ Extra Tip:
After the container runs, you might want to stop and remove old containers every time you re-run Jenkins (otherwise you'll get "container already exists" error).
I can show you how to fix that easily too if you want!

