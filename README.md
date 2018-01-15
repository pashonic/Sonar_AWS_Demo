# Solution Highlights

* Immutable environment AMI based deployments using Autoscaling groups. 
* AMI baking is done via Ansible.
* Infrastructure creation via Cloud Formation executed via Ansible.
* Anisble scirpts can be ran against local workstation VM.

# Non-System Requirements For Running

* Permission to access this GIT repo (which you have if you see this).
* Valid AWS Access Key pair.

# System Requirements For Running

* SHH public key located at following system location: ~/.ssh/id_rsa.pub. See aws_create_ami.yml if you need to change location.
* GIT Installed. 
* This GIT repo cloned to system.
* Tools Installed (via pip:
  * Ansible.
  * Boto python package.
  * Boto3 python package.
  * AWS CLI.
  
Optional:
[Disable SSH Host Checking](http://bencane.com/2013/07/22/ssh-disable-host-checking-for-scripts-automation/)

Note: Fresh ubuntu 16.04 environment was used to testing solution; Run the setup_environment_debian script in sudo mode:
```
$sudo bash setup_environment_debian
```

# How To Run

## Preparing System Environment.
1. Review "System Requirements For Running" section of this document.
2. Inject aws access keys:
```
$aws configure
```

## Baking AMI.

1. Execute AMI baking command:

```
$ansible-playbook aws_create_ami.yml --extra-vars "targetenv=aws sshconfiguser=centos"
```
Note: You may get SSH host checking prompt.
2. Wait awhile (5-6 minutes)
3. Note down the AMI ID from Anisble output:
```
TASK [debug] *******************************************************************
ok: [localhost] => {
    "msg": "AMI ID ami-17ed5d6f"
}
```

## Creating Infrastructure.

1. Execute AMI baking command:
```
ansible-playbook aws_create_update_stack.yml --extra-vars "ami_id=[AMI ID] stack_name=[Stack Name]"
```
2. Wait awhile (9-10 minutes)
3. Get the LB "DNS name" URL from Cloud Formation output. (AWS Console)
4. Attempt to access DNS name from browser (Http/Port 80).

Note: Wait 1-2 minutes for URL to work.

## Configuring local VM (no AWS).

# Assignment Requirements I didn't Fulfill Due to Time.

## I didn't encrypt database, big missing security feature.
I would have used Clusters Aurora to allow for encryption.

## I didn't do the Telemetry portion. I would have done the following if I had time:
* Implemented Cloud Watch Alarms against instances for CPU and memory. 
* Implemented Cloud Watch Alarms LB.
* Setup RDS alerts for database.
* Maybe Alarms for the scaling group, Alarm if instance cycles.
* Implement Route 53 health check alerting against endpoint URL. 
* All the alerts would email pager duty (Severe) or Team email list (Warnings).
* Use tools such as New Relic or Appdynamics.
* EC2 Detailed monitoring is enabled.

# Things I Would Done if I Had More Time.

* Implement HTTPS and get proper whole domain cert. (Security)
* Encrypt the database, using clustred aurora. (Security)
* Encrypt database password via KMS (Security) (Decrypt in userdata)
* Remove public key from AMI. (Security)
* Put library dependencies in S3 or owned repo. (Security)
* Use office approved hardened Linux image. (Security)
* Imeplment DNS failover Route Policy using healthchecks. This could also greatly improve downtime between updates (overkill Availability) 
* Improve parameterization and remove hard-coded values from templates/scripts (Readability)
* Delete or tag Ansible generated AWS temp objects for baking AMI.
* Performance tune (Figure out correct EC2 and RDS Database type)
* Evaluate costs of solutions.

# Things I Would Have Done if I Unlimited tTime.
* Create docker image just for running scripts.
* Create one script that does entire process via one command.

# How use case scenarios are handled.

### Updating Sonarqube.

Implmenting and testing changes can be done both locally and in AWS.

Engineers would duplicate the production database and test it against the new AMI.

Updating would simply involved updating the existing CF instance with the new AMI.

There will be downtime during the update process (1-2 minutes)

### Sonar crashes.

If sonar or the server crashes the ELB will mark instance and unhealthy and Autoscaling group will replace instance.

### Developer wants to run local version of Sonar.

Developer can use the above "Configuring local VM" steps.

# 3rd Party Tools/Libraries



