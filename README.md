# AWS-lambda-testing

---

## Assumed Roles

When an AWS lambda interacts with another part of AWS (ie S3) it requires a specific role to be made so that it has access for to complete its task. These roles should be as limiting as possible, only allowing exactly what the lambda needs for it to run.

As a local user attempting to test the Lambda you need to be able to assume the role of the function in order for it to run.

Docs: https://docs.aws.amazon.com/cli/latest/userguide/cli-roles.html

1. Create a policy for your user that gives access to assume said role
2. Create a profile in your local machine under `~/.aws/config` for the role you are going to assume
3. Edit the trust relationship of the role to allow the user to assume it

Once these steps are complete you can now assume the role, to do this there is a helpful CLI tool: https://github.com/remind101/assume-role

In your lambdas package.json create a test script with a command like: 
`eval $(assume-role ROLE_NAME) && node test.js`

Running npm test will now assume the role of the lambda and run it locally!

## AWS Environment

Lambda functions can build and run differenctly in the AWS Lambda Environment then on your local machine. To ensure proper testing we can use a sandboxed local environment that mimics AWS Lambda: https://github.com/lambci/docker-lambda

To run your lamnda function inside of this container you can use the command: 
``--rm -v “$PWD”:/var/task lambci/lambda:nodejs6.10 index.handler “`cat sample.json`”``

To compile native dependencies in node_modules insdie this environment you can use the command:
`docker run --rm -v "$PWD":/var/task lambci/lambda:build-nodejs6.10`

## Using Both

To both assume a role and test it inside the AWS Lambda Environment you will need to grab the AWS environment variables from the assumed role and pass them into the docker container.

``docker run --env-file <(assume-role ROLE_NAME | sed ‘/^#/d’ | sed ‘s/^export //g’ | sed ‘s/“//g’) --rm -v “$PWD”:/var/task lambci/lambda:nodejs6.10 index.handler “`cat sample.json`”``
