# Jenkins

- Free to use.
- Powerful plugins.
- Great to understand CICD.

# LAB:
Enter IP with port for jenkins server already created.
- `http://52.31.15.176:8080/`
- Username: devopslondon
- Password: **Confidential**


## First project in Jenkins:
- Creating a project = same as creating a pipeline.
- Create a new item (new pipeline):
  - Name: zainab-first-project
  - Freestyle project.
  - Click Ok. 
  - Description - Testing Jenkins.
  - Discard old builds option - Asks how much history you want to keep everytime the pipeline runs (build= instances of everytime the pipeline runs). 
    - Max builds = 5.
  - Add build step: 
    - Execute shell
    - `uname -a` - Gives you details about the OS you  use.
  - Save.
- Click **Build now** on the left. 
- In the dashboard, the job you created will be in a build queue. This will be until worker nodes are spun up to run the jobs.
- When worker nodes are idle for a specific amount of time, they are removes by jenkins.
- To review your job, click on the #1 in build history --> console output.
  - Shows details of what happened when the job ran. 
  - Shows the uname command running and the output of that.
  
![alt text](<Images/Screenshot 2025-02-05 113546.png>)

## Creating a new pipeline which outputs date and time:
1. Create a new Item.
    - Free style project
    - Add build step - execute shell
    - Add command `date`.
2. Save.
3. Build now.
4. Check in console output - should see the date and time. 

![alt text](<Images/Screenshot 2025-02-05 114159.png>)

## Create a multi-stage pipeline
You can link jobs together (the pipelines you created) and they can run one after the other to create a multi-stage pipeline.

1. Go to your first project job.
2. Post-build actions:
   - build other projects.
   - specify the `zainab-get-date` project.
   - Make sure no comma or space at the end.
   - Trigger only if build is stable (next job will only run if first is successful).
3. Build now (first project).
4. In console output of first build #2, can see that it triggered the next project. 
   
![alt text](<Images/Screenshot 2025-02-05 115439.png>)
5. In console output of 2nd project, can see it was triggered by the first project. 
   
![alt text](<Images/Screenshot 2025-02-05 115607.png>)

# Jenkins - CICD pipeline for Sparta Test App

Make sure your ec2 instance with your app is running.
 - If restarting ec2 to re-attempt lab, use `npm install` and `pm2 start` in the app folder.

- Developer working.
- Trigger - developer pushes the code to the remote repo (github) (dev may need to pull changes from main repo first to get indentical environments for testing purposes).
  - Git = distributed version control system.
  - Staging files = what goes into the next commit.
  - Commit = snapshot of files as they are right now - nothing moves from local machine with a commit. 
- This trigger/git push happens on the dev branch.
- Github sends a notification to jenkins.
  - In the form of a webhook.
  - Jenkins is listening for changes.

### Jenkins is made up of:
  - Master node (jenkins can be set up with only master node and it can do all jobs for you but its dangerous - it should only organise and manage pipelines. Can't afford failure on master node.)
  - Agent nodes - actually execute the jobs. Used by the master node to run jobs.

### Jobs:
  - **Job1** - Test the code - exists in dev branch so needs to be tested in the dev branch.
    - If results is successful, then job 2 carried out.
  - **Job 2** - Merge - dev to main branch.
    - If successful, move to job 3.
  - **Job 3** - deploy the code to  EC2 instance.

1. Need to setup webhooks.
2. Set up credentials- jenkins needs SSH credentials so it can access github code.
   - for pulling the code and being able to merge the changes (job 2).
3. Jenkins also needs AWS key credentials to deploy code onto AWS EC2. Needs to be able to authenticate using the private key. 

**Continuous deployment** = straight to production ---> to the end users.
**Continuous delivery** = Usually to a test environment first. 

# Creating the CICD Pipeline


![alt text](<Images/Screenshot 2025-02-06 114353.png>)



## Setting up a new repo with app code.

- Create a new repo on github:
  - Name: tech501-sparta-app-cicd
  - Public
- In local pc, create a new repo:
  - `cd ~/Documents/github`
  - `mkdir tech501-sparta-app-cicd`
- Add the code from previous app repo on github to this new repo.
  - `cd tech501-sparta-app-cicd`
  - ` git clone https://github.com/zainabx78/tech501-sparta-app repo`
  - Unzip the code `unzip nodejs20-sparta-test-app.zip `
- Delete the zip code using `rm`.
- `git init`
- `git branch -M main`
- `git push -u origin main`
- Set remote origin:
  - Delete .ssh folder from previous repo due to clone `rm -rf .git`
  - Set a new origin for the new repo:
    - `git remote set-url origin https://github.com/zainabx78/tech501-sparta-app`
    - `git remote add origin https://github.com/zainabx78/tech501-sparta-app-cicd.git`

- `git add .`
- `git commit -m "created app folder"`
- `git push -u origin main`


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

![alt text](<Images/Screenshot 2025-02-06 144919.png>)

## Setting up webhook (specify dev branch).

- In already made job - configuration
- In branches to build: 
  - `*/dev`
  - Only if dev branch exists but leave it as main for now.
- In build triggers:
  - Check the box `GitHub hook trigger for GITScm polling`

### Setting up github side webhook:

- Github- your app repo
- settings - webhooks
- Add webhook:
  - Payload Url: 
    - The jenkins server ip goes in this.
    - `http://52.31.15.176:8080/github-webhook/`
    - Disable SSL verification
    - Save.

### Testing the webhook: 

- In git bash terminal:
  - Make changes to your README.md file `nano`.
  - Push the changes.
  - The changes should automatically start running the job on jenkins if it works properly.


## Creating a dev branch:

- Change the branch to dev on jenkins.
  - In job config - change branch specifier to `*/dev` from main.
- In local git bash terminal - repo 
- `git checkout -b dev`
- Could also do 2 commands separate (`git branch dev` + `git checkout dev`) or `git switch dev`.
- In git bash:
  - `nano README.md`
  - Make a change to it and save.
  - `git status`
  - `git add .`
  - `git commit -m "dev branch test`
  - `git push origin dev`
- Should see the job run on jenkins automatically and be successful.

## JOB 2 - Merging changes from dev to main if job is successful:

- Create a new item:
  - Name: `zainab-job2-ci-merge`
  - Build triggers:   
    - Build after other projects are built  
    - Enter name of job1: `tech501-zainab-job1-ci-test`
    - trigger only if build is stable.
    - In source code management:
      - add git url and credentials
      - ssh link.
      - `git@github.com:zainabx78/tech501-sparta-app-cicd/`
    - Branch = dev
  - In build environment: enable ssh agent and specify credentials. 
    - because our repo was public, read only access i.e. cloning for job1 does not require ssh access. But job2 requires write access so needs ssh authentication.
  - Add a build step - execute shell script
```bash
git checkout main # switches to the main branch
git pull origin main # Fetches the latest changes from the remote main branch and merges them into your local main branch.
git merge origin/dev # Merge changes from origin/dev into main.
git push origin main # Pushes the updated main branch to the remote repository.

```
- If you want to merge dev into main, you have to be in main and then do git merge dev

### 2nd way to merge: the better way:

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

scp -o StrictHostKeyChecking=no -r /var/jenkins/workspace/zainab-job2-ci-merge/app ubuntu@ec2-54-216-167-202.eu-west-1.compute.amazonaws.com:/home/ubuntu/repo

#ssh into the ec2 server through jenkins. 

ssh ubuntu@ec2-54-216-167-202.eu-west-1.compute.amazonaws.com << 'EOF'
	cd /home/ubuntu/repo/app
  sudo systemctl restart nginx
  npm install
  pm2 stop app.js
  pm2 start app.js
EOF

```

![alt text](<Images/Screenshot 2025-02-07 165723.png>)

![alt text](<Images/Screenshot 2025-02-10 141032.png>)

![alt text](<Images/Screenshot 2025-02-07 165735.png>)
![alt text](<Images/Screenshot 2025-02-07 165741.png>)




### Making a change to the front page of the file
- Make sure in dev branch (LOCAL GIT REPO IN YOUR MACHINE!).
- Go to app ---> views folder ---> nano into index file ---> add whatever text you want after the h2 line.

![alt text](<Images/Screenshot 2025-02-07 170455.png>)


- `git add .`
- `git commit -m "test"`
- `Git push origin dev`

After the push it should automatically trigger the job 1 to run then job 2 and job 3. The changes will display on the app publicIP url (tried 2 times).

### **Success output in job 3-**

![alt text](<Images/Screenshot 2025-02-10 145621.png>)

### **Success shown on the app website:**

![alt text](<Images/Screenshot 2025-02-07 165651.png>)
![alt text](<Images/Screenshot 2025-02-07 170335.png>)

Remember!! EC2 public Ip changes everytime the instance is restarted.

## Connecting the database with the app in pipeline
 - Add export command into the bash script of job 3:

```
scp -o StrictHostKeyChecking=no -r /var/jenkins/workspace/zainab-job2-ci-merge/app ec2-54-216-133-229.eu-west-1.compute.amazonaws.com:/home/ubuntu/repo

ssh ec2-54-216-133-229.eu-west-1.compute.amazonaws.com << 'EOF'
	cd /home/ubuntu/repo/app
    sudo systemctl restart nginx
    export DB_HOST=mongodb://172.31.52.149:27017/posts
    npm install
    pm2 kill
    pm2 start app.js
EOF 

```

Make sure you change db priv Ip after restarting.

![alt text](<Images/Screenshot 2025-02-10 122730.png>)

![alt text](<Images/Screenshot 2025-02-10 133243.png>)

## Blockers:

Didn't enable SSH agent on the job.

![alt text](<Images/Screenshot 2025-02-06 163731.png>)

## Best practices

### If a job fails:
- Check console output
- Notifications about failure of the job set up.

### Creating jobs:
- Use multiple jobs and connect them for a single pipeline (not a single job).
- Use jenkins plugins.
- Make sure manual attempts work first.

## Key info:

Single node - jobs run on one node. (built-in node). - simplest set up. 

If use multi-node set up - master node needs permissions to run ec2 instances as worker nodes and also remove and delete them.

Advantages of CI: Detect issues early
Built-in security and monitoring at the beginning. 

Required dependencies for jenkins server:
- Install java (jdk).
- Accessible on port 8080.
- Plugins that the CICD pipeline requires.

**Biggest value to the business** = end users being able to use those changes - code sitting around brings no value, has to actually be used on the website/application.