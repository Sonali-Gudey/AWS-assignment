# AWS-assignment


### Question-1: Create S3 bucket from AWS CLI
#### a. Create an IAM role with S3 full access
i. After login into aws from CLI using the AWS configure. Use the create role command with a JSON file containing the trust policy to create a role.
```
aws iam create-role --role-name sonali-q1-s3access  --assume-role-policy-document file://ec2-trust-policy.json
```

ii. The trust policy, which specifies that EC2 is the use case, is shown below.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "ec2.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

iii. Use the attach-role-policy command, referencing AmazonS3FullAccess, to grant S3 full access. This command allows full access to all S3 resources in the account by attaching the AmazonS3FullAccess policy to the role.

```
aws iam attach-role-policy --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess --role-name sonali-q1-s3access
```

<img width="1111" alt="Screenshot 2023-05-12 at 12 00 43 PM" src="https://github.com/Sonali-Gudey/AWS-assignment/assets/123619701/d12fc8c5-5634-4316-b939-5346b80efcf6">



#### b. Create an EC2 instance with the above role.

i. Inorder to create an EC2 instance from the command line we need to create an instance profile for the role just created.

```
aws iam create-instance-profile --instance-profile-name sonali_instance_profile

```


<img width="1055" alt="Screenshot 2023-05-12 at 11 43 58 AM" src="https://github.com/Sonali-Gudey/AWS-assignment/assets/123619701/d313905c-d52e-43ab-a3d6-18d67305ae4e">


```
aws iam add-role-to-instance-profile --instance-profile-name sonali_instance_profile --role-name sonali-q1-s3access
```

```
aws ec2 run-instances \
    --image-id ami-0889a44b331db0194 \
    --count 1 \
    --instance-type t2.micro \
    --key-name sonali-ec2-key \
    --iam-instance-profile Name=sonali_instance_profile \
    --region us-east-1

```

<img width="1195" alt="Screenshot 2023-05-12 at 12 22 27 PM" src="https://github.com/Sonali-Gudey/AWS-assignment/assets/123619701/93f09582-4dbe-41f3-8d3b-3d55e7afa092">


#### c. Create a S3 bucket from AWS CLI.

```
aws s3api create-bucket --bucket sonali-s3-q1  --region us-east-1
```

<img width="1092" alt="Screenshot 2023-05-12 at 12 30 05 PM" src="https://github.com/Sonali-Gudey/AWS-assignment/assets/123619701/da1c62db-0ed2-42ce-9977-f85de1de460b">





Output:



<img width="735" alt="Screenshot 2023-05-12 at 12 28 37 PM" src="https://github.com/Sonali-Gudey/AWS-assignment/assets/123619701/d2b5fcc5-8f68-485e-a071-15004db13636">










### Question-2: Put files in S3 bucket from lambda

#### a.1 Create a new IAM policy that allows the Lambda function to write objects to an S3 bucket:
```
aws iam create-policy --policy-name lambda-s3-put-policy --policy-document file://lambda-s3-put-policy.json

```

```
Contents of lambda-s3-put-policy.json file:
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "S3PutObject",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::your-bucket-name/*"
        }
    ]
}
```


#### a.2. Create a new IAM role for the Lambda function and attach the policy created in step 1:

```
aws iam create-role --role-name lambda-s3-put-role --assume-role-policy-document file://lambda-trust-policy.json
```

```
aws iam attach-role-policy --policy-arn arn:aws:iam::574344495913:policy/lambda-s3-put-policy --role-name lambda-s3-put-role
```

```
Contents of lambda-trust-policy.json:

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "lambda.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}

```

#### b.1 Create a new IAM policy that allows the Lambda function to write logs to CloudWatch:

```
aws iam create-policy --policy-name lambda-cloudwatch-policy --policy-document file://lambda-cloudwatch-policy.json
```

```
Contents of json:

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "CloudWatchLogs",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:*:*:*"
        }
    ]
}
```

#### b.2 Attach the CloudWatch logs policy to the Lambda function role:

```
aws iam attach-role-policy --policy-arn arn:aws:iam::574344495913:policy/lambda-cloudwatch-policy --role-name lambda-s3-put-role
```

<img width="1092" alt="Screenshot 2023-05-12 at 1 09 09 PM" src="https://github.com/Sonali-Gudey/AWS-assignment/assets/123619701/6fe58f57-9dac-4fd0-9292-63648497ab77">



#### c. Create a new Lambda function using the above role

```
 aws lambda create-function \
--function-name sonali-lambda-function \
--runtime python3.8 \
--handler sonali-handler-function \
--role arn:aws:iam::574344495913:role/lambda-s3-put-role \
--timeout 10 \
--zip-file fileb://~/Documents/lambda-function-code.zip
```

#### d.1 Schedule the job to run every minute. Stop execution after 3 runs
i. Create a rule in cloud watch events with cron expression "* * * * ? *" to trigger above created lambda function. Then enable the rule.

ii. After 3 executions disable the rule.

```
import boto3
import datetime, time
import json

s3 = boto3.resource('s3')


bucket_name = 'sonali-s3-q1'
key_name = 'transaction{}.json'

logs_client = boto3.client('logs')


def lambda_handler(event, context):
    try:
        # Generate JSON in the given format
        transaction_id = 12345
        payment_mode = "card/netbanking/upi"
        Amount = 200.0
        customer_id = 101
        Timestamp = str(datetime.datetime.now())

        transaction_data = {
            "transaction_id": transaction_id,
            "payment_mode": payment_mode,
            "Amount": Amount,
            "customer_id": customer_id,
            "Timestamp": Timestamp
        }
        
        # Save JSON file in S3 bucket
        json_data = json.dumps(transaction_data)
        file_name = key_name.format(Timestamp.replace(" ", "_"))
        s3.Bucket(bucket_name).Object(file_name).put(Body=json_data)
        
        # Log the S3 object creation event
        log_group = 'sonali_logs'
        log_stream = 'sonali_stream_data'
        log_message = f"Object created in S3 bucket {bucket_name}"
        logs_client.create_log_group(logGroupName=group_name)
        logs_client.create_log_stream(logGroupName=group_name, logStreamName=stream_name)
        logs_client.put_log_events(
            logGroupName=log_group,
            logStreamName=log_stream,
            logEvents=[{
                'timestamp': int(round(time.time() * 1000)),
                'message': log_message
            }]
        )
        
        # Stop execution after 3 runs
        if context.invoked_function_arn.endswith(':1'):
            print('First execution')
        elif context.invoked_function_arn.endswith(':2'):
            print('Second execution')
        elif context.invoked_function_arn.endswith(':3'):
            print('Third execution')
        else:
            print('Stopping execution')
            return
        
    except Exception as e:
        print(e)

```

<img width="1189" alt="Screenshot 2023-05-14 at 8 10 03 PM" src="https://github.com/Sonali-Gudey/AWS-assignment/assets/123619701/fb4cf169-9e7a-494c-af9a-346cb751f97b">

#### d.2 Check if cloud watch logs are generated.

<img width="1189" alt="Screenshot 2023-05-14 at 8 10 52 PM" src="https://github.com/Sonali-Gudey/AWS-assignment/assets/123619701/8c6a8719-ae4c-4a96-a280-4f5d391ddb5d">

### Question-3: API gateway - Lambda integration

#### a. Modify lambda function to accept parameters

```
import boto3
import datetime
import json

s3 = boto3.resource('s3')
bucket_name = 'sonali-s3-q1'
key_name = 'rj{}.json'

def lambda_handler(event, context):
    try:
        # Parse input data
        body = event['body']
        timestamp = str(datetime.datetime.now())
        body["timestamp"] = timestamp
        

        # Save JSON file in S3 bucket
        json_data = json.dumps(body)
        file_name = key_name.format(timestamp.replace(" ", "_"))
        s3.Object(bucket_name, file_name).put(Body=json_data)

        # Log the S3 object creation event
        print(f"Object created in S3 bucket {bucket_name}: {file_name}")

        return {
            "file_name": file_name,
            "status": "success"
        }

    except Exception as e:
        print(e)
        return {
            "status": "error"
        }

```

#### b. Create a POST API from API Gateway, pass parameters as request body to Lambda job. Return the filename and status code as a response.
i. Go to API Gateway create new rest API. Then Create Resource. In the resource create post method. 

ii. Go to integration request add mapping template "application/json". Put below code there.

```
#set($inputRoot = $input.path('$'))
{
    "body": $input.json('$')
}
```

iii. Test the API post request method. iV. Deploy the api and copy url.

<img width="1427" alt="Screenshot 2023-05-14 at 8 22 08 PM" src="https://github.com/Sonali-Gudey/AWS-assignment/assets/123619701/8e122e32-bb0c-42a5-858e-da829a5c40de">

#### c. Consume API from the local machine and pass unique data to lambda.
i. Open postman appication and select post method then paste the url with endpoint. 

ii In the body select raw and json type. Give the body there then hit the api.







