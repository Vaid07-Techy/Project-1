## Serverless Image Upload Workflow using AWS S3 and Lambda

 Project Overview :

This project implements a serverless image-upload workflow using Amazon S3, AWS Lambda, IAM, and Amazon CloudWatch Logs.

When an image is uploaded to an Amazon S3 bucket, an S3 ObjectCreated event automatically triggers an AWS Lambda function. The Lambda function extracts details about the uploaded object and records the execution information in CloudWatch Logs.

The project demonstrates an event-driven serverless architecture without requiring an EC2 server.

```
                          USER
                           │
                           │ Upload Image
                           ▼
                  ┌─────────────────┐
                  │    Amazon S3    │
                  │   Image Bucket  │
                  └────────┬────────┘
                           │
                           │ ObjectCreated Event
                           ▼
                  ┌─────────────────┐
                  │   AWS Lambda    │
                  │                 │
                  │ image-upload-   │
                  │ handler         │
                  └────────┬────────┘
                           │
                           │ Execution Logs
                           ▼
                  ┌─────────────────┐
                  │   CloudWatch    │
                  │      Logs       │
                  └─────────────────┘
```

<h3>Workflow :</h3>

- The workflow operates as follows:

1. A user uploads an image to an S3 bucket.

2. Amazon S3 detects the new object.

3. S3 generates an ObjectCreated event.

4. The event automatically invokes the Lambda function.

5. Lambda receives the S3 event as a JSON object.

6. Lambda extracts:

7. S3 bucket name

8. Uploaded object name/key

9. Object size

10. Event type

11. Event timestamp

12. Lambda writes the extracted information to CloudWatch Logs.

13. CloudWatch stores the Lambda execution logs for monitoring and troubleshooting.

<h2>Process :</h2>

<h3>1. Create S3 Bucket</h3>

Create an S3 bucket with a unique name.

Example:
```
serverless-image-upload-2026
```

The bucket stores uploaded images.

Example:
```
serverless-image-upload-2026/
│
├── cat.jpg
├── dog.jpg
└── flower.png
```

The bucket does not need to be publicly accessible for this project.

<b> Setting :</b>

Go to Amazon S3 > Buckets > click "Create Bucket" > select Bucket Type "General Purpose" > Keep default "Global Namespace" selected >set name > click "Create Bucket"

![](images/Screenshot%202026-10-04%20145444.png)


<h3>2. Create AWS Lambda Function</h3>

Function Name
```
image-upload-handler
```

Runtime
```
Python 3.12
```

Lambda Code
```
import json
import urllib.parse
from datetime import datetime


def lambda_handler(event, context):

    print("Lambda function started")
    print("Received event:")
    print(json.dumps(event, indent=2))

    # Get S3 event information
    record = event["Records"][0]

    bucket_name = record["s3"]["bucket"]["name"]
    object_key = urllib.parse.unquote_plus(
        record["s3"]["object"]["key"]
    )

    object_size = record["s3"]["object"].get("size")
    event_time = record["eventTime"]
    event_name = record["eventName"]

    print("===== IMAGE UPLOAD DETAILS =====")
    print(f"Bucket Name : {bucket_name}")
    print(f"Object Name : {object_key}")
    print(f"Object Size : {object_size} bytes")
    print(f"Event Type  : {event_name}")
    print(f"Event Time  : {event_time}")
    print("================================")

    return {
        "statusCode": 200,
        "body": json.dumps({
            "message": "Image upload processed successfully",
            "bucket": bucket_name,
            "object": object_key,
            "size": object_size,
            "event": event_name
        })
    }
    
```
<b>Setting :</b>

Go to Lambda > Functions > Click "Create Function" > select "Author From Scratch" >set function name > select runtime > click "Create Function"

![](images/Screenshot%202026-10-04%20145725.png)

Go inside your cerated function > under "Code" tab add your python code > click "Deploy"

![](images/Screenshot%202026-10-04%20150055.png)

<h3>3. Connect S3 to Lambda</h3>

I created an S3 Event Notification to automatically trigger my Lambda function whenever an image is uploaded to the S3 bucket. It enables event-driven processing, so I don't need to manually invoke Lambda after every upload.

<b>setting :</b>

Go to S3 > Buckets > go inside your > under properties tab > go to Event Notification setting > click "Create event notification" > set name > under "Event Type" setting select "All object create events" > under "Destination" setting select "lambda Function" > select "choose from your lambda Function" and then select your created lambda function > click "save changes"

![](images/Screenshot%202026-10-04%20150359.png)

![](images/Screenshot%202026-10-04%20150435.png)

![](images/Screenshot%202026-10-04%20150606.png)

<h3>4. Upload Image to Bucket</h3>

Go to S3 > Bucket > go inside your Bucket > under "object" tab click "Upload" > click "Add files" > browse and select the file you want to upload > click "upload"

![](images/Screenshot%202026-10-04%20150926.png)

you will see your uploaded file here.

<h3>5. Verify the process</h3>

now verify whether lambda executed or not

Go to Cloudwatch > logs > log management > here you will see your log group >go inside log group > under "log stream" function you will see your log stream go inside that > now you will see your recorded logs after uploading an image

![](images/Screenshot%202026-10-04%20155107.png)

![](images/Screenshot%202026-10-04%20155140.png)

![](images/Screenshot%202026-10-04%20155032.png)

![](images/Screenshot%202026-10-04%20155047.png)

<b>log group : </b>

A CloudWatch Log Group is a container that organizes logs from a particular application or AWS service.
Think of it like a folder for logs.

In our project
Lambda function is:
```
image-upload-handler
```

AWS automatically created this Log Group:
```
/aws/lambda/image-upload-handler
```

Inside that Log Group, AWS stores the logs generated by your Lambda executions.

<b>log stream :</b>

A Log Stream is a sequence of log events generated by a particular execution environment of your application or Lambda function.

Lambda can have multiple executions and multiple execution environments running over time. CloudWatch uses log streams to organize those execution logs rather than putting everything into one giant stream.

```
CloudWatch
   ↓
Log Group = Folder
   ↓
Log Stream = Individual stream/file inside the folder
   ↓
Log Events = Actual log messages
```