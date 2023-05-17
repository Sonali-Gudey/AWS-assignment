
### Praneeth's approach:

#### Section 1 :- Create S3 bucket from AWS CLI
1. After login into aws from CLI using the AWS configure. For creating a role use create role command giving trust policy in a JSON file.
2. Creating Trust policy for Ec2 instance
3. For giving s3 full access use attach-role-policy command mentioning AmazonS3FullAccess.
4. Create an EC2 instance with the above role
5. To create an s3 bucket use "s3api create-bucket" command.


#### Section 2 :- Put files in S3 bucket from lambda
1. Create custom role for AWS lambda which will have only put object access

2. Add role to generate and access Cloudwatch logs

3. In python script, generate json in given format and save .json file in bucket created

4. Create lambda function using above role

5. Schedule the job to run every minute. Stop execution after 3 runs

6. Create a rule in cloud watch events with cron expression "* * * * ? *" to trigger above created lambda function.

7. Then enable the rule.

8. After 3 executions disable the rule.

9. check the colud watch log

#### Section 3 :- API gateway - Lambda integration
1. Modify lambda function to accept parameters and return file name.
2. Create a POST API from API Gateway, pass parameters as request body to Lambda job. Return the filename and status code as a respon
3. Go to API Gateway create new rest API. Then Create Resource. In the resource create post method. ii. Go to integration request add mapping template "application/json". Put below code there.

```
#set($inputRoot = $input.path('$'))
{
    "body": $input.json('$')
}
```

4. Consume API from the local machine and pass unique data to lambda.
5. Open postman appication and select post method then paste the url with endpoint. ii In the body select raw and json type. Give the body there then hit the api.
6. chech if cloudwatch log are generated


### Ankit's approach:

#### Section 1 :- Create S3 bucket from AWS CLI
1. Initially he generated access key from aws console to access aws from CLI.
2. By using AWS configure command he login into aws and by using create role command role is created giving put object access.
3. Then he added s3 full access policy to it.
4. Instance profile is created using create profile command and added it into role.
5. By using ec2 run instance command she created an instance in the profile just created.
6. By using  s3api create-bucket command s3 bucket is created.
#### Section 2 :- Put files in S3 bucket from lambda
1. Initially new custom policy is created from command line.
2. By using create role command role is created with put object access.
3. Then cloud watch logs access is attached to it.
4. Lambda function is created from GUI with required parameters.
5. Then event rule is created form aws console.
6. Lambda executes for 3 time then it stops.
7. cloud watch logs are generated.

#### Section 3 :- API gateway - Lambda integration
1. Lambda function is updated that accepts request from an API and put object in s3 bucket.
2. From API gateway new api is created with post request method.
3. Then he added lambda function to an API.
4. Then he tested the API and deployed it.
5. Then he hit the API from local using curl command


