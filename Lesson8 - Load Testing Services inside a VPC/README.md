# Lesson 7: Load Testing Services in a VPC
Goal: create a script to test a service in a VPC by giving lambda a static IP address and whitelisting that IP address.

### Step 1: Create a VPC
[Create a VPC with a public and private subnet.](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Scenario2.html)
Nordstrom developers should contact Public Cloud to request a new VPC.

### Step 2: Create an Internet Gateway
[Create an Internet Gateway to get access to the internet from your VPC.](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Internet_Gateway.html)

### Step 3: Create a Public Subet
[Create a a Public Subnet .](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Internet_Gateway.html)

### Step 4: Create an Elastic IP Address
This will be the IP address of your lambda.

### Step 5: Create a NAT Gateway
[Create a new NAT Gateway and assign it to the Public Subnet and Elastic IP address that you just created.](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-nat-gateway.html)


### Step 6: Create a Private Subnet
[Create a Private Subnet and add a new route to the route table which routes to your NAT Gateway.](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-nat-gateway.html)


### Step 7: Create Your Script
Add your VPC and Public Subnets to your serverless.yml file.
```
service: ******
provider: ******
  name: ******
  runtime: ******
  vpc:
    securityGroupIds:
      - "******"
    subnetIds:
      - "******"
```
### Step 8: Start Your Load Test
deploy your code and invoke serverless artillery to start your load test.
```
slsart deploy
slsart invoke
```
