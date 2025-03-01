# EC2 Instance Scheduler

## Overview
This repository contains an AWS Lambda function designed to automate the start and stop of EC2 instances based on a custom tag (`KeepOn`). The function evaluates the schedule defined in the `KeepOn` tag and ensures instances are running or stopped according to the specified time frame.

## Features
- **Automated Instance Management**: Starts and stops EC2 instances based on a predefined schedule.
- **Custom Scheduling**: Uses a tag format (`Mo+Tu+We+Th+Fr:08-19`) to define active hours.
- **Timezone Aware**: Uses Eastern Standard Time (EST) for scheduling.
- **Logging**: Provides detailed logs for monitoring and debugging.

## Tag Format
The `KeepOn` tag should be formatted as follows:

```
Mo+Tu+We+Th+Fr:08-19
```

- Days: `Mo`, `Tu`, `We`, `Th`, `Fr`, `Sa`, `Su`
- Hours: `start_hour-end_hour` (24-hour format)

Example:
- `Mo+We+Fr:09-17` → The instance runs on Monday, Wednesday, and Friday from 9 AM to 5 PM EST.

## How It Works
1. The Lambda function retrieves all EC2 instances with the `KeepOn` tag.
2. It extracts and parses the schedule from the tag.
3. It checks the current time in **Eastern Standard Time (EST)**.
4. If the current time matches the schedule:
   - It **starts** stopped instances at the scheduled start hour.
   - It **stops** running instances at the scheduled stop hour.

## Deployment
### Prerequisites
- AWS Lambda execution role with the following permissions:
  - `ec2:DescribeInstances`
  - `ec2:StartInstances`
  - `ec2:StopInstances`
- Python runtime (e.g., Python 3.8+)
- Boto3 library installed
- pytz library installed or layer added to the lambda

### Steps
1. Upload the script to an AWS Lambda function.
2. Set the execution role with required permissions.
3. Configure the function to trigger based on a CloudWatch event (e.g., every hour).
4. Deploy and monitor logs via AWS CloudWatch.

## Logging
The function logs important details such as:
- Current day and hour
- Instances being processed
- Scheduling decisions
- Errors encountered

Logs can be viewed in **AWS CloudWatch**.

## Error Handling
- If a malformed `KeepOn` tag is encountered, an error is logged, and the function continues processing other instances.
- If AWS API calls fail, the error is logged for debugging.

## Example CloudWatch Rule
To run the function hourly, create a CloudWatch rule with the following cron expression:
```
cron(0 * * * ? *)
```
This ensures the function runs at the beginning of every hour.

## Sample Template for CloudFormation/AWS Sam

```
AWSTemplateFormatVersion: '2010-09-09'
Transform: 'AWS::Serverless-2016-10-31'
Resources:
  EC2SchedulerFunction:
    Type: 'AWS::Serverless::Function'
    Properties:
      Handler: lambda_function.lambda_handler
      Runtime: python3.8
      CodeUri: ./
      Description: 'EC2 Instance Scheduler based on KeepOn tags'
      MemorySize: 128
      Timeout: 30
      Policies:
        - Version: '2012-10-17'
          Statement:
            - Effect: Allow
              Action:
                - 'ec2:DescribeInstances'
                - 'ec2:StartInstances'
                - 'ec2:StopInstances'
              Resource: '*'
      Events:
        ScheduleEvent:
          Type: Schedule
          Properties:
            Schedule: 'cron(0 * * * ? *)'
            Description: 'Run every hour'
```

## Sample main.tf Template for deployment with Terraform

```
resource "aws_lambda_function" "ec2_scheduler" {
  function_name    = "ec2-instance-scheduler"
  handler          = "lambda_function.lambda_handler"
  runtime          = "python3.8"
  filename         = "lambda_function.zip"  # Create this ZIP with the Python code
  source_code_hash = filebase64sha256("lambda_function.zip")
  role             = aws_iam_role.lambda_role.arn
  timeout          = 30
  memory_size      = 128
}

resource "aws_cloudwatch_event_rule" "hourly" {
  name                = "hourly-ec2-scheduler-trigger"
  description         = "Trigger EC2 scheduler every hour"
  schedule_expression = "cron(0 * * * ? *)"
}

resource "aws_cloudwatch_event_target" "run_ec2_scheduler" {
  rule      = aws_cloudwatch_event_rule.hourly.name
  target_id = "ec2_scheduler"
  arn       = aws_lambda_function.ec2_scheduler.arn
}
```


## Contributing
Feel free to submit issues or pull requests to improve functionality.


