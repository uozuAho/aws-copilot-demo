# AWS Copilot demo

Example of building and deploying an app using
[AWS Copilot](https://aws.amazon.com/containers/copilot/).

Follows the tutorial here: https://aws.amazon.com/blogs/containers/introducing-aws-copilot/

# Creating this project from scratch
This is how this repo + sit was created. You should be able to create your own
copy of this site by following these steps.

## install stuff
- sign up to AWS
- install [docker](https://www.docker.com/products/docker-desktop)
- install [aws cli](https://aws.amazon.com/cli/)
- install [copilot](https://aws.github.io/copilot-cli/docs/getting-started/install/)
- clone this repo
- `rm .workspace manifest.yml`

## init copilot, run the app!
This will get the web app up and running on AWS. The app is just a simple ngnix
server. See the [Dockerfile](./Dockerfile).

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
```

At this point, you'll have a number of cloudformation stacks. My knowledge of
cloudformation is a little shaky, but here's my understanding of each stack's
purpose. Assuming an app name of aws-copilot-demo:

- aws-copilot-demo-infrastructure-roles: IAM roles specific to this application
- StackSet-aws-copilot-demo-infrastructure-*: storage for build artifacts
- aws-copilot-demo-test: infrastructure template for the test environment
- aws-copilot-demo-test-aws-copilot-demo: deployed test environment

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

Update [](./copilot/aws-copilot-demo/manifest.yml):

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

- aws-copilot-demo-production: infrastructure template for the production environment
- aws-copilot-demo-production-aws-copilot-demo: deployed production environment

## ops n stuff
```sh
# stream app logs to your console
copilot svc logs --follow
# you can also see these logs in AWS -> cloudwatch -> logs -> log groups ->
#   /copilot/aws-copilot-demo-[test|production]-aws-copilot-demo
```

# todo
- how to bring down the app?
- how to deploy on push?
    - https://aws.amazon.com/blogs/containers/automatically-deploying-your-container-application-with-aws-copilot/
- how to monitor costs?
