# Jenkins CI/CD Pipeline for Flask Application

This repository demonstrates a Jenkins-based CI/CD pipeline setup to automate the build, test, and deployment of a simple Flask application onto an AWS EC2 instance.

---

## 🔧 Project Structure
```bash
├── app.py # Flask application
├── test_app.py # Pytest test case
├── requirements.txt # Dependencies
├── Jenkinsfile # Jenkins pipeline configuration
└── README.md # Documentation
```

## 🚀 Objective

To set up a Jenkins pipeline that:
- Clones the Flask app repository
- Installs dependencies
- Runs unit tests using `pytest`
- Deploys the application to a staging environment (EC2)
- Sends email notifications upon success/failure

---

## 🖥️ Jenkins Pipeline Stages

1. **Clone Repository**
   - Clones the latest code from the `main` branch.

2. **Install Dependencies**
   - Uses a virtual environment and installs dependencies from `requirements.txt`.

3. **Run Tests**
   - Runs unit tests using `pytest`.

4. **Deploy to EC2**
   - Stops any existing Flask app on EC2.
   - Clones the latest code to EC2.
   - Runs `app.py` in the background.

5. **Post Build**
   - Sends an email notification for build success or failure.

---


## ⚙️ Jenkinsfile Overview

This is a Declarative Pipeline using:
- `git` for cloning
- `sh` steps for build and testing
- `sshagent` for secure EC2 deployment
- `mail` plugin for notifications

> Credentials (SSH Key) must be configured in Jenkins with ID: `shakti-ec2`

---

## 📍 Deployment Server Details

- **Platform**: AWS EC2 (Ubuntu)
- **App Port**: `5000`
- **Access**: http://<your-ec2-public-ip>:5000
- **Note**: Ensure security group allows inbound TCP on port 5000.

---

## Pipline script
```bash
pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/shakti468/-Jenkins-CI-CD-Pipeline.git'
        BRANCH = 'main'
        EC2_HOST = '65.0.109.101'
        EC2_USER = 'ubuntu'
        SSH_KEY_ID = 'shakti-ec2' // Your Jenkins SSH credentials ID
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: "${BRANCH}", url: "${REPO_URL}"
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    . venv/bin/activate
                    pytest
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent([env.SSH_KEY_ID]) {
                    sh '''
ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << 'ENDSSH'
    # Stop existing Flask app
    pkill -f app.py || true

    # Remove old app, clone fresh
    rm -rf /home/ubuntu/app
    git clone https://github.com/shakti468/-Jenkins-CI-CD-Pipeline.git /home/ubuntu/app

    cd /home/ubuntu/app

    # Set up virtual environment & install dependencies
    python3 -m venv venv
    . venv/bin/activate
    pip install --upgrade pip
    pip install -r requirements.txt

    # Run Flask app in background inside venv
    nohup bash -c '. venv/bin/activate && python3 app.py' > flask.log 2>&1 &

    echo "✅ Flask app started on EC2"
ENDSSH
                    '''
                }
            }
        }
    }

    post {
        success {
            mail to: 'shaktiranjanm827@gmail.com',
                 subject: '✅ Jenkins Build Succeeded',
                 body: 'Your Flask app was built and deployed successfully.'
        }
        failure {
            mail to: 'shaktiranjanm827@gmail.com',
                 subject: '❌ Jenkins Build Failed',
                 body: 'Build or deployment failed. Please check Jenkins logs.'
        }
    }
}

```

## Configure Jenkins Global Variables Screenshots
![image](https://github.com/user-attachments/assets/4012d8aa-4238-436b-9c57-883b499e7f7c)

![image](https://github.com/user-attachments/assets/7bbf4ee0-7e38-4590-a104-827c4145b827)

## ec2 instance 
![image](https://github.com/user-attachments/assets/1ffa0ecc-681e-40a9-9e97-975902ff9cf7)

## Poll scm 
![image](https://github.com/user-attachments/assets/32eb1b38-34d1-4e8c-905f-b05aa9398cbb)

## Pipeline Image 

![image](https://github.com/user-attachments/assets/93da5ee8-e4f7-4cd1-85c0-90b0cf22d1bc)

## Build Success
![image](https://github.com/user-attachments/assets/3afd8acc-b3bb-41af-8c6f-2100e5d0879c)

Console Output

```bash
Started by user Hero Vired

[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins
 in /var/lib/jenkins/workspace/shakti_jenkins_pipeline
[Pipeline] {
[Pipeline] withEnv
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Clone Repository)
[Pipeline] git
The recommended git tool is: NONE
No credentials specified
 > /usr/bin/git rev-parse --resolve-git-dir /var/lib/jenkins/workspace/shakti_jenkins_pipeline/.git # timeout=10
Fetching changes from the remote Git repository
 > /usr/bin/git config remote.origin.url https://github.com/shakti468/-Jenkins-CI-CD-Pipeline.git # timeout=10
Fetching upstream changes from https://github.com/shakti468/-Jenkins-CI-CD-Pipeline.git
 > /usr/bin/git --version # timeout=10
 > git --version # 'git version 2.43.0'
 > /usr/bin/git fetch --tags --force --progress -- https://github.com/shakti468/-Jenkins-CI-CD-Pipeline.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > /usr/bin/git rev-parse refs/remotes/origin/main^{commit} # timeout=10
Checking out Revision 225cf0eccdd7dba87fb3c1e2b01bb56971f97ac9 (refs/remotes/origin/main)
 > /usr/bin/git config core.sparsecheckout # timeout=10
 > /usr/bin/git checkout -f 225cf0eccdd7dba87fb3c1e2b01bb56971f97ac9 # timeout=10
 > /usr/bin/git branch -a -v --no-abbrev # timeout=10
 > /usr/bin/git branch -D main # timeout=10
 > /usr/bin/git checkout -b main 225cf0eccdd7dba87fb3c1e2b01bb56971f97ac9 # timeout=10
Commit message: "Update test_app.py"
 > /usr/bin/git rev-list --no-walk 225cf0eccdd7dba87fb3c1e2b01bb56971f97ac9 # timeout=10
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Install Dependencies)
[Pipeline] sh
+ python3 -m venv venv
+ . venv/bin/activate
+ deactivate nondestructive
+ [ -n  ]
+ [ -n  ]
+ hash -r
+ [ -n  ]
+ unset VIRTUAL_ENV
+ unset VIRTUAL_ENV_PROMPT
+ [ ! nondestructive = nondestructive ]
+ [  = cygwin ]
+ [  = msys ]
+ export VIRTUAL_ENV=/var/lib/jenkins/workspace/shakti_jenkins_pipeline/venv
+ _OLD_VIRTUAL_PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/snap/bin
+ PATH=/var/lib/jenkins/workspace/shakti_jenkins_pipeline/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/snap/bin
+ export PATH
+ [ -n  ]
+ [ -z  ]
+ _OLD_VIRTUAL_PS1=$ 
+ PS1=(venv) $ 
+ export PS1
+ VIRTUAL_ENV_PROMPT=(venv) 
+ export VIRTUAL_ENV_PROMPT
+ hash -r
+ pip install --upgrade pip
Requirement already satisfied: pip in ./venv/lib/python3.12/site-packages (25.1.1)
+ pip install -r requirements.txt
Requirement already satisfied: Flask in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 1)) (2.3.2)
Requirement already satisfied: pytest in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 2)) (7.4.0)
Requirement already satisfied: Werkzeug>=2.3.3 in ./venv/lib/python3.12/site-packages (from Flask->-r requirements.txt (line 1)) (2.3.8)
Requirement already satisfied: Jinja2>=3.1.2 in ./venv/lib/python3.12/site-packages (from Flask->-r requirements.txt (line 1)) (3.1.6)
Requirement already satisfied: itsdangerous>=2.1.2 in ./venv/lib/python3.12/site-packages (from Flask->-r requirements.txt (line 1)) (2.2.0)
Requirement already satisfied: click>=8.1.3 in ./venv/lib/python3.12/site-packages (from Flask->-r requirements.txt (line 1)) (8.2.1)
Requirement already satisfied: blinker>=1.6.2 in ./venv/lib/python3.12/site-packages (from Flask->-r requirements.txt (line 1)) (1.9.0)
Requirement already satisfied: iniconfig in ./venv/lib/python3.12/site-packages (from pytest->-r requirements.txt (line 2)) (2.1.0)
Requirement already satisfied: packaging in ./venv/lib/python3.12/site-packages (from pytest->-r requirements.txt (line 2)) (25.0)
Requirement already satisfied: pluggy<2.0,>=0.12 in ./venv/lib/python3.12/site-packages (from pytest->-r requirements.txt (line 2)) (1.6.0)
Requirement already satisfied: MarkupSafe>=2.0 in ./venv/lib/python3.12/site-packages (from Jinja2>=3.1.2->Flask->-r requirements.txt (line 1)) (3.0.2)
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Run Tests)
[Pipeline] sh
+ . venv/bin/activate
+ deactivate nondestructive
+ [ -n  ]
+ [ -n  ]
+ hash -r
+ [ -n  ]
+ unset VIRTUAL_ENV
+ unset VIRTUAL_ENV_PROMPT
+ [ ! nondestructive = nondestructive ]
+ [  = cygwin ]
+ [  = msys ]
+ export VIRTUAL_ENV=/var/lib/jenkins/workspace/shakti_jenkins_pipeline/venv
+ _OLD_VIRTUAL_PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/snap/bin
+ PATH=/var/lib/jenkins/workspace/shakti_jenkins_pipeline/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/snap/bin
+ export PATH
+ [ -n  ]
+ [ -z  ]
+ _OLD_VIRTUAL_PS1=$ 
+ PS1=(venv) $ 
+ export PS1
+ VIRTUAL_ENV_PROMPT=(venv) 
+ export VIRTUAL_ENV_PROMPT
+ hash -r
+ pytest
============================= test session starts ==============================
platform linux -- Python 3.12.3, pytest-7.4.0, pluggy-1.6.0
rootdir: /var/lib/jenkins/workspace/shakti_jenkins_pipeline
collected 1 item

test_app.py .                                                            [100%]

============================== 1 passed in 0.11s ===============================
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Deploy to EC2)
[Pipeline] sshagent
[ssh-agent] Using credentials ubuntu
$ ssh-agent
SSH_AUTH_SOCK=/tmp/ssh-Vp0IX05x4bPe/agent.384655
SSH_AGENT_PID=384658
Running ssh-add (command line suppressed)
Identity added: /var/lib/jenkins/workspace/shakti_jenkins_pipeline@tmp/private_key_15554542671315722525.key (/var/lib/jenkins/workspace/shakti_jenkins_pipeline@tmp/private_key_15554542671315722525.key)
[ssh-agent] Started.
[Pipeline] {
[Pipeline] sh
+ ssh -o StrictHostKeyChecking=no ubuntu@65.0.109.101
Pseudo-terminal will not be allocated because stdin is not a terminal.
Welcome to Ubuntu 24.04.2 LTS (GNU/Linux 6.8.0-1024-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Wed Jun  4 21:04:07 UTC 2025

  System load:  0.0               Processes:             110
  Usage of /:   34.5% of 6.71GB   Users logged in:       1
  Memory usage: 29%               IPv4 address for enX0: 172.31.13.76
  Swap usage:   0%

 * Ubuntu Pro delivers the most comprehensive open source security and
   compliance features.

   https://ubuntu.com/aws/pro

Expanded Security Maintenance for Applications is not enabled.

45 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


*** System restart required ***
Cloning into '/home/ubuntu/app'...
Requirement already satisfied: pip in ./venv/lib/python3.12/site-packages (24.0)
Collecting pip
  Using cached pip-25.1.1-py3-none-any.whl.metadata (3.6 kB)
Using cached pip-25.1.1-py3-none-any.whl (1.8 MB)
Installing collected packages: pip
  Attempting uninstall: pip
    Found existing installation: pip 24.0
    Uninstalling pip-24.0:
      Successfully uninstalled pip-24.0
Successfully installed pip-25.1.1
Collecting Flask (from -r requirements.txt (line 1))
  Using cached flask-3.1.1-py3-none-any.whl.metadata (3.0 kB)
Collecting pytest (from -r requirements.txt (line 2))
  Using cached pytest-8.4.0-py3-none-any.whl.metadata (7.7 kB)
Collecting blinker>=1.9.0 (from Flask->-r requirements.txt (line 1))
  Using cached blinker-1.9.0-py3-none-any.whl.metadata (1.6 kB)
Collecting click>=8.1.3 (from Flask->-r requirements.txt (line 1))
  Using cached click-8.2.1-py3-none-any.whl.metadata (2.5 kB)
Collecting itsdangerous>=2.2.0 (from Flask->-r requirements.txt (line 1))
  Using cached itsdangerous-2.2.0-py3-none-any.whl.metadata (1.9 kB)
Collecting jinja2>=3.1.2 (from Flask->-r requirements.txt (line 1))
  Using cached jinja2-3.1.6-py3-none-any.whl.metadata (2.9 kB)
Collecting markupsafe>=2.1.1 (from Flask->-r requirements.txt (line 1))
  Using cached MarkupSafe-3.0.2-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (4.0 kB)
Collecting werkzeug>=3.1.0 (from Flask->-r requirements.txt (line 1))
  Using cached werkzeug-3.1.3-py3-none-any.whl.metadata (3.7 kB)
Collecting iniconfig>=1 (from pytest->-r requirements.txt (line 2))
  Using cached iniconfig-2.1.0-py3-none-any.whl.metadata (2.7 kB)
Collecting packaging>=20 (from pytest->-r requirements.txt (line 2))
  Using cached packaging-25.0-py3-none-any.whl.metadata (3.3 kB)
Collecting pluggy<2,>=1.5 (from pytest->-r requirements.txt (line 2))
  Using cached pluggy-1.6.0-py3-none-any.whl.metadata (4.8 kB)
Collecting pygments>=2.7.2 (from pytest->-r requirements.txt (line 2))
  Using cached pygments-2.19.1-py3-none-any.whl.metadata (2.5 kB)
Using cached flask-3.1.1-py3-none-any.whl (103 kB)
Using cached pytest-8.4.0-py3-none-any.whl (363 kB)
Using cached pluggy-1.6.0-py3-none-any.whl (20 kB)
Using cached blinker-1.9.0-py3-none-any.whl (8.5 kB)
Using cached click-8.2.1-py3-none-any.whl (102 kB)
Using cached iniconfig-2.1.0-py3-none-any.whl (6.0 kB)
Using cached itsdangerous-2.2.0-py3-none-any.whl (16 kB)
Using cached jinja2-3.1.6-py3-none-any.whl (134 kB)
Using cached MarkupSafe-3.0.2-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (23 kB)
Using cached packaging-25.0-py3-none-any.whl (66 kB)
Using cached pygments-2.19.1-py3-none-any.whl (1.2 MB)
Using cached werkzeug-3.1.3-py3-none-any.whl (224 kB)
Installing collected packages: pygments, pluggy, packaging, markupsafe, itsdangerous, iniconfig, click, blinker, werkzeug, pytest, jinja2, Flask

Successfully installed Flask-3.1.1 blinker-1.9.0 click-8.2.1 iniconfig-2.1.0 itsdangerous-2.2.0 jinja2-3.1.6 markupsafe-3.0.2 packaging-25.0 pluggy-1.6.0 pygments-2.19.1 pytest-8.4.0 werkzeug-3.1.3
✅ Flask app started on EC2
[Pipeline] }
$ ssh-agent -k
unset SSH_AUTH_SOCK;
unset SSH_AGENT_PID;
echo Agent pid 384658 killed;
[ssh-agent] Stopped.
[Pipeline] // sshagent
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Declarative: Post Actions)
[Pipeline] mail
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS

```

## Final Result in Browser (http://65.0.109.101:5000/)
![image](https://github.com/user-attachments/assets/f355fd90-2fb0-42e3-8206-9d358e083137)

## Email Notification

![image](https://github.com/user-attachments/assets/b89a3d4b-7cba-4c7f-89ca-de531ed68326)

## 📬 Contact
For questions or support, reach out to: shaktiranjanm827@gmail.com






