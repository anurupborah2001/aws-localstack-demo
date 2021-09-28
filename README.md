AWS Localstack Demo Implementation
-----------

1. aws configure

    >aws_access_key_id :  temp<br/>
   aws_secret_access_key :  temp<br/>
    region : ap-southeast-1<br/>
    output: json

## S3
1. Create S3 bucket
	aws s3 mb --endpoint-url=http://localhost:4566 --region ap-southeast-1 s3://static-s3-bucket

2. List S3 Bucket
	aws s3 ls --endpoint-url=http://localhost:4566 --region  ap-southeast-1

3. Copy object to AWS S3 bucket
	aws s3 cp demo.jpg ss3://static-s3-bucket/demo.jpg --endpoint-url=http://localhost:4566 --region  ap-southeast-1

4. provide Bucket permissions
	aws s3api --endpoint-url=http://localhost:4566 put-public-access-block --bucket static-s3-bucket --public-access-block-configuration "BlockPublicAcls=false,IgnorePublicAcls=false,BlockPublicPolicy=false,RestrictPublicBuckets=false"

5. Provide public access read of the url
   aws s3api --endpoint-url=http://localhost:4566 put-bucket-policy --bucket static-s3-bucket --policy "{"Version":"2012-10-17","Statement":[{"Sid":"PublicReadGetObject","Effect":"Allow","Principal":"*","Action":"s3:GetObject","Resource":"arn:aws:s3:::static-s3-bucket"}]}"

6. Create a static website in S3
    aws s3 --endpoint-url=http://localhost:4566 website "s3://static-s3-bucket" --index-document index.html --error-document index.html

7. Sync the S3 bucket with the website folder
    aws s3 --endpoint-url=http://localhost:4566 sync website "s3://static-s3-bucket"

8. Check the webiste using curl
    curl http://localhost:4566/static-s3-bucket/index.html

## Dynamodb

1. Create table in DynamoDB
   aws --endpoint-url=http://localhost:4566 dynamodb create-table --cli-input-json file://dynamodb/table-definition.json

2. List tables
    aws --endpoint-url=http://localhost:4566 dynamodb list-tables

3. Scan a table
    aws --endpoint-url=http://localhost:4566 dynamodb scan --table-name "Music"

4. Insert item to Dynamodb table
    aws --endpoint-url=http://localhost:4566 dynamodb batch-write-item --request-items file://dynamodb/insert-items.json

## IAM Role

1. Create role
    aws --endpoint-url=http://localhost:4566 iam create-role --role-name "aws-localstack-demo" --assume-role-policy-document file://dynamodb/execution-role.json

2. Create  trust Policy
    aws --endpoint-url=http://localhost:4566 iam create-role --role-name "aws-localstack-trust-demo" --assume-role-policy-document file://dynamodb/trust-policy.json

3. List Roles
    aws --endpoint-url=http://localhost:4566 iam list-roles

## Lambda
   
1. Zip the Lambda function
    zip function.zip index.js 

2. Create Lambda Function
    aws --endpoint-url=http://localhost:4566 lambda create-function --function-name "localstack-lambda" --zip-file fileb://lambda/function.zip --handler 'index.handler' --runtime nodejs14.x --role aws-localstack-demo 

3. Invoke Lambda Function and Output
    aws --endpoint-url=http://localhost:4566 lambda invoke --function-name "localstack-lambda" output.json --log-type Tail --query 'LogResult' --output text | base64 -d

4. List functions
    aws --endpoint-url=http://localhost:4566 lambda list-functions

## CDK 
1. Install
    npm i -g aws-cdk-local aws-cdk

2. Create CDK Skeleton
    cdklocal init sample-app --language typescript

3. Create Cloudformation Template
    cdklocal synth > cf-out.yml

4. CDk Bootstrap 
    cdklocal bootstrap

    CDk bootstrap with different region
    cdklocal bootstrap aws://000000000000/ap-southeast-1

5. CDK Deploy
   cdklocal deploy

6. List Cloudformation Stacks
    aws --endpoint-url=http://localhost:4566 cloudformation list-stacks

7. List the resources created with Cloudformation Stacks
   aws --endpoint-url=http://localhost:4566 cloudformation list-stack-resources --stack-name "CdkStack"

8. Check CDK difference
   cdklocal diff

9. Involde Lambda crated via CDK
    aws lambda invoke --cli-binary-format raw-in-base64-out --function-name CdkStack-HelloFunctionD909AE8C-b679adb4 --invocation-type RequestResponse --no-sign-request --endpoint-url=http://localhost:4566  output.json

10. Invoke lambda using Payload
    aws --endpoint-url=http://localhost:4566 lambda invoke --function-name "CdkStack-HelloFunctionD909AE8C-b679adb4" --no-sign-request  output.json --payload file://awsapigateway-aws-proxy.json --invocation-type RequestResponse


## APIGateway
1. Install
    npm i @aws-cdk/aws-apigateway  

2. Once CDK deploy witll give thr APi Gatewy URL : 
    Example : Outputs:
            CdkStack.MYAPIEndpoint1087FBBC = https://qx3khpbhdy.execute-api.ap-southeast-1.localhost/prod/

    We can check locally using the below url : 
      http://localhost:4566/restapis/qx3khpbhdy/prod/_user_request_/

### Reference
https://www.youtube.com/watch?v=3_sqr0G9zb0

