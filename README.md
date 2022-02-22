                              CREATING COST OPTIMIZED INFRASTRUCTURE BY LEVERAGING AWS LAMBDA SERVICES

In our current infrastructure, we have two environments: **_DEV_** and **_PROD_**

**_PROD_** **_ENVIRONMENT_**: Critical Environment. We want to have zero downtime in PROD.

**_DEV_** **_ENVIRONMENT_**: Non-Critical Environment. It means we can start and stop instances.
 
Usually, we do development in DEV env. from 9 am to 6 pm.

TO DO THIS... 

**There are multiple steps to take:**

**_1- Create a new IAM role_**:
  We need to create a new role to attach our policices.
  
**_2- Create two permissions_** - One is for getting the logs in Cloudwatch and the other is to have privilege to start and stop EC2 instances.

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "logs:CreateLogGroup",
                    "logs:CreateLogStream",
                    "logs:PutLogEvents"
                ],
                "Resource": "arn:aws:logs:*:*:*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "ec2:DescribeInstances",
                    "ec2:DescribeRegions",
                    "ec2:StartInstances",
                    "ec2:StopInstances",
                    "ec2:TerminateInstances"
                ],
                "Resource": "*"
            }
        ]
    }

**_3- Create a new Lambda Function and attach the role to the Lambda function_** 

**_Start_the_instances_**

    import boto3 

    def lambda_handler(event, context):

        #Get list of regions
        ec2_client = boto3.client('ec2')
        regions = [region['RegionName']
                    for region in ec2_client.describe_regions()['Regions']]


        #Itirate over the regions
        for region in regions:
            ec2 = boto3.resource('ec2',region_name=region)

            print("Region:", region)


            #Get only stopped instances
            instances = ec2.instances.filter(
                Filters=[{'Name': 'instance-state-name',
                            'Values': ['stopped']}])


            #Start the instances
            for instance in instances:
                instance.start()
                print('Started instance:', instance.id)
                
 **_Stop_the_instances_**
 
    import boto3 

    def lambda_handler(event, context):

        #Get list of regions
        ec2_client = boto3.client('ec2')
        regions = [region['RegionName']
                    for region in ec2_client.describe_regions()['Regions']]


        #Itirate over the regions
        for region in regions:
            ec2 = boto3.resource('ec2',region_name=region)

            print("Region:", region)


            #Get only running instances
            instances = ec2.instances.filter(
                Filters=[{'Name': 'instance-state-name',
                            'Values': ['running']}])


            #Stop the instances
            for instance in instances:
                instance.stop()
                print('Stopped instance:', instance.id)

**_4- Go to CloudWatch and build and Event under Amazon Event Bridge_**

  In order to reach the solution, we need to trigger Lambda function where will do thrugh CloudWatch.
  
**_5- Create EC2 Instances in two different regions as US-East-1 and US-West-1_**

  - US_East_1: 3 instances
  - US_East_2: 2 instancess
  
  


