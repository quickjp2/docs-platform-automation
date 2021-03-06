---
title: Platform Automation for PCF
owner: PCF Platform Automation
---

!!! warning
    The Platform Automation for Pivotal Cloud Foundry (PCF)
    is currently in beta and is intended for evaluation and test purposes only.

Platform Automation for Pivotal Cloud Foundry (PCF) commands enable PCF operators
to script and automate Ops Manager actions.
Platform Automation for PCF is a set of tasks that wrap and extend [om][om],
a command-line interface to Ops Manager.

In this introduction we'll cover more about:

* What Platform Automation is
* How it compares with Ops Manager
* Show how to download and test the setup of Platform Automation  

## What is Platform Automation for PCF
Platform Automation for PCF commands enable PCF operators
to script and automate Ops Manager actions.

Platform Automation for PCF uses om,
(and by extension, the Ops Manager API)
to enable command-line interaction with Ops Manager
(a dashboard for installing, configuring, and updating software products on PCF).
Platform Automation for PCF comes bundled with Concourse tasks
that demonstrate how to use these tasks
in a containerized Continuous Integration (CI) system,
and a reference pipeline
showing one possible configuration of these tasks.

To learn more about Ops Manager,
see [Understanding the Ops Manager Interface][pivotalcf-understanding-opsman].

Platform Automation for PCF commands are:

* **Legible**: They use configuration from
human-readable YAML files,
which users can edit,
and manage in version control.

* **Modular**: Each command has defined inputs and outputs
and performs a granular action.
They're designed to work together
to enable many different workflows.
Users can build systems to make changes to all their products together,
or one at a time.
Users can extract configuration from one environment
and use it as a template for configuring many more.

* **Built for Automation** Commands are idempotent,
so re-running them in a CI won't break builds.
They read and write config from files,
which machines can easily pass around.
They're available in a Docker container,
which makes the tools easy to use in CI.

* **Not Comprehensive**: Workflows that use Platform Automation for PCF
typically also contain `om` commands, custom tasks,
and even interactions with the Ops Manager user interface.
Platform Automation for PCF is a set of tools to use alongside other tools,
rather than a comprehensive solution.

Platform Automation for PCF comes bundled with Concourse [tasks][concourse-task-definition].
These task files wrap one or two Platform Automation for PCF commands
and their input and output definitions
in the YAML format that Concourse uses to build pipelines.
If you use a different CI/CD platform, you can use these Concourse files as examples
of the inputs, outputs, and arguments used in each step in the workflow.

The [Task Reference][task-reference] topic discusses these example tasks further.

!!! Info
    If your current pipeline is based on PCF Pipelines,
    we recommend building a replacement pipeline with the new tooling,
    as opposed to trying to modify your existing pipeline to use the new tools.
    Since Platform Automation for PCF can easily take over management of an existing Ops Manager,
    this should be fairly straightforward.

## Ops Manager vs Platform Automation for PCF

Ops Manager manages PCF operations manually. Platform Automation for PCF lets you automate the process.

**Ops Manager**

Ops Manager lets you install and configure PCF and PCF products manually.
Its UI takes you through the configuration decisions or changes that you need to make in situations such as:

* Installing PCF for the first time.

* Changing product configurations as a result of human decisions.

* Running major or minor version upgrades of PCF or PCF products.

**Platform Automation for PCF**

You can run Platform Automation for PCF manually from a command line.
But you can also embed them in scripts and pipelines
to perform Ops Manager functions when human attention is not needed, such as:

* Upgrading products to new patch versions.
  - Configuration settings should not differ between successive patch versions within the same minor version line.
    Underlying properties or property names may change,
    but the tile's upgrade process automatically translates properties to the new fields and values.
  - Pivotal cannot guarantee the functionality of upgrade scripts in third-party PCF product tiles.

* Replicating configuration settings from one product tile to the same product tile on a different foundation.
- Because properties and property names can change between patch versions of a product,
  you can only safely apply configuration settings across product tiles if their versions exactly match.

**Ops Comparison**

The following table compares how Ops Manager
and Platform Automation for PCF might run a typical sequence of PCF operations:

<table border="1">
  <tr>
    <th></th>
    <th>Ops Manager</th>
    <th>Platform Automation for PCF</th>
  </tr><tr>
    <th>When to Use</th>
    <th>First install and minor upgrades</th>
    <th>Config changes and patch upgrades</th>
  </tr><tr>
    <th>1. Create Ops Manager VM</th>
    <td>Manually prepare IaaS and create Ops Manager VM</td>
    <td><code>create-vm</code></td>
  </tr><tr>
    <th>2. Configure Who Can Run Ops</th>
    <td>Manually configure internal UAA or external identity provider</td>
    <td><code>configure-authentication</code> or <code>configure-saml-authentication</code></td>
  </tr><tr>
    <th>3. Configure BOSH</th>
    <td>Manually configure BOSH Director</td>
    <td><code>configure-director</code> with settings saved from BOSH Director with same version</td>
  </tr><tr>
    <th>4. Add Products</th>
    <td>Click <strong>Import a Product</strong> to upload file, then <strong>+</strong> to add tile to Installation Dashboard</td>
    <td><code>upload-and-stage-product</code></td>
  </tr><tr>
    <th>5. Configure Products</th>
    <td>Manually configure product tiles</td>
    <td><code>configure-product</code> with settings saved from tiles with same version</td>
  </tr><tr>
    <th>6. Deploy Products</th>
    <td>Click <strong>Apply Changes</strong></td>
    <td><code>apply-changes</code></td>
  </tr><tr>
    <th>7. Upgrade</th>
    <td>Manually export existing Ops Manager settings, power off the VM, then create a new, updated
    Ops Manager VM</td>
    <td><code>export-installation</code> then <code>upgrade-opsman</code></td>
  </tr>
</table>


## Downloading and Testing Platform Automation

The following describes the procedure for downloading, installing and testing the setup of Platform Automation.

### Prerequisites

You'll need the following in order to setup Platform Automation.

* Deployed Concourse

!!! note
    Platform Automation for PCF is based on Concourse CI.
    We recommend that you have some familiarity with Concourse before getting started.
    If you are new to Concourse, [Concourse CI Tutorials](https://docs.pivotal.io/p-concourse/guides.html) would be a good place to start.

* Persisted datastore that can be accessed by Concourse resource (e.g. s3, gcs, minio)
* Pivnet access to [Platform Automation][pivnet-platform-automation]

### Download Platform Automation

1. Download the latest [Platform Automation][pivnet-platform-automation] from Pivnet.
   This includes:
    * `Concourse Tasks`
    * `Docker Image for Concourse Tasks`

!!! note
    If the Pivnet link does not work for you,
    you might not have access to the product.
    Please communicate this in the #pcf-automation slack channel
    until the project is GA.

2. Store the `platform-automation-image-*.tgz`
   in a blobstore that can be accessed via a Concourse pipeline.

3. Store the `platform-automation-tasks-*.zip`
   in a blobstore that can be accessed via a Concourse pipeline.

### Testing Platform Automation Setup

Next we'll create a test pipeline to see if the assets can be accessed correctly.
   This pipeline runs a test task, which ensures that all the parts work correctly.

!!! note
       The pipeline can use any blobstore.
       We choose S3 because the resource natively supported by Concourse.
       The S3 Concourse resource also supports S3-compatible blobstores (e.g. minio).
       See [S3 Resource](https://github.com/concourse/s3-resource#source-configuration) for more information.
       If you want to use other blobstore, you need to provide a custom [resource type](https://concourse-ci.org/resource-types.html).

 In order to test the setup, fill in the S3 resource credentials and set the below pipeline on your Concourse instance.

```yaml
resources:
- name: platform-automation-tasks-s3
  type: s3
  source:
    access_key_id: ((access_key_id))
    secret_access_key: ((secret_access_key))
    region_name: ((region))
    bucket: ((bucket))
    regexp: platform-automation-tasks-(.*).zip

- name: platform-automation-image-s3
  type: s3
  source:
    access_key_id: ((access_key_id))
    secret_access_key: ((secret_access_key))
    region_name: ((region))
    bucket: ((bucket))
    regexp: platform-automation-image-(.*).tgz

jobs:
- name: test-resources
  plan:
  - aggregate:
    - get: platform-automation-tasks-s3
      params:
        unpack: true
    - get: platform-automation-image-s3
      params:
        unpack: true
  - task: test-resources
    image: platform-automation-image-s3
    file: platform-automation-tasks-s3/tasks/test.yml
```

## Executing Commands Locally with Docker
If you wish to use the underlying `om` and `p-automator` CLI tools from your local workstation,
we recommend using docker to execute commands.

With `p-automator` in particular, using Docker is necessary,
as the IaaS CLIs upon which we depend can be tricky to install.
With `om` it's more a matter of convenience -
you can just as easily download the binary if it's available for your system.

To execute commands in docker, first import the image:

```bash
docker import ${PLATFORM_AUTOMATION_IMAGE_TGZ} pcf-automation-image
```

Where `${PLATFORM_AUTOMATION_IMAGE_TGZ}` is the image file downloaded from Pivnet.

Then, you can use `docker run` to pass it arbitrary commands.
Here, we're running the `p-automator` CLI to see what commands are available:

```bash
docker run -it --rm -v $PWD:/workspace -w /workspace pcf-automation-image \
p-automator -h
```

Note that this will have access read and write files in your current working directory.
If you need to mount other directories as well, you can add additional `-v` arguments.

There are a couple of `om` commands useful enough to document here for convenience.
It can be very useful to pull configuration for the director and tiles locally.
Run the following command to get the staged config for a product:

```bash
docker run -it --rm -v $PWD:/workspace -w /workspace pcf-automation-image \
om --env ${ENV_FILE} staged-config --product-name ${PRODUCT_SLUG} --include-placeholders
```

`${ENV_FILE}` is the [env file] required for all tasks.
`${PRODUCT_SLUG}` is the name of the product downloaded from [pivnet].
The resulting file can then be parameterized, saved, and committed to a config repo.

Here's the command to get the director configuration:

```bash
docker run -it --rm -v $PWD:/workspace -w /workspace pcf-automation-image \
om --env ${ENV_FILE} staged-director-config --include-placeholders
```

{% include ".internal_link_url.md" %}
{% include ".external_link_url.md" %}
