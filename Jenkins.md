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
- Add the code from previous app repo on gitub to this new repo.
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
- In git bash:
  - `nano README.md`
  - Make a change to it and save.
  - `git status`
  - `git add .`
  - `git commit -m "dev branch test`
  - `git push origin dev`
- Should see the job run on jenkins automatically and be successful.

### Merging changes from dev to main if job is successful: Job 2:

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
    - Branch = main.
  - In build environment: enable ssh agent and specify credentials. 
  - Add a build step - execute shell script
```
git checkout main
git pull origin main
git merge --ff-only origin/dev
git push origin main

```
Blockers:
- Didn't enable SSH agent on the job.
![alt text](<Images/Screenshot 2025-02-06 163731.png>)