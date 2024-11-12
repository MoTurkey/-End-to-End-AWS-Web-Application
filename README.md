# End-to-End-AWS-Web-Application
An AWS-powered application that calculates powers and saves results in DynamoDB.

## Architecture:
"The project uses AWS Lambda, API Gateway, and DynamoDB. A simple HTML frontend communicates with these AWS services."

![image](https://github.com/user-attachments/assets/89b23126-24ba-4829-988a-c49290975087)


## Technologies Used:
AWS Lambda
API Gateway
DynamoDB
HTML & JavaScript

## Setup Instructions:
1. Clone the repository.
2. Deploy the Lambda function using AWS Management Console.
3. Configure API Gateway to trigger the Lambda function.
4. Create a DynamoDB table to store results.

# Backend Code (Lambda Function)
# math_function.py

import json

def lambda_handler(event, context):
    base = event.get('base', 2)
    exponent = event.get('exponent', 3)
    
    result = base ** exponent
    return {
        'statusCode': 200,
        'body': json.dumps({'result': result})
    }

# Frontend Code (HTML and JavaScript)
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <h1>Exponentiation Calculator</h1>
        <form id="exponentiationForm">
            <label for="base">Base:</label>
            <input type="number" id="base" name="base" value="2">
            <label for="exponent">Exponent:</label>
            <input type="number" id="exponent" name="exponent" value="3">
            <button type="submit">Calculate</button>
        </form>
        <div id="result"></div>
    </div>
    <script src="script.js"></script>
</body>
</html>

    
    fetch(`https://your-api-id.execute-api.region.amazonaws.com/prod/exponentiation?base=${base}&exponent=${exponent}`)
        .then(response => response.json())
        .then(data => {
            document.getElementById('result').innerHTML = `Result: ${data.result}`;
        })
        .catch(error => {
            document.getElementById('result').innerHTML = 'Error: Unable to fetch result.';
            console.error('Error:', error);
        });

## Setting Up and Deploying the Application
### Deploy Lambda Function
Create Lambda Function:

In the AWS Management Console, navigate to Lambda and click on Create function.
Choose Author from scratch.
Set a function name, such as ExponentiationFunction.
Choose Python 3.x as the runtime.
Under Function code, you can either upload your math_function.py as a .zip file or directly paste the code into the inline editor.
Set the execution role to allow the Lambda function to interact with other services (e.g., DynamoDB if you use it).
Click Create function.
Test the Lambda:

Once the function is created, click Test to create a test event.
Use an event with sample input to test, like:
{
  "base": 2,
  "exponent": 3
}

#### Set Up API Gateway
Create API Gateway:

Go to API Gateway in the AWS Console and click Create API.
Choose REST API and click Build.
Set an API name like ExponentiationAPI and create the API.
Create Resource and Method:

Under the API, click Actions > Create Resource.
Name the resource /exponentiation.
Select the /exponentiation resource, click Actions > Create Method, and select GET.
In the GET method setup:
Set Integration type to Lambda Function.
In the Lambda Function field, type the name of the Lambda function (ExponentiationFunction).
Click Save and confirm the setup.
Deploy API Gateway:

Under Actions, click Deploy API.
Create a new deployment stage (e.g., prod).
After deployment, you’ll get the Invoke URL for your API. This URL will be used in the frontend JavaScript file to call the Lambda function.

#### Set Up DynamoDB
Create DynamoDB Table:

Go to DynamoDB in the AWS Console and click Create table.
Set the table name (e.g., ExponentiationHistory).
Define the primary key (calculationId as String).
You can leave other settings as defaults or adjust based on your needs.
Click Create.
Update Lambda to Use DynamoDB (Optional): If you want to store the results in DynamoDB after each calculation, modify your Lambda function to write results to the DynamoDB table. Here’s how you can adjust the Lambda function to do this:
import json
import boto3
from uuid import uuid4

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('ExponentiationHistory')

def lambda_handler(event, context):
    base = event.get('base', 2)
    exponent = event.get('exponent', 3)
    
    result = base ** exponent
    calculation_id = str(uuid4())
    
    # Store the result in DynamoDB
    table.put_item(
        Item={
            'calculationId': calculation_id,
            'base': base,
            'exponent': exponent,
            'result': result
        }
    )
    
    return {
        'statusCode': 200,
        'body': json.dumps({'result': result, 'calculationId': calculation_id})
    }

#### Deploy Frontend
Upload Frontend Files to S3:

Navigate to S3 in the AWS Console and create a new bucket for your frontend files (e.g., exponentiation-frontend).
Upload your index.html, style.css, and script.js files to the bucket.
Enable Static Website Hosting:

In the Properties tab of your S3 bucket, enable Static website hosting.
Set index.html as the index document.
Copy the Website URL that is generated.
Update Frontend Code:

Update the fetch URL in script.js to the Invoke URL from API Gateway:
fetch(`https://your-api-id.execute-api.region.amazonaws.com/prod/exponentiation?base=${base}&exponent=${exponent}`)
Replace your-api-id and region with the actual values from your API Gateway.

