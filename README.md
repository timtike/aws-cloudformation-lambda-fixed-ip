# aws-cloudformation-lambda-fixed-ip
aws cloudformation yaml template to give lambda a fixed public ip


### The problem
One of our third parties required us to send data received by a client to their API which only accepted requests from whitelisted IP addresses. AWS Lambda is not the same an as EC2 instance as it runs on containers within the AWS infrastructure. Traffic would appear to be coming from certain IP addresses but there is no way to configure which IP address is used meaning that the IP address that the requests are sent from will not be the same.

### The solution
AWS Lambda supports executing your code from inside a VPC. With this ability we’re able to create a NAT (Network Address Translator) Gateway so that all out-bound connections from our lambda functions will exit from the NAT which is assigned to a fixed IP address. This fixed IP adress can then be whitelisted by our third-parties.

steps:
Create a new VPC to run your code in (or use an existing VPC)
Create a new Internet Gateway to communicate with the Internet from inside your VPC
Create a Public Subnet and add a new route to the route table which routes to your Internet Gateway from 0.0.0.0/0
Create a new Elastic IP address.
Create a new NAT Gateway and assign it to the Public Subnet and Elastic IP address that you just created.
Create a Private Subnet and add a new route to the route table which routes to your NAT Gateway from 0.0.0.0/0


you can run this cloudformation file to create all above infra and then add VPC&Private subnet to lambda

### How to Test

an api can get your public ip and send it to you, just run below code in your lambda to see if the response.json()["query"] match the created Elastic IP

```
import json
import requests

def lambda_handler(event, context):
    response = requests.get('http://ip-api.com/json/')
    print(response.json())
    print(f'===IP:{response.json()["query"]} ====')  
    return {
        'statusCode': 200,
        'body': json.dumps('completed!')
    }

```

output like:
```
{'status': 'success', 'country': 'United States', 'countryCode': 'US', 'region': 'VA', 'regionName': 'Virginia', 'city': 'Ashburn', 'zip': '20149', 'lat': 39.0438, 'lon': -77.4874, 'timezone': 'America/New_York', 'isp': 'Amazon.com, Inc.', 'org': 'AWS EC2 (us-east-1)', 'as': 'AS14618 Amazon.com, Inc.', 'query': '52.54.55.18'}
===IP:52.54.55.18 ====
```
