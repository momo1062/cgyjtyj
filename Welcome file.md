  

# Project Title

  

In this lab, we will explore IAM Misconfigurations and how they can be exploited. This will help to give an understanding of why it is important to follow the best practices while configuring anything on the cloud.

  

**Set-Up Requirements**

  

1.Linux or Mac OS

  

2.Python 3.6+

  

3.Terraform

```

curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -

sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"

sudo apt-get update && sudo apt-get install terraform

```

4. AWS CLI

```curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

unzip awscliv2.zip

sudo ./aws/install

```

5.jq

```

apt install jq

```

  

**Creating an AWS Administrator Account**

  

 - Sign Up for an Amazon AWS Account.



  

 - After login, search for Identity & Access Management (IAM).

 ![enter image description here](https://cdn.ttgtmedia.com/rms/onlineimages/SCC_0821_Singh_AWSIAM_SS2.jpg)

 - Click on Create User.




![](https://miro.medium.com/v2/resize:fit:828/format:webp/1*mjKzk569Dtxk8uFTOKZ97g.png)

 - Set a user name and click next.

 - Select attach policies directly -> in the menu below check the box for AdministratorAcess -> next
 -![enter image description here](https://miro.medium.com/v2/resize:fit:828/format:webp/1*szq3t1W49y4GMdgOWdZG8Q.png)

 -  **Review**  all the setting once and click on  **Create User**.

 **Getting Access Key and Secret Access Key**
 - Once the user is created, click on its name.
 - ![enter image description here](https://miro.medium.com/v2/resize:fit:828/format:webp/1*muY2yYSYS6lj6QC0tgJb7A.png)

 - On the user page, switch to the **Security credentials** tab. Scroll down to find a button named **Create access key** and click on it.
 - On the next page , choose Command Line Interface and then click next button
 - Give a description if you want , then click create access key

 - Note down both: Access Key and Secret Access Key

 ![enter image description here](https://miro.medium.com/v2/resize:fit:828/format:webp/1*jKKsgCbmNpREK2KUQqAK-w.png)

**Configuring AWS Profile**

 - After creating the AWS Administrator Account use the following command to configure the AWS profile on your terminal:
```
aws configure --profile cloudgoat #give any profile name
```
![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/awsprofile.png)

 -  Check if the user is configured with the AWS access key and Secret Key to the corresponding profile.

```
aws iam get-user --profile cloudgoat
```
![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/get-user.png)

**Installing CloudGoat**

   ```
git clone https://github.com/RhinoSecurityLabs/cloudgoat.git
cd cloudgoat
pip3 install -r ./requirements.txt
```

**Configuring CloudGoat**

-   Configure cloudgoat with the AWS profile, use the following command.

```
./cloudgoat.py config profile

```

-   Whitelist the IP-Address automatically.

```
./cloudgoat.py config whitelist --auto
```
**Scenario: Vulnerable Lambda**
**Run the command:**  `./cloudgoat.py create vulnerable_lambda`
This will create the first scenario and will add the required resources in your system.

Remember , we are using CloudGoat to create a practice environment for finding vulnerabilities in the system and simulate how a "hacker" might think and act in the real world! This will demonstrate the importance of preventing misconfigurations and highlight the need for following best practices!

**Scenario Resources that will be created:**

1 IAM User  
1 IAM Role  
1 Lambda  
1 Secret

**Knowledge Check!**
1.  **AWS IAM user**: An AWS IAM user is an entity with long-term credentials (such as username and password) used to interact with AWS services securely. Users have assigned permissions that determine their access levels to AWS resources.
    
2.  **AWS IAM role**: An AWS IAM role defines a set of permissions to control which actions an AWS service, IAM user, or application can perform within AWS. Roles can be assumed by trusted entities to temporarily obtain these permissions.
    
3.  **AWS Lambda**: AWS Lambda is a serverless compute service that allows you to run code without provisioning or managing servers. It automatically scales and manages the infrastructure required to run code in response to events triggered by AWS services or custom actions.
4. **AWS Policy :** A set of rules that define permissions and access controls for AWS resources. It specifies what actions are allowed or denied on specific AWS services and resources, helping to ensure security and compliance within an AWS environment.

**About the Scenario** 

Our goal is to find the scenario’s secret. (cg-secret-XXXXXX-XXXXXX)!

In this scenario, you start as the ‘bilbo’ user. You will assume a role with more privileges, discover a lambda function that applies policies to users, and exploit a vulnerability in the function to escalate the privileges of the bilbo user in order to search for secrets.

![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/vulnerable_lambda.png)

In cloud attacks, hackers often target misconfigurations as entry points to gain access to user accounts. Once inside, they attempt to escalate their privileges, gaining higher levels of access to resources. Finally, they aim to retrieve, modify, or delete sensitive information, potentially causing significant damage or data breaches. Therefore, maintaining robust security configurations and closely monitoring access controls is crucial for mitigating such risks.

**Let's Begin!**

![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/start_vl.png)
This is the IAM user 'bilbo' that has been created , this lists all the info about the account including access keys.

-   Configure the AWS Profile for bilbo using the following command
```
aws configure --profile bilbo
```
-   Get permissions for the ‘bilbo’ user.

```
#This command will give you the ARN & full name of you user.
aws --profile bilbo --region us-east-1 sts get-caller-identity

#This command will list the policies attached to your user.Remember to add THE NAME THAT **YOU** GET
aws --profile bilbo --region us-east-1 iam list-user-policies --user-name [your_user_name]

#This command will list all of your permissions.Once again , add the policy name and user name that is displayed in YOUR terminal!
aws --profile bilbo --region us-east-1 iam get-user-policy --user-name [your_user_name] --policy-name [your_policy_name]
```
![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/vullam1.png)

(Quick info: ARN= Amazon Resource Name , a unique identifier assigned to each resource in AWS)
-   Next we will list all roles , one of which we will be able to assume. We will then use this role for **privesc (Privilege Escalation)**.

   ```
#This command will list all the roles in your account, one of which should be assumable.Choose it and use it's details in the next step. 
aws --profile bilbo --region us-east-1 iam list-roles | grep cg-

This command will list all policies for the target role
aws --profile bilbo --region us-east-1 iam list-role-policies --role-name [cg-target-role]

This command will get you credentials for the cloudgoat role that can invoke lambdas.
aws --profile bilbo --region us-east-1 sts assume-role --role-arn [cg-lambda-invoker_arn] --role-session-name [whatever_you_want_here]
```
![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/vullam2.png)

-   Configure a new  AWS profile using the AWS Credentials that have now been generated on your and are appearing on your terminal.
`aws configure --profile [name_of_your_choice]`
	
	![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/assumed_role.png)



-   Manually add the  **SessionToken**.

	```
	vi .aws/credentials
	```
	![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/session_token.png)
	  
(Quick info: In AWS, a session token is a temporary set of credentials granted to an entity, such as an IAM user or role, to access AWS services. It is typically used for short-lived sessions that require temporary access to AWS resources.)

List all the lambdas to **identify** the target (vulnerable) lambda:
```
# This command will show you all lambda functions. The function belonging to cloudgoat (the name should start with "cg-")
# can apply a predefined set of aws managed policies to users (in reality it can only modify the bilbo user).
aws --profile assumed_role --region us-east-1 lambda list-functions
```
![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/vullam3.png)
