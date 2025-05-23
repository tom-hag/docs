AWS SSM (Systems Manager) https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent.html

>AWS Systems Manager Agent (SSM Agent) is Amazon software that runs on Amazon Elastic Compute Cloud (Amazon EC2) instances, edge devices, on-premises servers, and virtual machines (VMs). SSM Agent makes it possible for Systems Manager to update, manage, and configure these resources. The agent processes requests from the Systems Manager service in the AWS Cloud, and then runs them as specified in the request.



# The problem

After launching an ec2 (running Ubuntu) instance to use as a bastion host to access an rds postgres database, attempts to use
```zsh
aws ssm start-session --profile <profile-name> \
  --target <instance-id> \
  --document-name AWS-StartPortForwardingSession \      
  --parameters '{"portNumber":["5432"],"localPortNumber":["5432"]}'
```
were failing as it said the ec2 instance was not connected. The security groups, VPCs and networking seemed correct to start a session using `ssm`.

`ssh`'ing directly into the instance and running:
`sudo systemctl status snap.amazon-ssm-agent.amazon-ssm-agent.service`
yielded a seeming successful status.
```zsh
● snap.amazon-ssm-agent.amazon-ssm-agent.service - Service for snap application amazon-ssm-agent.amazon-ssm-agent
     Loaded: loaded (/etc/systemd/system/snap.amazon-ssm-agent.amazon-ssm-agent.service; enabled; preset: enabled)
     Active: active (running) since Thu 2025-05-22 03:58:46 UTC; 13min ago
```

inspecting the logs:
`sudo cat /var/log/amazon/ssm/amazon-ssm-agent.log`
There was an error that showed a Role issue:

```zsh
2025-05-22 03:17:15.4429 ERROR EC2RoleProvider Failed to connect to Systems Manager with SSM role credentials. error calling RequestManagedInstanceRoleToken: AccessDeniedException: Systems Manager's instance management role is not configured for account: <...>
        status code: 400, request id: e29b9640-97af-4a1b-bbde-0168cabe7994
```


# Resolution 

Adding an IAM role with the permission `AmazonSSMManagedInstanceCore`
and restarting the SSM agent with `sudo systemctl restart snap.amazon-ssm-agent.amazon-ssm-agent` service fixed the issue. And this can be confirmed with
```zsh
aws ssm describe-instance-information
```

which should now show the ec2 instance in question.

See also https://aws.amazon.com/blogs/database/securely-connect-to-an-amazon-rds-or-amazon-ec2-database-instance-remotely-with-your-preferred-gui/ for more detail on the architecture