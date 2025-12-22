# CICD with Jenkins

🎯 Goal for CICD: Use Jenkins to automate the 2-tier app deployment

<br>

---

## Session: Intro to CICD and Jenkins


### What is CI?
- continuous integration
- automatically merging code
- Triggered by:
  - Developers frequently pushing their code changes to a shared repository - this triggers CICD pipeline 
- tests are run automatically on the code before it is integrated into the main code

➕
- tests are run which helps you to identify and resolve bugs
    - detects problems early and frequently (unlike waterfall method)
  - reduces cost
  - helps to maintain a stable and functional software build

<br>

---

### What is CD?

Can mean:

1️⃣ **continuous delivery** (manual sign off/ approval)
      
  - ensures software is always in a deployable state, ready/ can be pushed production any time
  - often involves producing deployable artifact (saved somewhere and waiting to be deployed)
  - **requires manual approval / release decision**
  
<br>

 ➕ a deployable artifact is always ready and just needs manual approval to be deployed to end users
 
<br>

2️⃣ **continuous deployment** (automatically deploys code to production)
  - extends continuous delivery by automating the final step of deploying to production
  - **no manual intervention required**
  
<br>

 ➕ benefit which is also a disadvantage:
    - removes need for human approval but relies entirely on automated processes
  
<br>

---

### What is Jenkins?
- automation server
- open-source
- primarily used for CICD, but it can automate much more

### Why use Jenkins?

➕
- automate entire CICD pipeline
- reduces human error
- extensibility: Jenkins has 2000+ plug-ins that can support wide range of tools
- scalability: Jenkins server can scale easily by adding/using worker nodes or agents to run builds/ tests
- community support
- cross-platform: Windows, Linux, MacOS

➖
- complex setup for beginners 
- maintainance overhead: plug-ins and versions - getting to work and updating
- resource intensive when running multiple jobs
- user interface may be a little outdated, compared to modern tools

<br>

### Stages of Jenkins

#### A typical Jenkins CICD pipeline

🎭 **stages**
1. Source Code Management (SCM) usually from git
2. Build: compile the code, build into executable artifact
3. Test: automated tests are run (unit tests, integration tests, etc)
4. Package: into deployable artifact
5. If using cont deployment then the package is deployed into target environment .e.g test, production
6. Monitor: monitoring tools may need to be deployed and configured by pipeline, to observe performance, log issues, etc after deployment

<br>

### What alternatives are there for Jenkins?

- GitLab CI
- GitHub Actions
- CircleCI
- Travis CI
- Bamboo
- TeamCity
- GoCD
- Azure DevOps (Azure Pipelines)

<br>
  
### Why build a pipeline? Business value?

- faster time to market
- cost savings - automating repetitive processes
- reduced risk - catches things that will cause deployment failures
- developers can focus on writing code and developing/testing features, rather than deploying it
- scalability
- improved quality - catching bugs and fixing them early - end users get artifact earlier and can suggest improvements

<br>

---

## CICD pipeline structure

ADD INFO HERE ABOUT AGENT/ WORKER NODES

### Pipeline Overview

![alt text](image-32.png)

⚙️ **What does Jenkins do?**
- Receives webhook from GitHub
- Pulls latest code from dev
- Made up of master node - uses agent/worker nodes to actually carry out the jobs
- The master node (built in node / server) spins up (delegates) agent nodes that run 3 jobs:

<br>

🔹 CI = Jobs 1 & 2

🔹 CD = Job 3

<br>

1️⃣ TEST

- Developer pushes code to **dev branch**
- GitHub sends a **notification webhook** to Jenkins
- Jenkins job starts automatically
- Code tested on Jenkins worker node

2️⃣ MERGE

- Worker node checks out main branch
- New tested code **merged from dev branch to main branch**
- Main branch pushed to GitHub

3️⃣ DEPLOY

- Tested code from the worker node is **copied into the EC2 instance**
- ssh into EC2 instance and start app

<br>

🪝 **Why webhook?**
- ❌ Polling = Jenkins keeps asking GitHub “any changes?”
- ✅ Webhook = GitHub tells Jenkins immediately (faster + cleaner)

<br>

**Built in node vs worker nodes**
- it's simpler than having worker nodes and so makes it easier to get started with Jenkins
BUT
- it cannot scale up on demand
- if it crashes, no more CICD pipelines can be run
- any builds will have the same level of access to the file system as the Jenkins process

<br>

### What does a successful pipeline do?

✅ Success

- Tests pass → pipeline continues
- Tests fail → pipeline stops ❌
- 💡 This prevents broken code ever reaching main or AWS

security to build in:
JOB 1 - Jenkins needs access to github repo, so it needs private key in ssh folder - it needs this to do the merge
JOB 1 - public key needs to be on github
JOB 3 - Jenkins needs to ssh into EC2 instance so needs private key - copy private key from local machine to app - rsync of scp - then restart the app

STEPS TO DO
make separate github repo with app code
JOB 1 - Jenkins needs access to github repo, so it needs private key in ssh folder - it needs this to do the merge
JOB 1 - public key needs to be on github
get web hook set up so it can trigger job 1


<br>

---

## Code-along: Use Jenkins the first time

**Overview**
- Jenkins server is running
- No jobs yet
- Once a job runs, it is called a build
- Each job = a Jenkins project
- Linked jobs together = a pipeline


**Setup**
1. sign into Jenkins server
   - Server 2
   - Default address for Jenkins server 2: http://52.31.15.176:8080
   - Login details:
     - Username: devopslondon
     - Password: DevOpsAdmin

2. create first job

    new item > name:lucy-first-project > freestyle prject > ok to get to congigure page for the project > General >
    add description:testing jenkins > discard old builds > Max # of builds to keep: 5 > Build steps > pick execute shell > uname -a > save at the bottom

    ![alt text](image-9.png)


3. Build manually:
- Click Build Now
- Green tick = success
  Dashboard view:
  ![alt text](image-12.png)
- Job runs on a worker EC2 node

1. check console output

    ![alt text](image-13.png)
    ![alt text](image-16.png)

2. create a second job

   1. Copy existing job as a template > Name: lucy-get-date-time > Build step: `date`

    OR

   2. new item > name:lucy-get-date-time > freestyle prject > ok to get to congigure page for the project > General >
   add description:testing jenkins > discard old builds > Max # of builds to keep: 5 > Build steps > pick execute shell > date > save at the bottom

3. connect jobs 1 and 2

    **Why?**

    - to control execution order
    - to stop the pipeline if something fails

    **How?**

    lucy-first-project > configure > post build actions > build other projects > lucy-get-date-time (remove comma) >trigger only if build stable > save > click build now > check on dashboard


## 3-job Jenkins pipeline to deploy Sparta test app

### Code-along: Generate new key pair for Jenkins

**How?**
1. create an ssh key pair

    In Git Bash:
      
    ```
        cd .ssh/

        ssh-keygen -t rsa -b 4096 -C "lucyannestevenson7@gmail.com"

        name it: lucy-jenkins-2-github-key

      press enter on these:
          - Enter passphrase for "lucy-github-key" (empty for no passphrase):
          - Enter same passphrase again:

    ```

2. add public key to app repo in github

    go to app repo > settings > deploy keys > add > add public key name > cat lucy-jenkins-2-github-key.pub to get 'key' to fill in > tick allow write access 

    ![alt text](image-17.png)


**This allows Jenkins to:**
- Pull code
- Merge branches
- Push updates back to GitHub

**How does this fit into Pipeline?**
- Job 1: Uses this key to clone and test the repo
- Job 2: Uses this key to merge dev → main

<br>

---

### Job 1

🎯 Goal - Automatically run tests every time code is pushed to the dev branch

1. make new project for job 1 on Jenkins

**How?**

- Jenkins > click new item > name: lucy-sparta-app-job1-ci-test > click freestyle project > click ok > general > github project tick > https://github.com/lucystevenson/tech515-sparta-test-app-cicd.git but remove '.git' and replace with / like https://github.com/lucystevenson/tech515-sparta-test-app-cicd/ > source code management > click git > URL: git@github.com:lucystevenson/tech515-sparta-test-app-cicd.git (make sure this is SSH not HTTPS)> add credential > add > jenkins > kind: ssh username with private key > id&username: lucy-jenkins-2-github-key > description: to read/write to repo > private key > enter directly (cat lucy-jenkins-2-github-key in github - copy and paste)> add > then add this key to credentials drop down > branch > set default branch to */main > build environment > click provide node & npm bin/ folder to PATH > specify NodeJS version 20 > build steps > select execute shell > we are now in github repo > add 

  ```
  cd app
  npm install
  npm test
  ```

  ![alt text](image-19.png)

  authenticate with ssh github link
  ![alt text](image-18.png)


2. Setup the webhook to trigger Job 1

**Why are we using a webhook?**
- Instead of Jenkins constantly checking GitHub (polling)
- GitHub notifies Jenkins instantly when code is pushed

**How?**
1. get Jenkins to listen to webhook:

    On Jenkins:
    lucy-sparta-app-job1-ci-test > configure > build triggers > click 'GitHub hook trigger for GITScm polling' > save

    ![alt text](image-20.png)

2. set up notification to be sent on github repo to Jenkins

    on GitHub add webhook pointing to http://<jenkins-ip>:8080/:

    settings > webhooks > add webhooks > payload URL: http://52.31.15.176:8080/github-webhook/ > add


    ![alt text](image-21.png)

3. test webhook in Git Bash 
    ```
      18  cd Documents/github/
      19  ls
      20  cd tech515-sparta-test-app-cicd/
      21  ls
      22  nano README.md
      23  git add .
      24  git commit -m "add line to README"
      25  git push
    ```

**Outcome**

- Developer pushes code to GitHub
- GitHub sends a webhook
- Jenkins runs Job 1 automatically
- Tests pass or fail immediately

<br>

---

### Job 2

🎯 Goal - Automatically merge the dev branch into the main branch, only if Job 1 succeeds

**What has already happened in job 1?**

We have taken files that have been pushed to github on the dev branch and tested the files on a worker node

**Outcome for job 2**

Only when that job 1 is successful, we build job 2 that checks out the main branch and merges the new tested code from the dev branch to the main branch and pushes this to github

❌
- We don't need a webhook here - we will set up job 2 so it automatically builds after job 1 - NO need to tick 'github hook trigger for GITScm polling' for job 2 as we are already listening for a webhook on job 1 > if we add this here, job 2 will also be listening out for a webhook from github and if we push a change to github, job 2 will be triggered unneccesarily

- build environment > click provide node & npm bin/ folder to PATH > NO need to tick this on job 2 as we have already npm install in job 1


**Why We Did This**
- Protects the main branch
- Ensures only tested code is merged
- Removes the need for manual merges

**Problem it solves:**
- Human error during merging
- Untested code reaching production-ready branches

**How?**

1. Make project for job 2 on Jenkins

   - Jenkins > click new item > name: lucy-sparta-app-job2-ci-merge> click freestyle project > click ok

   - general > github project tick > https://github.com/lucystevenson/tech515-sparta-test-app-cicd.git but remove '.git' and replace with / like https://github.com/lucystevenson/tech515-sparta-test-app-cicd/ > source code management > click git > URL: git@github.com:lucystevenson/tech515-sparta-test-app-cicd.git (make sure this is SSH not HTTPS)> add credential > drop down > select lucy-jenkins-2-github-key > branch > set default branch to */dev

   - build environment > click SSH agent > add credentials 'lucy-jenkins-2-github-key' 
     - this loads your GitHub SSH private key into the job and enables Jenkins to write to the repo so it can do the git commands below

   - build steps > select execute shell > we are now in github repo > add 

    ```
    git fetch origin
    git checkout main
    git pull origin main
    git merge origin/dev
    git push origin main

    ```

   **What These Commands Do**
   - git fetch origin → updates all remote branches (origin/dev, origin/main, etc.)
   - git checkout main → make sure you’re on main locally
   - git merge origin/dev → merge the latest dev from GitHub into your local main
   - git push origin main → push the updated main back to GitHub

   > alternative - Jenkins Git Publisher plugin could be used instead of shell commands

    **How to add Git Publisher plugin?**
    
      Post build actions > add > git publisher > click 'push only if build succeeds', 'merge results' > branches > branch to push: main > target remote name: origin > save > make sure to remove shell commands now

      ![alt text](image-24.png)  
      ![alt text](image-26.png)


2. connect jobs 1 and 2

    **Why?**

    - Job 2 should only run if tests pass and build is stable

    **How?**

    - Dashboard > lucy-sparta-app-job1-ci-test > configure > post build actions > build other projects > lucy-sparta-app-job2-ci-merge (remove comma) >trigger only if build stable > save > click build now > check on dashboard

3. check job 1 and job 2 work

   - Create and switch to dev branch locally
   - Make a change and push to GitHub

   In Git Bash:
   ```
      27  git branch dev
      28  git switch dev
      29  nano README.md (add line here)
      30  git add .
      31  git commit -m "add line to README"
      32  git push
   ```

**Outcome**
- GitHub webhook triggers Job 1
- Job 1 runs tests
- If successful, Job 2 runs
- dev is merged into main

<br>

---

### Job 3

🎯 Goal for job 3 - copying the already-tested code that exists in the Jenkins workspace to your EC2 instance and running app ec2 instance

**What has already happened in jobs 1 and 2?**

We have taken files that have been pushed to github on the dev branch, tested the files on a worker node and then, only when that is successful, we checkout the main branch and merge the new code from the dev branch to the main branch and push this to github

**Outcome for job 3**

We need to take the tested code from the worker node and copy this into the EC2 instance, then ssh into EC2 instance and run commands to start up app

- rsync code → EC2
- ssh into EC2
- install deps (if needed)
- restart app (pm2 / node)


**How?**

1. create an AWS EC2 instance

- Sparta test app using Node JS 20 needs Ubuntu 22.04 LTS
- Use normal SG rules, but allow from anywhere (to get it working initially) OR Jenkin worker nodes SG to allow Jenkins to SSH in (for better security):
security group should have these inbound rules:
![alt text](image-27.png)

- Must have all dependencies / user data installed to run your app:

```
#!/bin/bash

# Update system
sudo apt update -y
sudo DEBIAN_FRONTEND=noninteractive apt upgrade -yq

# Install Nginx
sudo apt install -y nginx
sudo systemctl enable nginx
sudo systemctl start nginx

# Install Node.js 20
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Install PM2 globally
sudo npm install -g pm2
```

2. make project for job 3

- Jenkins > click new item > name: lucy-sparta-app-job3-cd-deploy> click freestyle project > click ok

- general > github project tick > https://github.com/lucystevenson/tech515-sparta-test-app-cicd.git but remove '.git' and replace with / like https://github.com/lucystevenson/tech515-sparta-test-app-cicd/ 

- source code management > tick git > URL: git@github.com:lucystevenson/tech515-sparta-test-app-cicd.git > Credentials: lucy-jenkins-2-github-key > Branch: */main

⚠️ This is mandatory — every job that needs files must check out the repo

- build environment > click SSH agent > credentials > add > jenkins > kind: SSH username with private key > description: private key to SSH into EC2 instance > username: tech515-lucy-aws.pem > private key > enter directly > copy and paste private key in > add > then select private key you have just added 'tech515-lucy-aws.pem'
  - we need to add the private key credentials on Jenkins for Job 3 to access our EC2 instance

- build steps > select execute shell >
  - this copies the tested code from Jenkins worker node to ec2 instance, then we ssh into the ec2 instance and run the app

I originally tried this code:

```
rsync -av --delete . ubuntu@<EC2_PUBLIC_IP>:/home/ubuntu/sparta-test-app

ssh ubuntu@<EC2_PUBLIC_IP> << 'EOF'
  cd /home/ubuntu/app
  npm install
  pm2 stop app.js
  pm2 start app.js
EOF
```

but I came across a blocker - I needed to stop Strict Host Key Checking so I updated my code to this:

```
rsync -av --delete -e "ssh -o StrictHostKeyChecking=no" \
  ./ ubuntu@3.254.77.197:/home/ubuntu/app/

ssh -o StrictHostKeyChecking=no ubuntu@3.254.77.197 << 'EOF'
  cd /home/ubuntu/app/app
  npm install
  pm2 restart all || pm2 start app.js
EOF
```

**Why is rsync better than scp?**
rsync is better than scp because it only transfers changed files, whereas scp copies everything unnecessarily

3. connect jobs 2 and 3

    **Why?**

    - Job 3 should only run if job 2 works and build is stable

    **How?**

    - lucy-sparta-app-job2-ci-merge > Configure > add post-build Actions > Build other projects > projects to build > add lucy-sparta-app-job3-cd-deploy > Trigger only if build is stable > save

4. test pipeline

In Git Bash:

`cd tech515-sparta-test-app-cicd/` should be on dev branch

cd into views folder

`cd app/views/`

Update line 'Change via Jenkins CICD pipeline on 16/12/25 12:25' in 'index.ejs' file with current time:

`nano index.ejs`

![alt text](image-23.png)

push change to github

```
git add . stage change
git commit -m "change front page 9:22"
git push
```

ADD PICTURE OF SPARTA APP HOMEPAGE HERE!!!


---
To deliver: Document with Markdown ready to provide the link to it by **TO FILL IN**

Use principles for good documentation such as easy to navigate, readability
Include (you decide on the order):
- diagram to explain your CICD pipeline
- why we setup the CICD pipeline the way we did, benefits you have seen, benefits for an organisation
- how you setup each of your jobs (including authentication/security), webhook & how the pipeline is triggered, what the result should be at the end

<br>

---

## Links to using SSH authentication with a repo on GitHub
* Link to [SSH doc](SSH/README.md)

