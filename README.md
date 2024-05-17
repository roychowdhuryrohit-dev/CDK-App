# AWS CDK Fullstack
This project is about getting hands-on experience with deploying a Full Stack web app based on **React(w/ TailwindCSS)** and deployed using **AWS CDK** which is a infrastructure-as-a-code solution by AWS.

It was hosted using **Amazon Amplify**.

## Usage
Following steps should be run to deploy the web app:

1. `aws`, `aws-cdk`, `npm` and `node` CLI tools should be installed and set in the path.
   
2. Make sure to provide necessary IAM access for `aws` by running:
```bash
$ aws configure
```

3. Now we have to set the following enviroment variables:
```bash
$ export S3_BUCKET_NAME="[AVAILABLE_BUCKET_NAME]"
$ export INPUT_TABLE_NAME="[INPUT_RECORDS_TABLE]"
$ export OUTPUT_TABLE_NAME="[OUTPUT_RECORDS_TABLE]"
$ export HOST_URL="http://localhost:3000"
```

4. If everything is set, we are ready to deploy our AWS infrastructure. We need the AWS Account Number and Region for running the following:
```bash
$ cd infra
$ cdk bootstrap aws://[ACCOUNT-NUMBER]/[REGION]
$ npm run build
$ cdk synth
$ cdk deploy
```

5. Our S3 bucket, Lambda functions, API gateway, DynamoDb tables and IAM role policy have been created and configured. Now we need the API endpoints of the two lambda functions ("_gen-presigned-url-s3_" & "_save-form-dynamo-db_"). These can be found by going to Lambda>Functions>[FUNCTION_NAME]>Configuration>Triggers. Copy the second API endpoint which is for production.

6. Once we have noted the two lambda endpoints, we have to set them as environment variables to build our React app.
```bash
$ export REACT_APP_GEN_S3_URL_API_ENDPOINT="[GEN_S3_URL_API_ENDPOINT]"
$ export REACT_APP_SAVE_FORM_DYNAMO_DB_API_ENDPOINT="[SAVE_FORM_DYNAMO_DB_API_ENDPOINT]"
```

7. Finally we can start our web app by running:
```bash
$ cd ../app
$ npm install
$ npm start
```

### Control Flow

1. After the submit button is clicked, a _PRE-SIGNED URL_ is generated by our _gen-presigned-url-s3_ lambda function for uploading the text file to _S3_. This is required to avoid storing _S3_ credentials in our React code.

2. Next, the text file is uploaded to S3 bucket and the fields _id_, _input_text_ and _input_file_path_ are recorded in our DynamoDb _INPUT_TABLE_ by our second lambda function _save-form-dynamo-db_. This table has been registered with _DynamoDB Stream_ which triggers our third lambda function _process-stream-dynamo-db_.

3. Now our third lambda function receives the new _id_ and creates a _EC2_ instance with appropriate configurations and a script with the _id_ to process the output.

4. Once the EC2 instance has been created, it fetches our script _process_output.sh_ from S3 bucket and runs it to:
- query our _INPUT_TABLE_ in Dynamodb by _id_.
- Fetches the _INPUT_FILE_ from S3 bucket.
- append _INPUT_TEXT_ to the _INPUT_FILE_ and store the output file to the S3 bucket.
- record the resulting _id_ and _output_file_path_ to our DynamoDb _OUTPUT_TABLE_.

5. After finishing the jobs, the VM shuts down resulting in termination of itself.

## References

1. https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/introduction/
2. https://docs.aws.amazon.com/lambda/latest/dg/configuration-envvars.html#configuration-envvars-runtime
3. https://docs.aws.amazon.com/cdk/v2/guide/hello_world.html
4. https://medium.com/tomincode/launching-ec2-instances-from-lambda-4a96f1264afb
5. https://docs.amplify.aws/react/start/getting-started/introduction/
6. https://docs.aws.amazon.com/cdk/api/v2/docs/aws-construct-library.html
7. https://github.com/awsdocs/aws-doc-sdk-examples
8. https://tailwindcss.com/docs/guides/create-react-app




