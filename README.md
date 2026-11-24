AWS Automated Threat Detection & Response
 Project Overview
This project demonstrates a mid-level cloud security automation architecture. Instead of relying on manual intervention to respond to threats, this solution uses an event-driven pipeline to automatically remediate compromised IAM credentials.

When a threat is detected (simulated via an AWS GuardDuty event), an AWS Lambda function is triggered to immediately disable the compromised IAM Access Key, preventing further unauthorized access.

 Architecture & Components
The project was implemented in an AWS Sandbox environment and consists of the following workflow:

The Target (IAM): Created a mock compromised-user with an active Access Key.

The Brain (AWS Lambda): Created a Python 3.12 function (AutoRemediateIAMKey) using the boto3 SDK to interact with the IAM service.

The Permission (IAM Role): Configured the Lambda execution role with IAMFullAccess to allow it to modify user credentials.

The Trigger (Simulation): Simulated an AWS GuardDuty Finding event to test the automation.

 The Python Remediation Code
Below is the exact Python code used to parse the threat and disable the key:

python
import boto3

def lambda_handler(event, context):
    # 1. Extract the username and access key ID from the event
    finding = event['detail']
    username = finding['resource']['accessKeyDetails']['userName']
    access_key_id = finding['resource']['accessKeyDetails']['accessKeyId']
    
    # 2. Connect to IAM
    iam = boto3.client('iam')
    
    # 3. Disable the compromised key
    try:
        iam.update_access_key(
            UserName=username,
            AccessKeyId=access_key_id,
            Status='Inactive'
        )
        print(f"SUCCESS: Disabled key {access_key_id} for user {username}")
        return "Remediation Successful"
    except Exception as e:
        print(f"Error: {e}")
        raise e
🧪 Verification & Testing
To prove the automation works, I ran a simulation using a test event containing a real Access Key ID (AKIAUU335IFLZKJHYCUT) attached to the user compromised-user.

1. Lambda Execution Success:
The Lambda function successfully parsed the event and executed the Python code.
(Insert Screenshot of the green "Execution results: succeeded" box here)

2. Automated Remediation Confirmed:
The AWS IAM Console confirms that the Access Key status was automatically changed to Inactive.
(Insert Screenshot of the IAM page showing "AKIAUU335IFLZKJHYCUT - Inactive" here)

 Security Context
This project highlights the concept that "Identity is the new perimeter."

In a real-world scenario, this automated pipeline (triggered by Amazon GuardDuty and EventBridge) reduces the "Breakout Time" (the time an attacker has access) from hours or days down to milliseconds. This is a core component of a modern DevSecOps and Cloud Incident Response strategy.<img width="1361" height="607" alt="Screenshot 2026-11-24 122838" src="https://github.com/user-attachments/assets/93ae6d0a-3931-45fe-bc4a-08d801894ae1" />
<img width="1358" height="607" alt="Screenshot 2026-11-24 122742" src="https://github.com/user-attachments/assets/665a5a11-25b6-444a-b825-4eb1903b3bd4" />
<img width="1364" height="604" alt="Screenshot 2026-11-24 122457" src="https://github.com/user-attachments/assets/dd4efa5e-5233-4bec-b790-67913d9409c8" />
