# Creating AWS Lambda Functions using API Gateway

## Creating a CloudFormation role and policy for AWS Lambda

A Lambda Function will run under an IAM Role which can give it access to any required resources.

A basic role and policy look like this:

```json
        "role": {
            "Type": "AWS::IAM::Role",
            "Properties": {
                "AssumeRolePolicyDocument": {
                    "Version": "2012-10-17",
                    "Statement": [
                        {
                            "Effect": "Allow",
                            "Principal": {
                                "Service": [
                                    "lambda.amazonaws.com",
                                    "apigateway.amazonaws.com"
                                ]
                            },
                            "Action": ["sts:AssumeRole"]
                        }
                    ]
                },
                "Path": "/"
            }
        },

        "policy": {
            "DependsOn": [
                "role"
            ],
            "Type": "AWS::IAM::Policy",
            "Properties": {
                "PolicyName": "LegislationHTMLPolicy",
                "Roles": [
                    {"Ref": "role"}
                ],
                "PolicyDocument": {
                    "Version": "2012-10-17",
                    "Statement": [
                        {
                            "Effect": "Allow",
                            "Action": "lambda:InvokeFunction",
                            "Resource": "*"
                        },
                        {
                            "Effect": "Allow",
                            "Action": ["logs:*"],
                            "Resource": ["arn:aws:logs:*:*:*"]
                        },
                        {
                            "Effect": "Allow",
                            "Action": [
                                "cloudwatch:*"
                            ],
                            "Resource": "*"
                        },
                        {
                            "Effect": "Allow",
                            "Action": "ec2:*",
                            "Resource": "*"
                        },
                        {
                            "Effect": "Allow",
                            "Action": "s3:*",
                            "Resource": "*"
                        }
                    ]
                }
            }
       },
```

The role allows both Lambda and AWS Gateway to use it.

The policy allows access to lambda (for API gateway to invoke it), cloudwatch (for writing log messages), and s3 (for accessing buckets).  

I think we needed to add ec2 rights in order to access some URIs within EC2 (e.g. load balancers and instances).


## Writing Lambda Functions in .NET using API Gateway Proxy Integration

API Gateway receives HTTP requests and can invoke Lambda functions.  

Query String Parameters can be defined in API Gateway and be passed to the Lambda function as parameters.  This requires writing a mapping for the parameters in the API gateway.  I think it doesn't have much support for URI path parameters.

The API Gateway Proxy Integration feature allows us to pass all of the information about the request into the Lamdba function, and process it there.  Then the API Gateway doesn't need to know how to read our requests.

To create the project:

* Install AWS Toolkit for Visual Studio
* Create a new project of type "AWS Lambda Project (.NET Core - C#)"
* Add nuget package "Amazon.Lambda.APIGatewayEvents".  

Then implement function code, a starting point looks like this:

```csharp
using System;
using System.Collections.Generic;
using Amazon.Lambda.Core;
using Amazon.Lambda.APIGatewayEvents;

[assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.Json.JsonSerializer))]

namespace Example
{
    public class Function
    {
        ILambdaLogger logger = null;

        public APIGatewayProxyResponse FunctionHandler(APIGatewayProxyRequest request, ILambdaContext context)
        {
            logger = context.Logger;
            logger.Log("Your URI was: " + request.Path == null ? "(none)" : request.Path);
 
            return new APIGatewayProxyResponse
            {
                StatusCode = (int)HttpStatusCode.OK,
                Body = "<p>Hello world</p>",
                Headers = new Dictionary<string, string> {
                    {
                        "Content-Type", "text/html"
                    }
                }
            };
        }
}
```

From this code, note that:

* The APIGatewayProxyRequest input parameter contains all of the information about the request.  To recieve this parameter we have to enable "use lambda proxy integration" in the API gateway settings.
* The request.Path contains the URI path of the API gateway request.  This allows us to parse the URI path.
* The APIGatewayProxyResponse allows us to set the HTTP response from the API Gateway.  We can set the response code, headers and the response body.

To deploy the function to AWS:

* Right click the project and "Publish to AWS Lambda"
* Specify the AWS account using an Access Key and Secret Key
* Give the function a name that it will be shown by in AWS Lambda.  You can deploy the same function with different names.
* Select the IAM Role (which we created in CloudFormation earlier)
* Set the memory limit.  This also sets the CPU performance.  The higher we set the memory limit the more CPU performance we receive.
* If the Lambda accesses EC2 instances in a VPC, then set the VPC subnet and security group.  If these are set the Lambda won't have internet access unless there is a NAT in the VPC.  Leave them unset and the Lambda has Internet access but can't access resources in the VPC.
* Click Upload

Now skip to the API Gateway section to connect an API Gateway to the function.

Other notes:

* The C# Lambda Function uses .NET Core framework.  Not all C# libaries support .NET Core.  (Saxon does not)  


## Writing Lambda Functions in Java using API Gateway Proxy Integration

To create the project:

* Download the AWS Toolkit for Eclipse and extract it to a folder.
* Run Eclipse
* Click Help -> Install New Software
* Click Click Add -> Local -> and select the extracted folder.
* Select Core and Deployment tools.
* From the AWS Toolbar, click AWS -> Preferences and configure the Access Key and Secret Key
* From the AWS Toolbar -> click AWS -> AWS Lambda Java Project
* Set a project name, group ID, artifact ID, class name as usual

Then implement function code, a starting point looks like this:

```java
package uk.gov.legislation.lambda.test;

import org.json.simple.JSONObject;
import org.json.simple.parser.JSONParser;

import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.LambdaLogger;
import com.amazonaws.services.lambda.runtime.RequestStreamHandler;

public class LambdaFunctionHandler implements RequestStreamHandler {

	LambdaLogger logger = null;
	
	@Override
    public void handleRequest(InputStream inputStream, OutputStream outputStream, Context context) throws IOException {
 
        logger = context.getLogger();
		    		    
        JSONParser parser = new JSONParser();
        BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
        JSONObject request = (JSONObject) parser.parse(reader);

        // do something here
        logger.Log( (String) request.get("path"));

        JSONObject responseJson = new JSONObject();
        JSONObject headerJson = new JSONObject();
	    headerJson.put("Content-Type", "text/html");
        responseJson.put("statusCode", 200);
        responseJson.put("headers", headerJson);
        responseJson.put("body", "<p>Hello world</p>");

        OutputStreamWriter writer = new OutputStreamWriter(outputStream, "UTF-8");
        writer.write(responseJson.toString());
        writer.close();
    }
}
```

In pom.xml the dependencies are:

```xml
    <dependency>
      <groupId>com.amazonaws</groupId>
      <artifactId>aws-lambda-java-core</artifactId>
      <version>1.1.0</version>
    </dependency>
	<dependency>
	    <groupId>com.amazonaws</groupId>
	    <artifactId>aws-java-sdk-s3</artifactId>
	    <version>1.11.603</version>
	</dependency>
    <dependency>
	    <groupId>com.googlecode.json-simple</groupId>
	    <artifactId>json-simple</artifactId>
	    <version>1.1.1</version>
	</dependency>	
```

From this code, note that:

* The InputStream and OutputStream will contain JSON to communicate with API Gateway.  We can decode and encode it with a standard JSON library.
* The InputStream input parameter contains all of the information about the request.  To recieve this parameter we have to enable "use lambda proxy integration" in the API gateway settings.
* The request.Path contains the URI path of the API gateway request.  This allows us to parse the URI path.
* The OutputStream allows us to set the HTTP response from the API Gateway.  We can set the response code, headers and the response body.

To deploy the function to AWS:

* Right click -> AWS Lambda -> Upload function to AWS Lambda
* Give the function a name that it will be shown by in AWS Lambda.  You can deploy the same function with different names.
* Set the memory limit.  This also sets the CPU performance.  The higher we set the memory limit the more CPU performance we receive.
* Select the IAM Role (which we created in CloudFormation earlier)
* Click upload

Note that any settings for the function that you can't set in the Eclipse UI can be set in the AWS Console under Lambda.  E.g. the VPC settings.


## Other general Lambda Function notes

* Global variables may persist between invocations of the function.
* The /tmp filesystem is available for use.  Files there may persist between invocations of the function.
* The maximum time for a Lambda Function to run is currently 15 minutes.  The API Gateway time is much shorter, at 30 seconds.  For longer functions we'd probably need to invoke them using the AWS SDK.
* The maximum memory for a Lambda Function is currently 3008MB.  The higher the memory limit specified, the more CPU percentage is allocated.  (so setting a higher limit means the function runs faster)
* The API Gateway currently has a request/response size limit of 6MB.  For dealing with larger objects the Lambda Function will need to interact with S3 or any other external API.

## Setting up API Gateway to call a Lambda Function using API Gateway Proxy Integration

API Gateway can call our Lambda Function when it receives HTTP requests.

To create the API:

* In AWS Console go to API Gateway
* Click Create API
* Give the API a name
* Click Actions->Create Resource
* To proxy all requests matching / or below, set the Resource Path to /{proxy+}
* Click Actions->Create Method and set the method type to ANY
* Click Integration Request and set the method to run our Lambda Function.  Tick the box called Use Lambda Proxy Integration.   
* Click Actions->Deploy API 

You will be given the URI for the API.

Note that:

* If you make any change to the API you need to re-run the Deploy API action to make it take effect.


# References

* https://docs.aws.amazon.com/toolkit-for-eclipse/v1/user-guide/lambda-tutorial.html
* https://www.baeldung.com/aws-lambda-api-gateway


