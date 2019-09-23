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

## iam_privesc_by_rollback
1. Create scenario
`./cloudgoat.py create iam_privesc_by_rollback`
2. 

## iam_privesc_by_attachment
1. Create scenario
`./cloudgoat.py create iam_privesc_by_attachment`


## cloud_breach_s3
1. Create scenario
`./cloudgoat.py create cloud_breach_s3`


## ec2_ssrf
1. Create scenario
`./cloudgoat.py create ec2_ssrf`


## rce_web_app
1. Create scenario
`./cloudgoat.py create rce_web_app`


## codebuild_secrets
1. Create scenario
`./cloudgoat.py create codebuild_secrets`

## Removal
### iam_privesc_by_rollback
