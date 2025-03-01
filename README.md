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

## Contributing
Feel free to submit issues or pull requests to improve functionality.


