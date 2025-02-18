# Setting up a jenkins server with AWS

## Create a key pair and move it to .ssh directory on local pc. 

1. On AWS console, go to key pairs. Create a .pem key pair (private key will be downloaded).
2. Move the private key to the .ssh folder:
   - `mv /c/Users/zaina/Downloads/aws-key-zainab.pem /c/Users/zaina/.ssh`
3. Create your ec2 instance and attach the public key you created on AWS to the ec2. 


## Creating the Jenkins servers

1. Create EC2 instance:
   - Ubuntu 22.04 LTS
   - Key pair that you previously created.
   - t3 micro. (too small need to upgrade version to t3.small)

2. SSH into the EC2 instance you created
   - ` ssh -i /c/Users/zaina/.ssh/zainab-ssh-key.pem ubuntu@34.245.236.120`

3. Configure the jenkins server on the EC2
- Run these commands:

```
# Update
sudo apt update

# Install java (pre-requisite for jenkins)
sudo apt install openjdk-17-jre

# Check if it's installed properly
java --version
 
# Install the dependencies for jenkins 
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# Update (with new jenkins dependencies)
sudo apt-get update

# Install jenkins
sudo apt-get install jenkins

# Permenantly start jenkins service
sudo systemctl start jenkins.service

# Check status of jenkins (should be running)
sudo systemctl status jenkins

```
![alt text](<Images/Screenshot 2025-02-17 142618.png>)

- Jenkins runs on port 8080 so open that port in the inbound rules of security group of ec2. 

![alt text](<Images/Screenshot 2025-02-17 143016.png>)

- Use grep to see if jenkins is running on port 8080.

![alt text](<Images/Screenshot 2025-02-17 142556.png>)

4. Access the jenkins server through public Ip of EC2 and port 8080 at the end: 
   - `http://18.200.197.9:8080/`

5. To get the password, use cat and the file path given on the jenkins screen:
![alt text](<Images/Screenshot 2025-02-17 142607.png>)

   - Paste the contents into the jenkins server to access it.

- Install suggested plugins.
- Skip and use as admin. 

## Creating the pipeline on my own jenkins server:

- REMEMBER! All the changes are made in the git bash terminal on your local pc to the repo you created for this app. The changes are pushed to github for anything to happen. 
- Start up the app ec2 that was already created previously in 2 tier architecture. Make sure app is running.
- `ssh -i /c/Users/zaina/.ssh/zainab-ssh-key.pem ubuntu@34.245.236.120`


## Creating credentials for jenkins to be able to access github

- In local git bash terminal
- `ssh-keygen -t rsa -b 4096 -C "zainabfarooq001@gmail.com"`
- Name: `zainab-jenkins-2-github-sapp`

Private key needs to be on the jenkins server (needs to access github).

Public key on the github.

### Putting public key on github

1. Just putting the padlock (public key) on one repo not all the github account. Go to your cicd repo with app code.
2. Go to the repo settings
3. Click deploy keys - add key.
  - Put the same name as the key `zainab-jenkins-2-github-sapp`
  - In git bash, `cat zainab-jenkins-2-github-sapp`
  - Paste the contents into the github key box.
  - Allow write access.
  
If you try to add the same key to padlock another repo on the same github account, it won't let you do it.

Each repo needs a different key.

![alt text](<Images/Screenshot 2025-02-06 140006.png>)

## Setting up Job 1 + giving jenkins access

- Enter Jenkins:
  - `http://52.31.15.176:8080/`

- First install nodejs plugin in plugins section.
- Then go to tools section and install nodejs version.  
- Need to also restart the jenkins server
- `sudo cat /var/lib/jenkins/secrets/initialAdminPassword` to get password. username = admin. 

![alt text](<Images/Screenshot 2025-02-18 115414.png>)

`npm install -g eslint@7.32.0 webpack@5.65.0 typescript@4.4.3`

Dashboard:

- Create item:
  - Name: `tech501-zainab-job1-ci-test`
  - Discard old builds - max 5
  - Description - CI with github webhook

![alt text](<Images/Screenshot 2025-02-06 143333.png>)

  - Check the box called Github project
    - Enter your github repo url in the box (https version).
    - `https://github.com/zainabx78/tech501-sparta-app-cicd.git`
    - Remove `.git` from the end and add a `/`
    - `https://github.com/zainabx78/tech501-sparta-app-cicd/`
  - Source code management - `Git`
    - The URL needed here will be the SSH url.
    - `git@github.com:zainabx78/tech501-sparta-app-cicd.git`
    - Message will show failed.
    - Need to add a key
    - Add - Jenkins
      - Kind: SSH username with private key
      - ID: same as what key is called on local machine `zainab-jenkins-2-github-sapp`
      - Description - `To read and write to the repo`
      - Username: Same as ID `zainab-jenkins-2-github-sapp`
      - Private key: Enter directly. Paste contents of the private key into the box. Use `cat` command. 
      - ADD.
      - If theres an error, enter the jenkins server (ssh) and use `ssh -T git@github.com` or ssh add. 
  
![alt text](<Images/Screenshot 2025-02-06 141638.png>)

- Click your own key from the drop down now.
  - Branches to build: */main
  - In build environment:
    - Check the box `Provide Node & npm bin/ folder to PATH`
    - Node version 20
  - Build steps:
    - Execute shell
    ```
      cd app
      npm install
      npm test
    ```
  - Save
  - Build now.
  - Check console output.
  - If job hangs, just abort it and retry.
  
# Setting up a webhook (for jenkins to listen to github changes)

## Setting up webhook (specify dev branch).

- In already made job - configuration
- In branches to build: 
  - `*/dev`
  - Only if dev branch exists but leave it as main for now.
- In build triggers:
  - Check the box `GitHub hook trigger for GITScm polling`


## Setting up github side webhook:

- Github- your app repo
- settings - webhooks
- Add webhook:
  - Payload Url: 
    - The jenkins server ip goes in this.
    - `http://52.31.15.176:8080/github-webhook/`
    - Disable SSL verification
    - Save.

## Testing the webhook: 

- In git bash terminal:
  - Make changes to your README.md file `nano`.
  - Push the changes.
  - The changes should automatically start running the job on jenkins if it works properly.

# Job 2:

- Install ssh agent plugin in jenkins plugins
- Can use a jenkins plugin. 
- Plugin called git publisher.
- Before using this plugin, change config (if used previous method for job 2 using the shell script):
  - In job 2 configuration:
    - Remove the build step.
    - add a post build action : git publisher.
    - Check `push only if build succeeds` box.
    - Check `force push` box.
    - Add branch: 
      - branch to push = main
      - Target remote branch = origin
      - Save.
      - Also check the merge box. 
![alt text](<Images/Screenshot 2025-02-07 121934.png>)

  - Branch specifier = dev
  - Also check the `ssh agent` box in `build environment`

![alt text](<Images/Screenshot 2025-02-07 122024.png>)

  - Save
- Then in local bash terminal, make changes to your README.md file in the dev environment in repo.
  - Push the changes to remote repo on github.
  
  ![alt text](<Images/Screenshot 2025-02-07 120045.png>)
  ![alt text](<Images/Screenshot 2025-02-07 120105.png>)


- Should see job 1 and 2 run and the changes should also be pushed to the github main branch from the dev branch (check on github both branches).
  
![alt text](<Images/Screenshot 2025-02-07 120547.png>)


## JOB 3 - Deploying code on EC2 instance


- ssh into ec2 previously created in 2-tier app:
  - `ssh -i /c/Users/zaina/Downloads/zainab-ssh-key.pem ubuntu@54.216.167.202`

### Create Item:
- Name Job 3: zainab-job3-cd-deploy 
- Configure the git repo links. 
- Add ssh agent- with credentials
- Add job- add a execute shell job:
- The app folder location in jenkins: 
  - In job2 shell script add `pwd` and run the job to check where my app folder is on jenkins.
  - `/var/jenkins/workspace/zainab-job2-ci-merge/app`
- Also add a trigger so it watches job 2 before running.

- scp command to copy code from jenkins to the ec2 instance:
`scp -o StrictHostKeyChecking=no -r /var/jenkins/workspace/zainab-job2-ci-merge/app ubuntu@ec2-54-216-167-202.eu-west-1.compute.amazonaws.com:/home/ubuntu`
  - Possibility that files are locked and scp won't copy files to the folder.
    - May need to stop app running first (not good for production).
    - Best way is to update code and re-run it (no interruption to the users):
    - E.g. canary deployment - 10% of user traffic redirected to a different server for testing purposes and start slowly shifting the traffic.
    - E.g. Blue-green deployment - replica of servers to be tested on - switching between the 2. E.g. only make the switch from traffic going to blue to green when the green servers work properly with the changes. Problem with green, switch back to blue. 
- Ssh into ec2 through jenkins:
  - `ssh ubuntu@ec2-54-216-167-202.eu-west-1.compute.amazonaws.com`


![alt text](<Images/Screenshot 2025-02-07 161820.png>)

Output with just the first 2 commands (scp and ssh)- can see successful ssh into the ec2.

![alt text](<Images/Screenshot 2025-02-07 161244.png>)



Turn this into a script for the jenkins job shell script:
```
scp -o StrictHostKeyChecking=no -r /var/jenkins/workspace/zainab-job2-ci-merge/app ubuntu@ec2-54-216-167-202.eu-west-1.compute.amazonaws.com:/home/ubuntu

ssh ubuntu@ec2-54-216-167-202.eu-west-1.compute.amazonaws.com

```
Next, update script to run app through jenkins:

```
#This copies files from jenkins server to the ec2 - If you run scp again with an updated file, it will replace the existing file with the new one.


scp -o StrictHostKeyChecking=no -r /var/jenkins/workspace/zainab-job-2/app ec2-34-245-236-120.eu-west-1.compute.amazonaws.com:/home/ubuntu/app

ssh ubuntu@ec2-34-245-236-120.eu-west-1.compute.amazonaws.com << 'EOF'
cd /home/ubuntu/app
sudo systemctl restart nginx
export DB_HOST=mongodb://172.31.52.149:27017/posts
npm install
pm2 kill
pm2 start app.js
EOF 

```
