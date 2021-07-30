# AWS SAM and GitHub Actions

AWS SAM is used to create a small serverless application, a single AWS Lambda Python 3.8 function invoked by an Amazon API Gateway endpoint. Once the code is pushed to GitHub, a GitHub Actions workflow triggers a GitHub CI/CD pipeline to build and deploy the code to the targeted AWS account.

AWS blog posts on this topic:

* https://aws.amazon.com/blogs/compute/using-github-actions-to-deploy-serverless-applications/

## Definitions

### GitHub Actions Runner

The application that runs a job from a GitHub Actions workflow. A GitHub hosted runner is a virtual machine hosted by GitHub with the runner application installed.

## Prerequisites

* AWS CLI
* AWS SAM CLI
* An Amazon S3 bucket in the working AWS account to store the deployment build package

## Steps Taken

### GitHub Repo

Created this GitHub repo and cloned it locally.

### Make a SAM application

AWS SAM has several quick-start templates to help create an application, one of which I used for this project. It's a hello-world python application. [More on this here](https://docs.aws.amazon.com/serverlessrepo/latest/devguide/serverlessrepo-quick-start.html).

If you want a different runtime than python 3.8, run `aws sam init -h` for all the things you can choose from.

Run this command at the root of the cloned repository directory to create the application, changing the `-n` value to the name you want for your SAM application. Choose zip when asked if you want a zip or an image.

```bash
sam init -r python3.8 -n github-actions-with-aws-sam --app-template "hello-world"
```

## Check that Things are Wired Up Correctly (Locally)

AWS SAM allows for testing the applications locally, since it has a default event in `events/event.json`.

### Test the Lambda
Invoke the Lambda function by navigating into the root of the SAM app directory and passing the default event:

```bash
sam local invoke HelloWorldFunction -e events/event.json
```

Note that `HelloWorldFunction` is the logical id of the lambda resource found in the `template.yaml` file at the root of the SAM app.

You should get a 200 status code and a `hello world`.

### Test the API Gateway

Start the API locally with:

```bash
sam local start-api
```

This tells AWS SAM to launch a Docker container with a mock API Gateway endpoint listening on localhost:3000.

Use curl to call the hello API (in a second terminal tab) or paste the URL into a browser window. You should see the the output of HTML either way.

```bash
curl http://127.0.0.1:3000/hello
```

The `/hello` page is defined by the template file, which defines the HelloWorldEvent as having as a path property of `/hello`.

### Update the Lambda Code

The Lambda function initially had a simple JSON output of `{hello world}` from a `"body": json.dumps...` after being created by the SAM init command. I updated the Lambda to have a return of HTML instead.

#TODO: Set up the lambda to read an html file stored outside of the lambda
htmlFile = open('content/html-file.html', 'r')
