# Introduction to Jenkins

This repository holds the details of actions needed to be performed while doing the L3 'An Introduction to Jenkins' by Shariq Mustaquim - sharimus@

## Install Jenkins

1. Change region to Sydney (ap-southeast-2)
2. Create a KeyPair
3. Clone my GitHub repo: <TODO>
4. Run the following Command to create the stack:
   
         aws cloudformation create-stack --region ap-southeast-2 --capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_NAMED_IAM --template-body file://template.yml  --stack-name introToJenkins --parameters ParameterKey=KeyName,ParameterValue=<key_name>
5. Run the following command to find the public ip

         aws cloudformation describe-stacks --region ap-southeast-2 --query "Stacks[*].Outputs[?OutputKey=='JenkinsMasterURL'].OutputValue" --output text
6. Navigate to Public IP
7. Run the following command to find the SSH command to connect to the server:

         aws cloudformation describe-stacks --region ap-southeast-2 --query "Stacks[*].Outputs[?OutputKey=='JenkinsMasterSSH'].OutputValue" --output text

8. 'cd' to directory where you save the private 'pem' key and run the command from the above step to SSH into the EC2 instance. 
9. Find the password in the EC2 instance:
   $ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
10. Add the password to Jenkins UI and click continue
11. Accept default plugins <TODO>


## Set up Credential for the Build Agent

1. On the Home page, Click Credentials > System > 'Global credentials (unrestricted)' > Add Credentials
2. Select Kind as 'SSH Username with private key'
3. Type the following:

   - Username: **ec2-user**
   - Private Key: **Enter directly**
   - Key: Copy/Paste the **Private key** created in 'Install Jenkins' Step #2.

## Connect Jenkins Master to the Build Agent

1. Login to the Jenkins UI
2. Click 'Build Executor Status' on the home page 
3. Click 'Configure' (Gear Icon) and set '# of executors' to 0, then click 'Save', you will see no executor under 'Build Executor Status' now.
4. Click 'Build Executor Status' on the home page again
5. Click 'New Node', in Node name enter 'build-agent-1', select 'Permanent agent' and click 'OK'
6. Type the following on the next page:

   - Description: **Amazon Linux 2**
   - \# of executors: **4** 
   - Remote Root Directory: **/home/ec2-user**
   - Labels: **beanstalk**
   - Launch method: **Launch slave agents via SSH**
      - Host: (Type the result retrieved from running the following command to find the Private Dns name of Build agent):

            aws cloudformation describe-stacks --region ap-southeast-2 --query "Stacks[*].Outputs[?OutputKey=='JenkinsBuildAgentPrivateDnsName'].OutputValue" --output text
      - Credentials: **ec2-user**
      - Host key Verification Strategy: **Non verifying  Verification Strategy**

   Click 'Save'

7. Go to Home Page again and click 'build-agent-1' from section 'Build Executor Status', then click the 'Log' link to confirm agent was successfully connected by looking for a log entry at end:

         "Agent successfully connected and online"


### Connect Jenkins to GitHub

1. Login and navigate to the Jenkins UI
2. Click 'Manage Jenkins' from the left hand menu and click 'Configure System'
3. Locate the 'GitHub' section 
4. Click 'Advanced' button
5. From 'Additional actions' drop-down, select 'Convert login and password to token'
6. Click the radio button 'From login and password'
7. Enter your GitHub username and password, then click 'Create token credentials'
8. You should see a confirmation 

         Created credentials with id <id...>
9. Click 'Save'

_(Optional: Confirm Jenkins created a Personal Access Token in GitHub)_

9. Open a browser tab and login to your GitHub account
10. Click on your Profile Picture in top right of page 
11. Click Settings > Developer Settings > Personal Access Tokens
12. Confirm 'Jenkins Github Plugin token' exists
13. Click and view the scopes (i.e permissions) defined for the Token
14. Leave the GitHub browser window open
    
### Create a Code Repository for Application

1. Open a browser tab and login to your GitHub account (if not already logged in)
2. Go to the following URL: https://github.com/shariqmus/eb-python
3. Click 'Fork' button on the top right to fork the repo in your own account
4. Go to 'Settings' under the repository (with Gear Icon)
5. Click Webhooks from left side navigation
6. Click 'Add webhook'
7. Run the following command on a terminal to obtain the WebHook URL:
   
         aws cloudformation describe-stacks --region ap-southeast-2 --query "Stacks[*].Outputs[?OutputKey=='JenkinsGitHubWebHookURL'].OutputValue" --output text
8. Copy the URL obtained from above command output to 'Payload URL' field
9.  Click 'Add webhook' button
10. Leave the browser window open

### Create a Jenkins project to deploy to Elastic Beanstalk

1. Login/Navigate to Jenkins homepage
2. Click 'New Item'
3. Enter item name as 'my-eb-app'
4. Enter description as "Deploy to beanstalk"
5. Select the check-box 'Restrict where this project can be run' and type: 'beanstalk' (should match the label on the build agent configured)
6. Click 'Source Code Management' tab:
   - Select Git radio-button and copy the clone URL of the repository forked in 'Create a Code Repository' for Application Step#3 
   - In 'Additional Behaviours', click 'Add' and select 'Check out to specific local branch'
7. Click 'Build Triggers' tab:
   - Select 'GitHub hook trigger for GITScm polling'
8. Click 'Build' tab: 
   - Click 'Add build step' > 'Execut shell'
   - Type the following:

         # Initialize beanstalk application
         eb init my-eb-app --platform python-3.6 --region ap-southeast-2
         # Use environment named 'DevelopmentEnvironment'
         eb use DevelopmentEnvironment
         # deploy new version
         eb deploy
         # report on deployment status
         eb health
         eb status

### Commit a change to code and confirm Jenkins is invoked for Build

1. Open a browser tab and login to your GitHub account (if not already logged in)
2. Go to the 'eb-python' repository in your account
3. Click 'application.py' file
4. Click the pencil icon on the top-right of the editor window to edit the file
5. Search for word 'Congratulations'
6. Update it to say: 'Congratulations - _your name_'
7. Click 'Commit changes'
8. Navigate to Jenkins homepage and confirm a build is started as soon as the commit was performed.
9. Check the Console output under the Build in Jenkins by clicking build number (e.g. '#2') and the 'Console Output'
10. Navigate to Beanstalk environment URL and confirm your changes were deployed:
      
         aws cloudformation describe-stacks --region ap-southeast-2 --query "Stacks[*].Outputs[?OutputKey=='EBEnvironmentURL'].OutputValue" --output text

### Cleanup

1. Finally delete the stack to cleanup the resources:
   
         aws cloudformation delete-stack --region ap-southeast-2 --stack-name introToJenkins

