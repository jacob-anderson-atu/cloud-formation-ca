# cloud-formation-ca
## Lab 1 CA on Cloud Formation for ATU PgDip DevOps 

## Static S3 Website:
### Tutorial and Reference Material
* **tutorial:** medium.com/@elnala24/project-s3-static-website-with-cloudformation-2253652b7405
* **reference:** AWS CloudFormation User Guide
* **extensions:** AWS CloudFormation Snippets and CloudFormation Linter

### Improved Features
* Added an 'Outputs' block to print the web url using '!GetAtt S3Bucket.WebsiteURL' to avoid having to search the management interface.
* Enabled versioning on the s3 bucket as a simple website best practice
* Configured explicit security properties in 'PublicAccessBlockConfiguration' to define public read

### Deployment
Commands used are as follows
1. 'aws cloudformation create-stack --stack-name s3-web-stack --template-body file://static-website/template.yaml'
2. 'aws cloudformation describe-stacks --stack-name s3-web-stack'
3. 'aws s3 cp .\static-website\website\index.html s3://my-static-website-andersjr'

### Shortcomings
I attempted to create the index.html inline within the template.yaml in an effort to eliminate the need to copy the file over after creating the stack, but i couldn't easily figure out the cloudformation --capabilities required to execute the s3object transform block, so I scrapped the idea. I also recognize the bucket naming could be significantly better/dynamic for scaling. Also the security is basically just opening it up completely for viewing and could use some more refining.

## Three Tier Architecture
### Tutorial and Reference Materials
* **example project:** github.com/mertechcloud/3Tier_webapp_deployment_in_AWS
* **reference:** AWS CloudFormation User Guide
* **extensnions:** AWS CloudFormation Snipped and CloudFormation Linter
* **LLM/AI assistance:** No external LLM prompts were used. However, inline GitHub Copilot suggestions in VSCode were enabled. Suggested completions were reviewed before acceptance and the resulting code was validated and tested as part of the deployment process.

### Approach and Decisions
I used the example project I found as a solid starting foundation for my three tiered architecture. Some of the initial framework was pretty straightforward pulled from it, but the further in I got the more I saw their specific implementation was too nuanced for me so I made mine more generic and uniform. I also much prefer the YAML to JSON. 

My approach was to create two discrete EC2 instances for the web server and the application server, the example used user data scripts within a single EC2 instance. I thought that didn't show off the networking and security group configuration well enough. The example also used Auto Scaling Group for high availability, but I did not include that as I wanted to keep it relatively simple on my AWS free tier. The example also used an S3 bucket for website content, but as I just did that I wanted to just create the simple HTML pages inline upon initilization of the EC2 instances. I also did not enable SSH on my web server as I had no use for it and best practices say to close as much as possible.

My architecture is as follows:
Internet
|
HTTP port 80
|
Internet Gateway
|
Public Web Subnet
10.0.1.0/24
|
Web EC2 with Apache
|
TCP port 8080
|
Private App Subnet
10.0.2.0/24
|
App EC2 with Python http
|
TCP port 3306
|
Private Database Subnets
10.0.3.0/24 + 10.0.4.0/24
|
RDS (mysql)

Another significant change I made was the example only used a security group for the webserver and then used network ACLs for the appserver and database connections. I thought that felt confusing so I used security groups for all three. 

### Webserver and Appserver
For these I had some amount of prior knowledge with basic apache reverse proxy. So I created the basic html page inline on the webserver and then used a reverse proxy to pull a second html page from the appserver in order to validate the connection between the two. On the appserver I didn't have external network access to easily install packages so I just used a python http service that responds on port 8080 and serves up a single html page. When it came time to connect the appserver to the database I have a lot less confidence with databases so I left them connected by networks but not actively talking.

### Security / Scalability / Performance / Cost
I know there are a lot of avenues to improve upon what I created. In regards to security you can use the AWS managed database passwords to not have to input the database password at all. Naturally you can replace http with https for better security. I did however use security group to security group rules which is better than 0.0.0.0/0 between tiers. It currently isn't particularly scalable, but it could be improved using the Auto Scaling Groups feature, although that would increase cost. The currently reliability is not great considering each EC2 instance is a single point of failure. This could be addressed through high availability and deployment across availibility zones with load balancers but those are also costly and complex. As far as cost I deliberately avoided NAT Gateways, load balancers, Multi-AZ databases, Route 53, ASG, etc to keep costs low as well as simplicity, this does come with a tradeoff of lower availability/scalability/performance/etc.