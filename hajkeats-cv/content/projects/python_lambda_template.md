---
title: "Terraform Python Lambda Template"
date: "December 2020"
---

### [Repository](https://github.com/hajkeats/python_lambda_template)


### Lambda ready meals

One of the features of AWS I'm particularly fond of is [Lambda](https://aws.amazon.com/lambda/). The ability to launch a serverless function to the cloud which can be triggered asynchronously is incredibly handy, and the platform has a tonne of potential uses. A Lambda function could form part of a large microservices architecture or data pipeline, as well as serve as the backend for a simple bot, making queries to APIs or other AWS services. Its these small nifty jobs where it really thrives. Its also a very neat and low cost way to get code off the ground quickly. 

_"The AWS Lambda free usage tier includes 1M free requests per month and 400,000 GB-seconds of compute time per month."_

Since Lambda functions are 'serverless', AWS takes care of the hardware, environment and runtime, and the developer must simply concern themself with the code and it's packaging and deployment.

In order to take the packaging and deployment process off my mind, I've created a repository inspired by [Ruzin Saleem's](https://github.com/ruzin/terraform_aws_lambda_python), with all the Terraform Infrastructure-as-Code required to launch a basic Lambda function with a Python runtime. Also included are two bash scripts:

- `create_pkg.sh` which creates a Python virtual environment and packages it along with your code.
- `cleanup_pkg.sh` which removes the local copies of the packaged code and virtual environment created, once they are deployed to AWS.

Both scripts are triggered every time, using `null_resource` resources with `always_run` triggers, meaning each time you run `terraform apply`, you rebuild your function and push it to AWS. This simple workflow is ideal for continual development and testing of your Python function.

Moving forward I'll be adding the code to other repositories as a submodule, and making changes to incorporate various triggers, IAM roles and environment variables. I'll push the changes to branches specific to the repositories that use the submodule. Any improvements or fixes to the template repository will be pushed to master and then merged into the branches.