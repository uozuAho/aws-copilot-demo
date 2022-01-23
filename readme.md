# AWS Copilot demo

Example of building and deploying an app using
[AWS Copilot](https://aws.amazon.com/containers/copilot/).

Created by following the tutorials here:
- https://aws.amazon.com/blogs/containers/introducing-aws-copilot/
- https://aws.amazon.com/blogs/containers/automatically-deploying-your-container-application-with-aws-copilot/

# Creating this project from scratch
Follow these steps to create your own copy of this site on AWS.

## install stuff
- sign up to AWS
- install [docker](https://www.docker.com/products/docker-desktop)
- install [aws cli](https://aws.amazon.com/cli/)
- install [copilot](https://aws.github.io/copilot-cli/docs/getting-started/install/)
- clone this repo
- `rm .workspace manifest.yml`

## build and run the site locally
The app/site is just a simple ngnix server. See the [Dockerfile](./Dockerfile).

```sh
docker build -t aws-copilot-demo .
docker run -d --rm -p 80:80 aws-copilot-demo
curl localhost
# You should see whatever is in index.html
```

## init copilot, run the app on AWS!
This will get the web app up and running on AWS.

```sh
copilot init
# choose an app name
# select 'Load Balanced Web Service'
# choose a service name (can be same as app name)
# choose the Dockerfile in this repo
# copilot starts deploying...
# ...
# select 'yes' for a test environment
# copilot creates all necessary infrastructure (phew, no cloudformation!)
# ...
# eventually, you'll get a link to your app!
# You can get the links at any time with:
copilot svc show
```

At this point, you'll have a number of cloudformation stacks. My knowledge of
cloudformation is a little shaky, but here's my understanding of each stack's
purpose. Assuming an app name of aws-copilot-demo:

- `aws-copilot-demo-infrastructure-roles`: IAM roles specific to this application
- `StackSet-aws-copilot-demo-infrastructure-*`: storage for build artifacts
- `aws-copilot-demo-test`: infrastructure template for the test environment
- `aws-copilot-demo-test-aws-copilot-demo`: deployed test environment

## Creating a separate production environment
The steps above created a 'test' environment. Create another one:

```sh
copilot env init
# name it something like production
# choose defaults
# copilot starts building infrastructure...
# ...
```

This creates another cloudformation stack template with the name '*-production'.
The site's not running yet though.

Update [manifest.yml](./copilot/aws-copilot-demo/manifest.yml):

```diff
-#environments:
-#  test:
-#    count: 2               # Number of tasks to run for the "test" environment.
+environments:
+  # test:
+  production:
+    # Same as test. This just demonstrates that these values can be overridden
+    # per environment.
+    cpu: 256
+    memory: 512
+    count: 1
```

Then run

```sh
copilot svc deploy --env production
```

Once this finishes, you should get a link to your production app, and have two
new cloudformation stacks:

- `aws-copilot-demo-production`: infrastructure template for the production environment
- `aws-copilot-demo-production-aws-copilot-demo`: deployed production environment

## ops n stuff
```sh
# stream app logs to your console
copilot svc logs --follow
# you can also see these logs in AWS -> cloudwatch -> logs -> log groups ->
#   /copilot/aws-copilot-demo-[test|production]-aws-copilot-demo
```

## deploying on push
If you haven't already, create a github repo and push your changes. Then:

```sh
copilot pipeline init
# select test & production environments
```

This creates a (quite large and complicated!) `buildspec.yml` file, and
`pipeline.yml` file that define the build and deployment process. Commit these,
then:

```sh
git push
copilot pipeline update
# You will be prompted to add the AWS app to your github repo/account, in order
# to allow AWS/copilot to watch your repo for changes.

# You can check the pipeline status with:
copilot pipeline status
# You can also see the pipeline status at AWS -> CodePipeline -> Piplines ->
#   pipeline-aws-copilot-demo-aws-copilot-demo
```

Now whenever you push changes to git, the pipeline deploys changes to the test
and production environments. Remember you can see the test & prod URLS with
`copilot svc show`.

## tests
The test environment deployment has test steps. In `pipeline.yml`, change the
`test_commands` from `exit 0` to `exit 1`. This simulates a failing test. Now:

```sh
# Pipeline updates are required after all changes to the pipeline :(
copilot pipeline update
git push
```

## deleting everything
```sh
copilot app delete
```

# todo
- how to monitor costs?
- how to monitor health?
- how to customise the generated stacks? Eg. if I want to define some metrics/
  alerts, can I put the cloudformation for them somewhere and have copilot
  deploy it?
