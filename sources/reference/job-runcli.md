page_main_title: runCLI
main_section: Reference
sub_section: Jobs

# runCLI job

runCLI jobs combine the unlimited flexibility of runSh jobs with the added
convenience of automatically configured environments and tools. In addition to
running custom scripts as described in runSh, you can also add
[cliConfig](resource-cliconfig.md) resources as inputs to this job. The relevant
CLI tools will be preconfigured for your scripts to use. runCLI jobs also give
you access to the `shippable_replace` tool to help with replacing placeholders
in files with values from the environment.

## Basic example

````
jobs:
  - name: myRunCLIJob
    type: runCLI
    steps:
      - IN: awsCLIConfig
      - TASK:
        - script: aws s3 ls
````

The snippet above shows a basic example with required tags for configuring a runCLI job.

- [name](job-runcli.md#name): The name of your job. Set it to something that
describes what the job does and is easy to remember. This will be the display
name of the job in your pipeline visualization.

- [type](job-runcli.md#type): The type of your job. This must be `runCLI`.

- [steps](job-runcli.md#steps): The steps that should be executed in your job.
Steps can have any number of `IN` and `OUT` resources. `IN` resources that are
[cliConfig](resource-cliconfig.md) will configure the appropriate CLI tools
before your job starts running. You can have a single `TASK` property in your
steps to specify your script commands. Everything under the `steps` section
executes sequentially.

## Complete reference

````
jobs:
  - name: <name>                                
    type: runCLI                                
    on_start:                                   #optional
      - script: echo 'This block executes when the TASK starts'
      - NOTIFY: slackNotification
    steps:                                      
      - IN: <resource>                          
      - IN: <resource>                          #optional
      - TASK:                                   
        - script: <command>                     
        - script: <command>                     #optional
      - OUT: <resource>
      - OUT: <resource>
    on_success:                                 #optional
      - script: echo 'This block executes after the TASK section executes successfully'
      - NOTIFY: slackNotification
    on_failure:                                 #optional
      - script: echo 'This block executes if the TASK section fails'
      - NOTIFY: slackNotification
    on_cancel:                                  #optional
      - NOTIFY: slackNotification
    always:                                     #optional
      - script: echo 'This block executes if the TASK section succeeds or fails'
      - NOTIFY: slackNotification
````

In addition to the required parameters described in the **Basic reference** section above, you can also
configure your runCLI job with the parameters described below:

  - [on_start](job-runcli.md#on_start): Specify `script` or `NOTIFY` sections
that are executed when the job starts.

  - [on_success](job-runcli.md#on_success): Specify `script` or `NOTIFY`
sections that are executed only if the `TASK` section succeeds.

  - [on_failure](job-runcli.md#on_failure): Specify `script` or `NOTIFY`
sections that are executed only if the `TASK` section fails.

  - [on_cancel](job-runcli.md#on_cancel): Specify `NOTIFY` sections to send
notifications if the job is canceled.  This cannot run scripts.

  - [always](job-runcli.md#always): Specify `script` or `NOTIFY` sections that
are always executed at the end of the job, regardless of whether the `TASK`
section failed or succeeded.  Scripts will not run if the job is canceled,
but notifications will be sent.

## Configured CLI tools

The runCLI job uses the subscription integration specified in the
[cliConfig](resource-cliconfig.md) to determine which CLI tools to configure.
These tools are configured with the credentials contained in the subscription
integration. Here is a list of the tools configured for each integration type:

| Integration Type                    | Configured Tools           |
| ------------------------------------|-------------|
| AWS                                 | [AWS CLI](https://aws.amazon.com/cli/); [EB (Elastic Beanstalk) CLI](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3.html) |
| Amazon EC2 Container Registry (ECR) | [Docker Engine](https://docs.docker.com/engine/reference/commandline/docker/) |
| Docker Hub                          | [Docker Engine](https://docs.docker.com/engine/reference/commandline/docker/) |
| Docker Trusted Registry             | [Docker Engine](https://docs.docker.com/engine/reference/commandline/docker/) |
| Google Container Engine             | [gcloud](https://cloud.google.com/sdk/gcloud/); [kubectl](https://kubernetes.io/docs/user-guide/kubectl/) |
| Google Container Registry (GCR)     | [Docker Engine](https://docs.docker.com/engine/reference/commandline/docker/) |
| JFrog Artifactory                   | [JFrog CLI](https://www.jfrog.com/confluence/display/CLI/CLI+for+JFrog+Artifactory) |
| Kubernetes                          | [kubectl](https://kubernetes.io/docs/user-guide/kubectl/) |
| Private Docker Registry             | [Docker Engine](https://docs.docker.com/engine/reference/commandline/docker/) |
| Quay.io                             | [Docker Engine](https://docs.docker.com/engine/reference/commandline/docker/) |


##Environment variables

In order to make it easier to write your scripts and work with `IN` and `OUT` resources, we have made several environment variables available for use within your `TASK` section of your `runCLI` job.

A complete list of these variables is available in the [Environment variables for unmanaged jobs docs](/reference/jobs-unmanaged/), along with simple tutorials showing how you can work with `IN` and `OUT` resources in your scripts.  

## Status notifications for Pull Requests

If you have a [`gitRepo`](/reference/resource-gitrepo/) as IN to your `runCLI` job, you can see the job status for Pull Requests to your git repository .

You can turn this ON or OFF using the `showBuildStatus` tag in the `gitRepo` resource.

For example, the runCLI job will be configured in `shippable.jobs.yml` as shown below for this scenario:

```
jobs:
  - name: <name>                                
    type: runCLI                               
    steps:                                      
      - IN: myGitRepo                           
        showBuildStatus: true                   #set to true if you want job status to be shown in your SCM UI
      - TASK:
        - script: <command>
        - script: <command>

```

The corresponding  `gitRepo` resource in `shippable.resources.yml` needs to include the tag `buildOnPullRequest: true` as shown below:

```
resources:
  - name: myGitRepo                             
    type: gitRepo                               
    integration: <integrationName>             
    pointer:
      sourceName: <repoName>                    
      branch: <branchName>                      
      buildOnPullRequest: true              #specify true to trigger the job for pull requests to the git repository

```

* If the **runCLI** job is triggered as a result of the a Pull Request for the configured repo (`repoName`) and (`branchName`), the build status message be displayed in the UI of your source control provider.

* When the **runCLI** job is in the processing state the GitHub UI will look like
<img src="../../images/reference/jobs/runCLI/processingBuildStatus.png" alt="Build Status Processing" style="width:800px;vertical-align: middle;display: block;margin-right: auto;"/>

* When the **runCLI** job successfully completes, the GitHub UI will look like
<img src="../../images/reference/jobs/runCLI/successBuildStatus.png" alt="Build Status Success" style="width:800px;vertical-align: middle;display: block;margin-right: auto;"/>

* When the **runCLI** job is cancelled or failed the GitHub UI will look like
<img src="../../images/reference/jobs/runCLI/failedBuildStatus.png" alt="Build Status Failed" style="width:800px;vertical-align: middle;display: block;margin-right: auto;"/>

## shippable_replace

When you create a `runCLI` job, the `shippable_replace` command-line tool is
automatically installed. `shippable_replace` looks for placeholders in one or
more files and replaces them with the values of the corresponding environment
variables.

`shippable_replace` is used as follows:

```
shippable_replace [paths to files]
```

The files passed to `shippable_replace` may contain one or more placeholders
for environment variables. Placeholders are strings of the form
`$ENVIRONMENT_VARIABLE_NAME` or `${ENVIRONMENT_VARIABLE_NAME}`. We recommend
`${ENVIRONMENT_VARIABLE_NAME}`, as it is less likely to cause errors if you have
an environment variable name that is a substring of another environment
variable's name.

For example, say you wanted to insert the current job name into two different
files: `file1.json` and `test/file2.txt`. The name of the environment variable
containing the job name is `JOB_NAME`. (See the [runSh
documentation](job-runsh.md) for the list of available environment
variables.)

`file1.json` might look like this:
```
{
  "foo": "bar",
  "jobName": "${JOB_NAME}"
}
```

`test/file2.txt` contains this:

```
This job, ${JOB_NAME}, is the coolest job ever.
```

`shippable_replace` is then added to the task section of the `runCLI` job:

```
jobs:
  - name: myRunCLIJob
    type: runCLI
    steps:
      - IN: <resource>
      - TASK:
        - script: shippable_replace file1.json test/file2.txt
```

After `shippable_replace`, `file1.json` now looks like this:

```
{
  "foo": "bar",
  "jobName": "myRunCLIJob",
  "fizz": "buzz"
}
```

And `test/file2.txt` looks like this:

```
This job, myRunCLIJob, is the coolest job ever.
```
