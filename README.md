# Logstash-AWS-WAF
AWS WAF integration with Microsoft Sentinel via S3.

# Logstash EC2 Setup

1. Create an EC2 instance -
Create an EC2 instance with Ubuntu Server (20.04+). Ubuntu Server 18.04 LTS is also fine but try to stick with the latest version at the moment. Make sure the 64-bit (x86) is selected.

![image](https://github.com/TheNadav/Logstash-AWS-WAF/assets/105583152/d8d641cf-16b3-4abd-942e-73661e2b37e9)

2. Configure Instance Details and move forward to the Add Storage step. Make the size of the disk space to a minimum 30GB gp3 with the default IOPS (3000)

![image](https://github.com/TheNadav/Logstash-AWS-WAF/assets/105583152/cc117970-dd55-46de-b1a3-b5285c2fdb9f)

3. Update the EC2 - `sudo apt update && upgrade`

4. Install Java - `sudo apt-get install default-jre`

5. Create IAM role for the EC2 with the following settings 
-Trusted entity type - AWS service
-Use case > Service or use case > EC2

Assigne AmazonS3ReadOnlyAccess policy and set the logs bucket ARN at the `"Resource":` parameter.

6. Set the bucket policy-
```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "[EC2-Role-ARN]"
      },
      "Action": "s3:GetObject", "s3:ListBucket", "s3:DescribeBucket",
      "Resource": "arn:aws:s3:::[my bucket name]",
      "Condition": {
        "ArnEquals": {
          "aws:SourceArn": "[ec2_ARN]"
        }
      }
    }
  ]
}
```


# Installing Logstash APT

1. Download and install the Public Signing Key:
`wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic-keyring.gpg`

2. Save the repository definition to /etc/apt/sources.list.d/elastic-8.x.list:
`echo "deb [signed-by=/usr/share/keyrings/elastic-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-8.x.list`

3. Run `sudo apt-get update` and the repository is ready for use. You can install it with:
`sudo apt-get update && sudo apt-get install logstash`
Logstash binary will be located at `/usr/share/logstash/`
Logstash configurations and settings will be located at `/etc/logstash/`

4. Install the Microsoft Log Analytics logstash output plugin
`bin/logstash-plugin install microsoft-logstash-output-azure-loganalytics`

5. Disable `pipeline.ecs_compatibility` at `/etc/logstash/logstash.yml`

![image](https://github.com/TheNadav/Logstash-AWS-WAF/assets/105583152/b4cbdcee-7419-4de9-9114-791c88cae236)


# Config File 

1. Create a new config file `sudo vim /etc/logstash/conf.d/logstash.conf` 

2. Use the <file> and edit the following parameters -
* `bucket => "aws-waf-logs-{bucketname}"`
* `workspace_id => "{WorkspaceID}"`
* `workspace_key => "{WorkspaceKey}"`
* `custom_log_table_name => "{TableName}"`

3. Start ship logs -
`sudo nohup bin/logstash --path.settings /etc/logstash >/dev/null 2>&1 &`

