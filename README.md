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
5. Install CloudGoat *this was taken from (https://github.com/RhinoSecurityLabs/cloudgoat)*
```bash
git clone git@github.com:RhinoSecurityLabs/cloudgoat.git ./CloudGoat
cd CloudGoat
pip3 install -r ./core/python/requirements.txt
chmod u+x cloudgoat.py
```

## iam_privesc_by_rollback

## iam_privesc_by_attachment

## cloud_breach_s3

## ec2_ssrf

## rce_web_app

## codebuild_secrets

## Removal
