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


