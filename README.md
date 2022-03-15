# Issues 

When building this project, I ran into several issues:
1. In the buildspec.yml file, line 20 (poetry run python generator.py validate) resulted in the following error: ImportError: cannot import name 'soft_unicode' from 'markupsafe' (/root/.cache/pypoetry/virtualenvs/generator-5SpHchWn-py3.7/lib/python3.7/site-packages/markupsafe/__init__.py)
    a. Fix to the above issue was a virtual Python environment with dependencies installed within
2. Next issue was CloudFormation permissions with the error: User: arn:aws:sts::1234567890:assumed-role/codebuildrole/AWSCodeBuild-27721881874ETC is not authorized to perform: cloudformation:CreateChangeSet on resource: arn:aws:cloudformation:us-east-1:1234567890:stack/leonidas-dev/* because no identity-based policy allows the cloudformation:CreateChangeSet action
    a. Attempts to attach inline policies specifically for cloudformation:CreateChangeSet didn't work.  Temporary workaround is provide full admin access to CloudFormation, CodeBuild, and CodeCommit.
3. Final issue was a result of the command run on line 29 of buildspec.yml (serverless deploy --conceal), result was: [Container] 2022/03/15 16:02:05 Phase context status code: COMMAND_EXECUTION_ERROR Message: Error while executing command: serverless deploy --conceal. Reason: exit status 1
    a. This appears to be a result of a lack of serverless.yml file.  In the /generator/templates/ directory there is a serverless.jinja2 file.  I simply renamed that file  serverless.yml and placed a copy into the /templates/ and /generator/ directories.

**After doing all these, the build was successful.  More updates will come with additional troubleshooting.**

# Leonidas

This is the repository containing Leonidas, a framework for executing attacker actions in the cloud. It provides a YAML-based format for defining cloud attacker tactics, techniques and procedures (TTPs) and their associated detection properties. These definitions can then be compiled into:

* A web API exposing each test case as an individual endpoint
* Sigma rules (https://github.com/Neo23x0/sigma) for detection
* Documentation  - see http://detectioninthe.cloud/ for an example

![Leonidas Architecture](./docs/architecture.png?raw=true "Leonidas Architecture")

## Deploying the API

The API is deployed via an AWS-native CI/CD pipeline. Instructions for this can be found at [Deploying Leonidas](./docs/deploying-leonidas.md).

## Using the API

The API is invoked via web requests secured by an API key. Details on using the API can be found at [Using Leonidas](./docs/using-leonidas.md)

## Installing the Generator Locally

To build documentation or Sigma rules, you'll need to install the generator locally. You can do this by:

* `cd generator`
* `poetry install`

## Generating Sigma Rules

Sigma rules can be generated as follows:

* `poetry run ./generator.py sigma`

The rules will then appear in `./output/sigma`

## Generating Documentation

The documentation is generated as follows:

* `poetry run ./generator.py docs`

This will produce markdown versions of the documentation available at `output/docs`. This can be uploaded to an existing markdown-based documentation system, or the following can be used to create a prettified HTML version of the docs:

* `cd output`
* `mkdocs build`

This will create a `output/site` folder containing the HTML site. It is also possible to view this locally by running `mkdocs serve` in the same folder.

## Writing Definitions

The definitions are written in a YAML-based format, for which an example is provided below. Documentation on how to write these can be found in [Writing Definitions](./docs/writing-definitions.md)

```yaml
---
name: Enumerate Cloudtrails for a Given Region
author: Nick Jones
description: |
  An adversary may attempt to enumerate the configured trails, to identify what actions will be logged and where they will be logged to. In AWS, this may start with a single call to enumerate the trails applicable to the default region.
category: Discovery
mitre_ids:
  - T1526
platform: aws
permissions:
  - cloudtrail:DescribeTrails
input_arguments:
executors:
  sh:
    code: |
      aws cloudtrail describe-trails
  leonidas_aws:
    implemented: True
    clients:
      - cloudtrail
    code: |
      result = clients["cloudtrail"].describe_trails()
detection:
  sigma_id: 48653a63-085a-4a3b-88be-9680e9adb449
  status: experimental
  level: low
  sources:
    - name: "cloudtrail"
      attributes:
        eventName: "DescribeTrails"
        eventSource: "*.cloudtrail.amazonaws.com"
```

## Credits

Project built and maintained by Nick Jones ( [NJonesUK](https://github.com/NJonesUK) / [@nojonesuk](https://twitter.com/nojonesuk)).

This project drew ideas and inspiration from a range of sources, including:

* [Pacu](https://github.com/RhinoSecurityLabs/pacu)
* [Rhino Security's AWS IAM Privilege Escalations](https://github.com/RhinoSecurityLabs/AWS-IAM-Privilege-Escalation)
* All of Scott Piper's AWS security work ( [https://github.com/0xdabbad00](https://github.com/0xdabbad00) / [@0xdabbad00](https://twitter.com/0xdabbad00) )
* [MITRE ATT&CK](https://attack.mitre.org/matrices/enterprise/)
* [MITRE CALDERA](https://github.com/mitre/caldera)
* [Red Canary's Atomic Red Team](https://github.com/redcanaryco/atomic-red-team)
* [Sigma](https://github.com/Neo23x0/sigma)
