
# AWS Web-Application using Amplify

## **Project Overview**
This Project is a complete end-to-end AWS web application.
It uses AWS services that are AWS Amplify, API Gateway, Lambda, DynamoDB, and IAM. The App will be deployed and hosted on AWS Amplify and will involve performing mathematical operations through Lambda Function that can be triggered by the user with Help of REST API integerated in the App created through AWS API Gateway. The result of the math operation will be stored in DynamoDB and will also be prompted in Web-app as well. 
## **AWS Services Used**
* #### **AWS Amplify** : *For Hosting a Webpage* 
* #### **AWS API Gateway**: *To Invoke the Lambda Math Function*
* #### **AWS Lambda**: *Performing Math Operation by Function* 
* #### **DynamoDB**: *To Store the Calulated Math Result*  
* #### **IAM**: *To Handle Permissions* 
## **Application Architecture**


![image](https://user-images.githubusercontent.com/94673191/217528364-92a9a29e-0e69-4c6f-8a67-f8924849da6f.png)

### Hosting a Webpage using AWS Amplify
 *To deploy and host the webpage, we will be using AWS Amplify.*
* First, we will create a zip file containing the index.html file and upload it to AWS Amplify.
* The deployment process as result will provide us with a domain link that users can use to access the deployed app.

### Building Serveless Math Function Using AWS Lambda 
 *To perform mathematical operations, we will use AWS Lambda, a serverless platform.*
* We will write a python code to perform the required mathematical operations that will be executed when the Lambda function is triggered.
* The code will be deployed and configured with a test event to verify its functionality.

### Linking Lambda Function to Web-App by using AWS API Gateway
 *To allow users to access the math function, we will create a REST API using AWS API Gateway.*
* We will create a POST method for the API to send data to the server.
* The API Gateway will be integrated with the Lambda function and will be given permission to invoke the function.
* We will enable cross-origin resource sharing (CORS) for the POST method in the resource actions to ensure that the API is accessible from any origin.
* Upon deployment of the API, an invoke URL will be generated that can be used to perform the actual function of the API.

### Creating NoSQL Database using DynamoDB
 *To store the result of the math operation, we will use DynamoDB, a key-value NoSQL database.*
* We will create a table in DynamoDB with the partition key as ID.
* The ARN of the created table will be stored to be used for granting the Lambda function IAM permission to write to the DynamoDB table.

### Handling Permissions using IAM
 *To manage access to different parts of the application, we will handle permissions using IAM.*
* The Lambda function will be granted permission to write to the DynamoDB table using IAM policies.

### Integrating API with Webpage
*One part is missing that is there is still no connection between AWS Amplify and API Gateway. In short, there is no way to invoke the created REST API through the app.To use the Math Functionality*

* To make this connection, we need to update the code in the index.html file and add the API invoke URL. 
* Then, we need to redeploy the code to AWS Amplify will automatically perform the redeployment changes and host it smoothly.

## **Testing and understanding working of Web-App**
*Now that we have completed building and architecting our Web-App which is fully functional.
We can test it by entering the values into App then click Calculate button after which we observe it will provide propmt with the result. Also if we see in Items of DyanamoDB table we can see the result is also stored there.*

### What did really happen internally ?
* At first when we clicked the calculate button it called the API Gateway which triggered the Lambda Function.
* The API along with invoking the Math Function provided it values using POST Method of REST API. 
* Then the Lambda Function Calculated the result and wrote it into the DynamoDB table. 
* Also the result which we got at prompt into Web-App it was fetched through API Gateway. 
## **Reference Codes** 

### Lambda Function Python Code
```python
# import the JSON utility package
import json
# import the Python math library
import math

# import the AWS SDK (for Python the package name is boto3)
import boto3
# import two packages to help us with dates and date formatting
from time import gmtime, strftime

# create a DynamoDB object using the AWS SDK
dynamodb = boto3.resource('dynamodb')
# use the DynamoDB object to select our table
table = dynamodb.Table('PowerOfMathDatabase')
# store the current time in a human readable format in a variable
now = strftime("%a, %d %b %Y %H:%M:%S +0000", gmtime())

# define the handler function that the Lambda service will use an entry point
def lambda_handler(event, context):

# extract the two numbers from the Lambda service's event object
    mathResult = math.pow(int(event['base']), int(event['exponent']))

# write result and time to the DynamoDB table using the object we instantiated and save response in a variable
    response = table.put_item(
        Item={
            'ID': str(mathResult),
            'LatestGreetingTime':now
            })

# return a properly formatted JSON object
    return {
    'statusCode': 200,
    'body': json.dumps('Your result is ' + str(mathResult))
    }
```

### Lambda Function IAM Execution Role Policy 
```json
{
"Version": "2012-10-17",
"Statement": [
    {
        "Sid": "VisualEditor0",
        "Effect": "Allow",
        "Action": [
            "dynamodb:PutItem",
            "dynamodb:DeleteItem",
            "dynamodb:GetItem",
            "dynamodb:Scan",
            "dynamodb:Query",
            "dynamodb:UpdateItem"
        ],
        "Resource": "DYNAMO-DB-TABLE-ARN"
    }
    ]
}
```

### Javascript used For Calling API and fetching Result. 
```javascript
        // callAPI function that takes the base and exponent numbers as parameters
        var callAPI = (base,exponent)=>{
            // instantiate a headers object
            var myHeaders = new Headers();
            // add content type header to object
            myHeaders.append("Content-Type", "application/json");
            // using built in JSON utility package turn object to string & store in a variable
            var raw = JSON.stringify({"base":base,"exponent":exponent});
            // create a JSON object with parameters for API call and store in a variable
            var requestOptions = {
                method: 'POST',
                headers: myHeaders,
                body: raw,
                redirect: 'follow'
            };
            // make API call with parameters and use promises to get response
            fetch("INVOKE-URL-API", requestOptions)
            .then(response => response.text())
            .then(result => alert(JSON.parse(result).body))
            .catch(error => console.log('error', error));
        }
```
