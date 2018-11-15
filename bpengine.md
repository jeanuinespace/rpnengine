# BPENGINE


## OVERVIEW :: UML DIAGRAM

![enter image description here](https://raw.githubusercontent.com/jeanuinespace/rpnengine/master/img/sequencDiagram.png)

## IMPLEMENTATION

![enter image description here](https://raw.githubusercontent.com/jeanuinespace/rpnengine/master/img/Overview-AWS.png)
 - This project uses Eclipse IDE (Using Maven & JAX-RS)
	 - Maven (package the code in .war format)
	 - .war file can be implemented in docker container (can be deployed on Linux / Windows)
 - Tomcat 8.5 Targeted Runtime
 - Uses AWS platform to scale the resources
 - Upload the .war to AWS Elastic Beanstalk ([more    information](https://d1.awsstatic.com/aws-answers/AWS_Web_App_Deployment_Java.pdf)). The .war will then be deployed on EC2 as API server(s).
 - Front-END (static contents such as .html & javascripts can be hosted on S3).
 - Users will be authenticated with AWS Cognito. Authenticated users would be able to call the API server hosted on EC2. (For simplicity, Cognito was not implemented in the submitted version)
 - The process will be running in multi-threaded mode (backend) and the  final result will be returned to the front-end webpage.
 - Using [SQS](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-basic-architecture.html) for distributed nature of the system to ensure data integrity.
 - Using DynamoDB (NoSQL DB) to store transactions (to cater for online & offline processing)
 - Future: Consider using Lambda & AWS Batch to save cost

![enter image description here](https://raw.githubusercontent.com/jeanuinespace/rpnengine/master/img/Overview-FUTURE.png)

- To deploy the program on AWS:
	- Maven Clean > Build > Install to generate the .war file
	- .war file can be obtained from /bpengine/target/
	- Finally, upload the .war file in AWS Elastic Beanstalk. 
		- For simplicity, here is an external [link](https://github.com/snowplow/snowplow/wiki/Create-a-new-application-in-Elastic-Beanstalk-and-upload-the-WAR-file-into-it) containing the instructions on how to upload the generated output file (.war) to Beanstalk. (Previously I included the AWS Beanstalk screenshots in the .pptx, and those screenshots can still be retrieved from the github *img* folder - *refer as: Beanstalk<#>.png*)
	- Once the program has been successfully deployed, you can then make the API call via the endpoint generated from the Beanstalk. 
	![enter image description here](https://raw.githubusercontent.com/jeanuinespace/rpnengine/master/img/Beanstalk5.png)
![enter image description here](https://raw.githubusercontent.com/jeanuinespace/rpnengine/master/img/Beanstalk6.png)
- To make use of SNS & SQS in the program, you will need to create the following in AWS environment:
	- SNS Topic:
![enter image description here](https://raw.githubusercontent.com/jeanuinespace/rpnengine/master/img/Beanstalk8.png)
	- SQS:![enter image description here](https://raw.githubusercontent.com/jeanuinespace/rpnengine/master/img/Beanstalk7.png)

> **Tips:** The deployment can be automated with CodePipeline. Once CodePipeline detected the code changes (e.g. CodeCommit), it will then trigger CodeBuild to generate the output file with Maven and finally deploy the latest program in production. For further instructions on the [setup](https://docs.aws.amazon.com/codebuild/latest/userguide/sample-elastic-beanstalk.html#sample-elastic-beanstalk-codepipeline).  More information on [AWS CodePipeline & CICD Tools\]
](https://aws.amazon.com/blogs/devops/bluegreen-infrastructure-application-deployment-blog/)




## CODE FILES:
- Code Repository & the necessary documentation (javadocs) will be sent separately via email.
- Actual code resides in my CodeCommit repository. The necessary links & credentials will be sent to you via email so that you can gain access to the code.
- More information about connecting to [AWS CodeCommit](https://docs.aws.amazon.com/codecommit/latest/userguide/how-to-share-repository.html) (similar to GitHub - enables https/ssh access).


## ASSUMPTIONS & DELIVERIES:

### Offline Transactions:
- Able to process most of the mathematical expressions. Tested on the following mathematical expressions (based on what I understood from the requirements):
![enter image description here](https://raw.githubusercontent.com/jeanuinespace/rpnengine/master/img/MathExpressions.png)

- The parsing of the mathematical expressions (from inflix to RPN) were thoroughly verified through the following process:
	- https://en.wikipedia.org/wiki/Reverse_Polish_notation
	- https://en.wikipedia.org/wiki/Shunting-yard_algorithm
![enter image description here](https://raw.githubusercontent.com/jeanuinespace/rpnengine/master/img/Offline-Parsing.png)


### Online Transactions:
- Due to time constraint, the web interface setup rather *simply* whereby user can just upload the same offline file -> Copy a specific line of transaction -> Paste it on the textfield -> Click on Submit button to post the transaction to SQS.
- Upon clicking the submit button, the transaction will be sent to SQS via a POST API
![enter image description here](https://raw.githubusercontent.com/jeanuinespace/rpnengine/master/img/index-webpage.png)
- Eventually the queued data will then be stored in the DynamoDB for further processing
![enter image description here](https://raw.githubusercontent.com/jeanuinespace/rpnengine/master/img/API-to-SQS-Console.png)

- The following snapshots showcase the configured AWS environment in SQS & DynamoDB
![enter image description here](https://raw.githubusercontent.com/jeanuinespace/rpnengine/master/img/DynamoStructure.png)

	![enter image description here](https://raw.githubusercontent.com/jeanuinespace/rpnengine/master/img/SQS-Sample.png)



## TEST RESULT:

- Populate 2 million records on simple addition expression (e.g. 1000.00 + 2000 = 3000.0). Total file size: 67.1MB. Please click [here](https://github.com/jeanuinespace/rpnengine/raw/master/sample.txt.zip) for the compressed sample file. (To test out the program with the file, make sure you place the file in *Logs* folder)
- Accounts affected: 100225 & 201444
- The following were the observations:
	- Started at 9:21:01.pm.  Completed around 10:30p.m.
	- Estimated time to finish: 108mins  
	- Running on 4-threads (Playing video in the background). No error encountered.
	- CPU & Memory consumptions are relatively consistent (I would use a more scientific way to test, if I have more time...):	![enter image description here](https://raw.githubusercontent.com/jeanuinespace/rpnengine/master/img/CPU-RAM.png)
	- Spikes in DynamoDB (The NoSQL DB were configured to autoscale by default):
![enter image description here](https://raw.githubusercontent.com/jeanuinespace/rpnengine/master/img/DynamoASG.png)
	
		![enter image description here](https://raw.githubusercontent.com/jeanuinespace/rpnengine/master/img/DynamoChart.png)
	- Balance updated matches with the anticipated result (3000 * 2mil = 6 billion)
	![enter image description here](https://raw.githubusercontent.com/jeanuinespace/rpnengine/master/img/DynamoTxSum.png)

## CHALLENGES

### Assumptions:
- No timestamp & the order is not important in overall transaction.
- Online transactions can run at the same time as offline batch transactions.
- Free to use any technology
- Transactions can still proceed even though the balance is negative
- There is no irregularity in the input file (i.e. format of the log file is stated the same as per requirements)


### What can be improved? (Reflection):

- The overall architecture can be further optimised in terms of cost & performance.
- The infrastructure can be further expanded using docker containers & performance can be improved by caching similar mathematical expressions
- The interface can be further improved to upload files & final output presentation
- I was too engrossed in solving the REGEX and Mathematical expression. As a result, I have neglected the other aspects (e.g. application integration-deployment, validation, testing, performance tuning, error-handling & documentation)
- Deployment in AWS (including the roles & permissions) can be scripted in [CloudFormation](https://aws.amazon.com/cloudformation/) template, rather than configuring it via the console. This will further improve the clarity & consistency of the implementation instructions.
- Deployment consistency can be further improved by using AWS CICD (e.g. CodePipeline, CodeBuild & CodeDeploy)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTcwOTM2OTQzOSwxNTg1ODA4ODM4LDEzNz
E1NDIwMywyMDA1NzY5NTgxLDI1ODU4MzA4MV19
-->