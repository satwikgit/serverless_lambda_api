# Serverless API Gateway with RDS

## Background info: 

- The aim of the test is to develop a cloudformation template for 
- Maria RDS
- Cognito pool
- Rest api to interact with the Maria RDS permitted to users in the cognito pool only.
- This is achived through Nested CF templates i.e a main template in turn deploys additional templates in the order listed. 
- The advantage being that the stack would only be deployed if all the resources are deployed or it will be rolled back to a 'zero' state. 
- The CF stack has certain parameters that need to be set before it can be deployed.

## SetUp:

There are five yml files and one lambda function zip file needed to deploy the stack 
- core.yml,
- MariaRDS.yml, 
- CognitoUserPool.yml, 
- lambda-swagger.yml,
- swagger.yml.
- energicos-test.zip

the core.yml calls nested yml in the order listed above. swagger.yml is nested in lambda-swagger.yml.

To deploy nested CF templates, the templates need to be uploaded to an s3 bucket first. This is necessary as this is the only way AWS would permit nested stacks to be deployed. 


###1) Create an s3 bucket - {s3-bucket}
  No special permissions needed for this example.
  You can leave the default settings of s3 bucket as is.
  You can create an s3 bucket both from console or from command line as follows
  ```
    aws s3api create-bucket --bucket {s3-bucket} --region us-east-1
  ```
  this will give a s3 location 
  the URL to the s3 bucket would be https://s3.{aws-region}.amazonaws.com/{s3-bucket}/

###2) Change the URLs of nested templates in 
  1) core.yml - locations of nested templates

      ```yaml
        MariaRDS:
          ...
            ...
            TemplateURL: {s3-bucket-url}/MariaRDS.yml
            ...
            ...
        CognitoUserPool:
          ...
            ...
            TemplateURL: {s3-bucket-url}/CognitoUserPool.yml
            ...
            ...
        CrudLambda:
          ...
            ...
            TemplateURL: {s3-bucket-url}/lambda-swagger.yml
            ...
            ...
      ```

  2) lambda-swagger.yml - change the S3 PATH (not the URL) of swagger file  
      ```yaml
        ApiGatewayRestApi:
          Type: 'AWS::ApiGateway::RestApi'
            ...
              ...
                  Location: !Sub "s3://{s3-bucket}/swagger.yml" 
                  ...
                  ...
      ```
###3) upload the following files to s3
  1) core.yml
  2) MariaRDS.yml
  3) CognitoUserPool.yml
  4) lambda-swagger.yml
  5) swagger.yml
  6) energicos-test.zip

  the following command lets you copy all yml files energicos-test.zip in your folder to the s3 bucket
  ```
    aws s3 cp ./ s3://{s3-bucket}/ --recursive --exclude "*" --include "*.yml" --include "energicos-test.zip"
  ```



###4) Deploying the CF template 
  1) From Console: 
    1) click on create stack button to start the process
    2) Specify an Amazon S3 template URL in Choose a template - specify the s3 url for core.yml.
    3) fillout the parameter values 
      1) stack name: any
      2) DBPassword: password to the db to be created
      3) ServerlessDeploymentBucket: {s3-bucket}
    4) give the necessary permissions as needed to create the stacks.
    5) It will take a while for the stack to complete 
    6) Once the stack is of CREATE_COMPLETE status, the Outputs tab will have the 'ServiceEndpoint' to access the service deployed. 
  2) From CommandLine:
    1)
    ```
      aws cloudformation create-stack --stack-name myteststack --template-url {s3-bucket-url}/core.yml --parameters ParameterKey=DBPassword,ParameterValue=SomeGoodPassword ParameterKey=ServerlessDeploymentBucket,ParameterValue={s3-bucket} --capabilities CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND
    ```
    2) It will take a while for the stack to complete .. you can check the status by
    ```
      aws cloudformation describe-stacks --stack-name myteststack
    ```
    3) Once the stack is of CREATE_COMPLETE status, the Outputs tab will have the 'ServiceEndpoint' to access the service deployed. 

###5) Accessing the services
- create user in the newly created cognito pool. 
- login with the username and password to have the auth token
- you can access the service with the auth token in the Authorization token in the header.


