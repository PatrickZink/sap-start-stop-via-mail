# sap-start-stop-via-mail

## Description

There is a shared-service account that provides the AWS WorkMail email address. A free test domain is used for this purpose, which is fully sufficient. With the help of SES rules, emails are sent to the respective S3 buckets of the connected AWS accounts. If there is only one account in which the SAP systems are located, everything can also be operated in a single account.

In the respective target accounts, there is an S3 bucket in which the email is made available by SES. The S3 bucket has a bucket policy that allows SES from the shared-service account to save the emails as a file.

Each S3 bucket is equipped with a trigger that is activated for new files under the prefix "incoming-email/." This trigger initiates the Lambda function, which first checks whether the sender of the email is on the allow list. This ensures that unauthorized individuals who have managed to find out the email address cannot also start and stop the SAP systems. The Lambda function then searches the subject line for the words 'Start' or 'Stop' and the respective SID.

Subsequently, the SAP system with the mentioned SID is started or stopped. This process manages the EC2 instances as well as the respective SAP and HANA services. The core of this automation is my adapted version, which, unlike the [original](https://github.com/aws-samples/aws-ssm-automation-for-start-stop-sap?tab=readme-ov-file#readme), sends more detailed SNS notifications to provide precise information about which system for which SID or in which account was started or stopped. This makes it easier to keep track of a multitude of SAP systems and AWS accounts.

![SAP-Start-Stop-via-Mail drawio](https://github.com/PatrickZink/sap-start-stop-via-mail/assets/70896863/90d89723-d6ac-4367-8bd4-0ca3aa5510f2)


## Installation

Unfortunately, the AWS WorkMail domain cannot be created with CloudFormation, so manual steps are required to set up the shared service account. All resources in the respective target accounts are created using a CloudFormation template.

1. Creating AWS WorkMail in Shared Service Account:
![image](https://github.com/PatrickZink/sap-start-stop-via-mail/assets/70896863/4995d2b8-5c57-40b0-b2e0-f3a67d0b2845)

2. Enter desired domain:
![image](https://github.com/PatrickZink/sap-start-stop-via-mail/assets/70896863/cc8b04b4-9a4d-4955-b5e8-b41dede91300)

3. Upload SSMStopStartSAP.yaml into an S3 bucket in the target account.
4. Create Stack with sap-start-stop-via-mail.yaml.
5. Specify input parameters for sap-start-stop-via-mail.yaml.
![image](https://github.com/PatrickZink/sap-start-stop-via-mail/assets/70896863/face20d9-a3da-4385-8a2d-27d5453d7791)
6. In the email inbox of the account entered for the "EmailforNotifications" parameter, there should be an SNS confirm subscription email received. Please confirm this.
7. Tag the EC2 instances accordingly -> [README](https://github.com/aws-samples/aws-ssm-automation-for-start-stop-sap?tab=readme-ov-file#readme)
8. (Optional) An additional optional tag "SID:related" has been implemented. With this, EC2 instances can be stopped and started. Proceed as follows:
   - Tag-Key -> SID:related -> e.g., S4H:related
   - Tag-Value -> Specify EC2 instance IDs comma-separated -> e.g., i-089c6ffd0b9b572e6,i-0b90e2791acbbd243
9. In the shared service account SES -> Email receiving -> INBOUND_MAIL -> Create Rule
   - Rule name -> CustomerName
   - Recipient conditions -> account1@sap-start-stop.awsapps.com
   - Add new action -> Deliver to S3 Bucket -> Bucket name from Step 5
   - Object key prefix -> incoming-email/
   ![image](https://github.com/PatrickZink/sap-start-stop-via-mail/assets/70896863/683a9f37-c0c2-4818-a55b-fc5a5b766c94)
   ![image](https://github.com/PatrickZink/sap-start-stop-via-mail/assets/70896863/827661ed-2b1c-4a44-8e36-c0ee8bf3ab80)
   ![image](https://github.com/PatrickZink/sap-start-stop-via-mail/assets/70896863/3be4672a-e77f-47a4-9fbf-032a575358a4)
   ![image](https://github.com/PatrickZink/sap-start-stop-via-mail/assets/70896863/a8710d56-ceb0-41fd-847a-e2f0fac9b5f4)

To deploy the solution to additional target accounts, repeat only steps 3 through 9.


