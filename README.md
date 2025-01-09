# NBA Games Notification
## Table of Contents
1. [Overview]()
2. [Features]()
3. [Architecture]()
4. [Setup and Configuration]()
5. [Environment Variables]()
6. [How it Works]()
7. [Detailed Workflow]()
8. [Error Handling]()
9. [Dependencies]()
10. [Execution and Deployment]()
11. [Future Enhancements]()

## Overview
This project automates the extraction and notification of NBA game data from the [SportsDataIO API](), which includes details about scheduled, in-progress, and final games. The extracted game data is processed via an AWS Lambda function, formatted, and then sent using Amazon SNS (Simple Notification Service) to notify subscribers via email. The scheduling is managed by AWS EventBridge rules to trigger the Lambda function periodically.
## Features
- Fetches NBA game details from the SportsDataIO API for the current day (Central Time zone).
- Supports formatting of game data, including:
    - Scheduled, in-progress, and final games.
    - Game status, teams, scores, quarters, start time, and the broadcast channel.

- Automatically sends game updates via an SNS Topic.
- Triggered periodically using AWS EventBridge rules.
- Comprehensive error handling for API requests and SNS publishing.

## Architecture
This project is designed around AWS services:
- **AWS Lambda**: Extracts, formats, and sends game data.
- **AWS SNS**: Distributes game updates to the subscribers (e.g., emails).
- **AWS EventBridge**: Schedules the Lambda function to run based on the configured rule frequency (e.g., daily or hourly).
- **SportsDataIO API**: Serves as the source of NBA game data.

## Setup and Configuration
### Prerequisites
Before deploying or running the project, you need:
1. An active AWS account with permissions for:
    - Lambda
    - SNS
    - EventBridge

2. A valid API key from [SportsDataIO]().
3. A Python runtime environment.

## Environment Variables
The Lambda function requires specific environment variables to be set:

| Variable Name | Description |
| --- | --- |
| `NBA_API_KEY` | SportsDataIO API key to access the game data. |
| `SNS_TOPIC_ARN` | ARN of the SNS Topic to publish NBA game updates. |
Make sure to add these variables in the Lambda function's environment settings before deployment.
## How it Works
1. **Trigger Schedule**: AWS EventBridge triggers the Lambda function based on the predefined rule frequency (in this scenario; every 2 hrs between 9am - 11pm and 12am - 2am).
2. **API Request**: The function fetches NBA game data from the SportsDataIO API for the current day.
3. **Data Formatting**: Game data is processed and formatted based on its status:
    - Scheduled
    - In Progress
    - Final

4. **Notification**: Formatted game details are sent to an SNS Topic, triggering email alerts to subscribers.
5. **Error Handling**: Safeguards are in place for API failures or SNS publishing issues to ensure robustness.

## Detailed Workflow
1. **Adjust for GreenWhich Meridian Time (GMT)**:
    - The Lambda function calculates the current date in the GMT Time Zone (CST: UTC+00).
    - Games for the determined Central Time date are fetched.

2. **Fetch NBA Games**:
    - A GET request is sent to the SportsDataIO API endpoint: 
>https://api.sportsdata.io/v3/nba/scores/json/GamesByDate/{date}?key={NBA_API_KEY}

- `NBA_API_KEY` is passed as a query parameter(from the environment variables).

1. **Process Game Data**:
    - The fetched JSON data about games is parsed.
    - Each game is formatted into readable strings depending on its `Status`:
        - **Final**: Includes final scores, quarter-by-quarter breakdown.
        - **In Progress**: Includes the current score and last play details.
        - **Scheduled**: Includes the start time and broadcast channel.
        - **Unknown**: Displays minimal details if data is incomplete.

2. **Send Notification via SNS**:
    - The compiled message containing all game data is published to the configured SNS Topic.
    - Subscribers receive the game updates through email or other configured notification channels.

3. **Error Handling**:
    - API errors (e.g., invalid responses, connectivity issues) are caught and logged.
    - SNS publishing errors are handled gracefully with error messages returned.

## Error Handling
1. **API Fetching**:
    - Catches exceptions during the data retrieval stage.
    - Logs the error with specific details to help debug.
    - Returns a 500 HTTP response in case of failures.

2. **SNS Publishing**:
    - Accounts for publishing failures (e.g., invalid topic ARN, permission issues).
    - Ensures error details are logged and returned properly.

## Dependencies
This project utilizes the following libraries:
- **`boto3`**: AWS SDK for Python, used to interact with AWS SNS.
- **`urllib.request`**: To make API requests to the SportsDataIO API.
- **`os`**: For accessing environment variables.
- **`json`**: For handling JSON data.
- **`datetime`**: For date and timezone management.

Ensure these are installed or preconfigured in the Lambda runtime environment.
## Execution and Deployment

### Create an SNS Topic
- Open the AWS Management Console.
- Navigate to the SNS service.
- Click Create Topic and select Standard as the topic type.
- Name the topic (e.g., nba_notifications) and note the ARN.
- Click Create Topic.
- Add Subscriptions to the SNS Topic
-After creating the topic, click on the topic name from the list.
- Navigate to the Subscriptions tab and click Create subscription.
- Select a Protocol:
- For Email:
Choose Email.
Enter a valid email address.
- Click Create Subscription.
If you added an Email subscription:
- Check the inbox of the provided email address.
- Confirm the subscription by clicking the confirmation link in the email.

### Create the SNS Publish Policy
- Open the IAM service in the AWS Management Console.
- Navigate to Policies → Create Policy.
- Click JSON and paste the JSON policy from nba_notification_sns_policy.json file
- Replace REGION and ACCOUNT_ID with your AWS region and account ID.
- Click Next: Tags (you can skip adding tags).
- Click Next: Review.
- Enter a name for the policy (e.g., nba_notification_sns_policy).
- Review and click Create Policy.
### Create an IAM Role for Lambda
- Open the IAM service in the AWS Management Console.
- Click Roles → Create Role.
- Select AWS Service and choose Lambda.
- Attach the following policies:
- SNS Publish Policy (nba_notification_sns_policy.json) (created in the previous step).
- Lambda Basic Execution Role (AWSLambdaBasicExecutionRole) (an AWS managed policy). 
- Click Next: Tags (you can skip adding tags).
- Click Next: Review.
- Enter a name for the role (e.g., nba_notification_role).
- Review and click Create Role.
- Copy and save the ARN of the role for use in the Lambda function.
### Deploy the Lambda Function
- Open the AWS Management Console and navigate to the Lambda service.
- Click Create Function.
- Select Author from Scratch.
- Enter a function name (e.g., notifications).
- Choose Python 3.x as the runtime.
- Assign the IAM role created earlier (nba_notification_role) to the function.
- Under the Function Code section:
- Copy the content of the src/nba_lambda.py.py file from the repository.
- Paste it into the inline code editor.
- Under the Environment Variables section, add the following:
> NBA_API_KEY: your NBA API key. 

>SNS_TOPIC_ARN: the ARN of the SNS topic created earlier.

- Click Create Function.
### Set Up Automation with Eventbridge
- Navigate to the Eventbridge service in the AWS Management Console.
- Go to Rules → Create Rule.
- Select Event Source: Schedule.
- Set the cron schedule for when you want updates (e.g., hourly).
- Under Targets, select the Lambda function (gd_notifications) and save the rule.
### Test the System
- Open the Lambda function in the AWS Management Console.
- Create a test event to simulate execution.
- Run the function and check CloudWatch Logs for errors.
- Verify that SNS notifications are sent to the subscribed users.









