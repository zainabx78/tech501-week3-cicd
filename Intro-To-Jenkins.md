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

Cloud- 

create key pair- for ssh
change instance in to ireland region
- manually create everything
- start with db vm
document how to do all this on aws