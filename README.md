<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Threat Detection with GuardDuty

**Project Link:** [View Project](http://learn.nextwork.org/projects/aws-security-guardduty)

**Author:** Abdulrahman Abdulkadir  
**Email:** abdabdulkadir62@gmail.com

---

![Image](http://learn.nextwork.org/confident_turquoise_quiet_hyena/uploads/aws-security-guardduty_sm42x3y4)

---

## Introducing Today's Project!

### Tools and concepts

 The services I used were Amazon S3, AWS CloudShell, IAM, GuardDuty, and AWS CLI. Key concepts I learnt include how to extract and use temporary credentials, configure AWS CLI profiles, simulate an attack using stolen credentials, detect malicious activity using GuardDuty, enable Malware Protection for S3, and test it using an EICAR file. I also gained insight into how anomaly detection works in cloud security.


### Project reflection

This project took me approximately 3 hours to complete. The most challenging part was waiting for GuardDuty to detect and report the malicious activity, as it required patience and careful verification. It was most rewarding to see GuardDuty successfully identify both the unauthorized access and the simulated malware upload, confirming my setup was secure and functioning as expected.


 I did this project today to deepen my hands-on understanding of AWS security tools like GuardDuty and S3 Malware Protection. Yes, this project absolutely met my goals—it helped me simulate real-world attack scenarios, understand how detections are generated, and see how AWS responds to threats in practice. It’s a great way to build skills that are directly relevant for cybersecurity roles.

---

## Project Setup

To set up for this project, I deployed a CloudFormation template that launches an intentionally vulnerable web application based on OWASP Juice Shop. The three main components are:
1. Web App Infrastructure – An EC2 instance runs the Juice Shop app. It includes networking resources like a VPC, subnets, internet gateway, route tables, load balancer, and auto-scaling group to host and manage the app.
2. S3 Bucket for Storage – A bucket stores a file named `important-information.txt` simulating sensitive data, which the EC2 instance can access.
3.GuardDuty for Security Monitoring– GuardDuty is enabled automatically to monitor and detect potential threats or suspicious activity during the later simulated attacks.


The web app deployed is called Juice Shop. To practice my GuardDuty skills, I will simulate attacks on this intentionally vulnerable app, such as stealing credentials or accessing sensitive data, and then observe how GuardDuty detects and reports these threats.

GuardDuty is a threat detection service by AWS that continuously monitors for malicious or unauthorized behavior to help protect AWS accounts, workloads, and data. In this project, it will detect and alert on suspicious activity when we simulate attacks on the vulnerable web application.


![Image](http://learn.nextwork.org/confident_turquoise_quiet_hyena/uploads/aws-security-guardduty_n1o2p3q4)

---

## SQL Injection

The first attack I performed on the web app is SQL injection, which means manipulating input fields to run unauthorized SQL commands in the backend database. SQL injection is a security risk because it can expose, modify, or delete sensitive data, and in some cases, allow full access to the application's database.


My SQL injection attack involved entering ' or 1=1;-- into the email field of the login form. This means I manipulated the SQL query so that the condition always returns true, effectively bypassing the need for a valid username and password and gaining unauthorized access to the admin portal.

![Image](http://learn.nextwork.org/confident_turquoise_quiet_hyena/uploads/aws-security-guardduty_h1i2j3k4)

---

## Command Injection

Next, I used command injection, which is an attack where malicious commands are inserted into a program that then executes them on the server. The Juice Shop web app is vulnerable to this because it doesn't properly sanitize user input before processing it. This allows attackers to run unauthorized system commands, potentially exposing files, leaking sensitive data, or gaining further control over the server environment.


To run command injection, I entered malicious JavaScript code into the username field of the admin profile. The script abuses unsanitized input handling to trick the server into executing system commands. It fetches IAM credentials from the EC2 metadata endpoint, then saves them to a public location in the app. The script runs with the server’s permission level, exposing sensitive AWS credentials. The result proves a critical vulnerability.

![Image](http://learn.nextwork.org/confident_turquoise_quiet_hyena/uploads/aws-security-guardduty_t3u4v5w6)

---

## Attack Verification

To verify the attack's success, I visited the public path where the credentials were saved: /assets/public/credentials.json. The credentials page showed me sensitive IAM credentials from the EC2 instance, including access keys and tokens. This confirmed that the command injection worked, and the server executed my malicious script as intended, exposing AWS environment secrets.

![Image](http://learn.nextwork.org/confident_turquoise_quiet_hyena/uploads/aws-security-guardduty_x7y8z9a0)

---

## Using CloudShell for Advanced Attacks

The attack continues in CloudShell, because CloudShell simulates an external environment with a different account ID from the one that owns the web app. This makes it perfect for mimicking an attacker using stolen credentials. By configuring CloudShell with the stolen IAM credentials and accessing the S3 bucket, I can trigger suspicious activity alerts in GuardDuty—just like a real breach would in a production AWS setup.


In CloudShell, I used wget to download the credentials.json file from the Juice Shop web app, which simulates how an attacker steals sensitive data after gaining access. Next, I ran a command using cat and jq to display the contents of the file in a readable JSON format. This allowed me to verify the stolen credentials and understand the structure of the data exposed through the vulnerability.

I configured a new AWS CLI profile named stolen to simulate an attacker using stolen credentials. This lets me run AWS commands as if I were the attacker, without using my own permissions. By setting up this profile in CloudShell, I can interact with AWS resources from a different identity, mimicking how a real breach would operate using compromised credentials.

![Image](http://learn.nextwork.org/confident_turquoise_quiet_hyena/uploads/aws-security-guardduty_j9k0l1m2)

---

## GuardDuty's Findings

After performing the attack, GuardDuty reported a finding within 10–15 minutes. Findings are alerts that help identify suspicious activity in my AWS environment. They include details about what happened, such as the use of stolen credentials from another account, who initiated the action, and which resources were involved. This information is useful for detecting and responding to potential security threats quickly.


GuardDuty's finding was called "UnauthorizedAccess\:IAMUser/InstanceCredentialExfiltration" which means stolen EC2 instance credentials were used from a different environment. Anomaly detection was used because GuardDuty noticed credentials tied to an EC2 instance were suddenly used by another account, which is unusual behavior and signals a possible breach or compromise of sensitive data.


 GuardDuty's detailed finding reported that an IAM role was used to access the juice shop’s S3 bucket, and an object was retrieved from it. The action confirmed the use of stolen credentials. The finding also showed the attacker's IP address and the AWS region, which matched the CloudShell environment. This information helps identify the source, method, and potential impact of the unauthorized activity.


![Image](http://learn.nextwork.org/confident_turquoise_quiet_hyena/uploads/aws-security-guardduty_v1w2x3y4)

---

## Extra: Malware Protection

 For my project extension, I enabled Malware Protection in GuardDuty to monitor and scan objects uploaded to my S3 bucket for malicious content. Malware is any software intentionally designed to cause damage to a computer system, such as viruses, ransomware, or spyware. Enabling this feature helps detect such threats early by automatically scanning files and providing detailed findings if anything suspicious is found.

To test Malware Protection, I uploaded an EICAR test file to my S3 bucket. The uploaded file won't actually cause damage because it's a harmless file created by the European Institute for Computer Antivirus Research to simulate malware. It's commonly used to check if antivirus and security systems like GuardDuty can properly detect threats without using real malicious code.


 Once I uploaded the file, GuardDuty instantly triggered a malware detection finding for the EICAR test file. This verified that Malware Protection is actively scanning S3 bucket uploads and can correctly identify potential threats, even when using test files. It confirmed that GuardDuty’s malware detection capability is working as expected, helping to secure the environment against malicious file uploads.


![Image](http://learn.nextwork.org/confident_turquoise_quiet_hyena/uploads/aws-security-guardduty_sm42x3y4)

---
