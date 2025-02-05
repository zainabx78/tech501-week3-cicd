# Creating 2-tier architecture on AWS


## Creating the DB VM on AWS:

Make sure you're in Ireland region!
- Name: tech501-zainab-db-vm
- AMI = ubuntu 22.04 LTS
- Security group:
  - Make sure ssh (port 22) is allowed in sg. 
  - Also need port 27017 (mongodb) allowed.
- Don't change defaults of vpc or sg.
- Create a new key pair:
  - Name: zainab-ssh-key
  - Downloads private key to your local pc.
  - Move it to ssh folder.

## Creating App VM

Make sure you're in Ireland region!
- Name: tech501-zainab-app-vm
- AMI = ubuntu 22.04 LTS
- Make sure ssh (port 22) is allowed in sg. 
- Also need port 80 (HTTP).
- Need port 3000 allowed too.
- Don't change defaults of vpc or sg.
- Use existing key pair created for db vm.

### SSH into the app vm:
- `ssh -i /c/Users/zaina/Downloads/zainab-ssh-key.pem ubuntu@54.73.45.82`

### Set up dependencies for the app:

- `sudo apt-get update -y` or `sudo apt update -y` same thing.
  - Safe command- good way to check internet access. Doesn't actually upgrade anything yet!
- `sudo apt-get upgrade -y`.
  - When purple confirmation screen shows up, user input still required- press tab and enter to press ok.
- `sudo apt install nginx -y`
  - More user input required- press tab and enter.
- `sudo systemctl status nginx` - check status of nginx.

Dependencies= anything that's required for the application to run.

**NodeJS installation-**

```
sudo DEBIAN_FRONTEND=noninteractive bash -c "curl -fsSL https://deb.nodesource.com/setup_20.x | bash -" && \
sudo DEBIAN_FRONTEND=noninteractive apt-get install -y nodejs
```

***** Make sure to copy this command from the markdown language terminal not the preview******

To check if it's installed:
node -v and npm -v. If you see the versions of these, means it's installed.

## SSH into the DB VM

- `ssh -i /c/Users/zaina/Downloads/zainab-ssh-key.pem ubuntu@54.73.45.82`
- Run update and upgrade commands:
  - `sudo apt update -y` `sudo apt upgrade -y`

## Installing MongoDB:

- Mongo db version 7.0.6
- Install gnupg and curl:
  - `sudo apt-get install gnupg curl`
- Download gpg key:
  - `curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \
   --dearmor`
- Create file list:
  - `echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list`
- Update:
  - `sudo apt-get update`
- Install a specific release: 7.0.6
  - `sudo apt-get install -y mongodb-org=7.0.6 mongodb-org-database=7.0.6 mongodb-org-server=7.0.6 mongodb-mongosh mongodb-org-mongos=7.0.6 mongodb-org-tools=7.0.6`
- Check status:
  - `sudo systemctl status mongod`
  - By default it starts turned off.
- Start mongodb:
  - `sudo systemctl start mongod`
- Check status:
  - `sudo systemctl status mongod`
    - Should be running. 

### Change BindIP:

Enter this:
- `sudo nano /etc/mongod.conf`
- Change the bind IP to 0.0.0.0
- 0.0.0.0 means it's accessible from anywhere.
- Don't do this for production.
- The default bind IP only allows local host to access the db (the db's own vm).

![alt text](<Images/Screenshot 2025-02-05 145932.png>)

### Restart mongodb server after making change:

- `sudo systemctl restart mongod`
- `sudo systemctl status mongod`
  - Should be running.

## In the db VM:
-  `sudo systemctl is-enabled mongod`
   -  Will see the db isn't enabled. That means it won't start on start up of VM when VM is stopped and started.
-  `sudo systemctl enable mongod`
   -  Makes sure it starts up on start of VM everytime.
   -  Doesn't start db just with enable this time. But will do every other time when you start VM.
-   `sudo systemctl start mongod`
    -   Still need to start it this time.
-   `sudo systemctl status mongod`
    -   Should be running.

## In the App VM:

- cd into the app folder and then `npm start` to run the app.
- In browser- open the app with `PublicIP:3000`.
  - Should be working!

## Create Environment Variable: APP VM
*** Need to do this everytime I restart vm or stop and start it.
- `export DB_HOST=mongodb://10.0.3.4:27017/posts`
  - Use the private IP of db in this command.
  - 27017 is the default port for mongodb.

Check if you have set it up correctly:
- `printenv DB_HOST`

Create dummy records:
- `npm install`
    - Clears the db and seeds it (adds records).
    - Also checks for db connection.
    - Also checks for app vulnerabilities.
Start the App again and see db records connected:
- `npm start`
- Enter `PublicIpOfAppVM:3000/posts` to see the connected db.

*** If you need to re-seed db and it doesn't work:
npm install might not populated db occasionally- may need to manually run a command to seed db:
- `node seeds/seed.js` 
- Need to be in app folder in app vm. 

## Connecting App VM to Db VM:

- Everytime you deploy/start up a vm from an image, to connect it to db you have to set the environment variable again and also seed the db:
  - `cd ~/repo/app`
  - `export DB_HOST=mongodb://10.0.3.4:27017/posts`
  - `printenv DB_HOST`
  - `npm install`
  - `npm start`


## Creating a reverse proxy:
In the app vm:

- nano into this file to edit the path- 
  - `sudo nano /etc/nginx/sites-available/default`
- Create a backup of the file before editing it.
  - `sudo cp -r /etc/nginx/sites-available/default /etc/nginx/sites-available/default-backup`
- Add this proxy pass into the file in the **location** section:
  - `proxy_pass http://localhost:3000;`
  - Make sure to take out the line already under location- 
    - Take out the line starting with `try_files.`
  
Should look like this at this point:
```bash
location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.

                proxy_pass http://localhost:3000;
}
```

- Restart the nginx app- 
  - `sudo systemctl restart nginx`
- Start the app- `npm start`
- Can also run app with `node app.js`
- nginx will redirect traffic 
  - we will no longer need to put 3000 at the end of the link. 
  - `http://20.39.216.155/` Just the Ip on port 80 will take us to the port 3000 link without specifying :3000.

## Running the app in the background: Using PM2 and &
1. Using pm2:
   - Install pm2 on the app vm:
     - ` sudo npm install -g pm2`
     - The -g flag means it's installed globally on your system.
     - Available to all users.
   - Start the pm2 process for your app:
     - ` pm2 start app.js --name "my-app"`
   - Check the status of the process:
     - `pm2 status`
   - To stop the app:
     - `pm2 stop "my-app"`
     - `pm2 delete`
   - To restart the app:
     - `pm2 restart "my-app"`
   - Logs:
     - `pm2 logs "my-app"`
2. Using the `&` command:
    - `npm start &`
      - Even when you quit the port running terminal, the app will still be running.
    - `jobs`- to see the background processes running.
      - Should see the app.
    - See the job ID:
      - `jobs -l`
    - Kill the process:
      - `kill -15 <jobID>`
        - The medium level graceful termination.
    - To restart just run `npm start &` again.