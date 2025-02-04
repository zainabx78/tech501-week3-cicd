# Using SSH instead of HTTP for github



Usually- using https to authenticate push local repo to github.

## Using SSH to authenticate and push local repo to github.

- SSH key pair - public and private parts.

1. Generate key pair.
2. Register public key on azure.
3. Register private key
4. Create a test repo on github.
5. Push changes. 

### Creating the key pair:
- `ssh-keygen -t rsa -b 4096 -C "zainabfarooq001@gmail.com"`
- In the path enter `/c/Users/zaina/.ssh/zainab-github-key`
  - This will be the name of your key.
  - 
### Registering public key on azure

In github:
1. settings
2. ssh and gpg keys
3. new ssh key
4. Name: same as keypair created: zainab-github-key.
5. In local terminal: 
  - `cat zainab-github-key.pub`.
6. Paste content into the box on github.
7. The key is for all the repos in my account.

### Registering private key with ssh agent

- eval `ssh-agent -s`
  - Should output a process id.
  - ssh agent running.
- `ssh-add zainab-github-key`
- `ssh -T git@github.com`
  - Should output successful authentication.

![alt text](<Images/Screenshot 2025-02-04 151456.png>)

### Create a test repo (on github and local)

- In github:
  - Create a test repo
  - Name: tech501-test-ssh
  - Public.

In terminal:
- `cd /documents/github`
- Create a new repo with same name:
  - `mkdir tech501-test-ssh`
- `cd tech501-test-ssh`
- `git init`
- `echo "# tech501-test-ssh" > README.md`
- `git remote add origin git@github.com:zainabx78/tech501-test-ssh.git`
- `git add .`
- `git branch -M main`
- `git commit -m "test"`
- `git push -u origin main`
  - Need to run this command only first time to push. 
  
![alt text](<Images/Screenshot 2025-02-04 155022.png>)