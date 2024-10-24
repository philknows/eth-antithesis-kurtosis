# Using Antithesis with Kurtosis


Antithesis is designed to supplement or replace your existing testing tools and live alongside your normal CI workflow. 
**NOTE**: *Because you’ll be using Antithesis in combination with Kurtosis, the setup and test kickoff steps below are slightly different from what you see in our online documentation. We go into detail about the key differences on the next page.*

To test with Antithesis, you’ll first have to: 
1. [Package your software using containers](https://antithesis.com/docs/getting_started/setup.html) and deploy it to the Antithesis Environment, an entirely self-contained environment (like a small private cloud) in which we run our tests.
2. Prepare your Kurtosis Configuration

Once you have the setup and preparation steps completed, you will be able to kick off your first test!

Every time you run a test, Antithesis generates a [triage report](https://antithesis.com/docs/reports/triage.html), with information about the high-level properties of your software and basic debugging information for suspected violations of those properties. As its name suggests, the triage report helps you decide whether to conduct a deeper investigation. 
If you need to dive deeper into a specific issue, Antithesis can generate a [bug report](https://antithesis.com/docs/reports/bug.html). From a moment in the triage reports, users can [initiate a debugging session](https://antithesis.com/docs/multiverse_debugging/overview.html), using the Antithesis Notebook to navigate and explore each bug.  


## Your First Test
This repo contains sample code for building images and running your first test.

We break the process down in greater detail below. The sample code, in addition to the default [test properties](https://antithesis.com/docs/using_antithesis/properties.html) in Antithesis, contains a few properties specific to your use case, and you can strengthen your testing by adding more properties to your code using [Antithesis’ SDKs](https://antithesis.com/docs/using_antithesis/sdk/overview.html). You can also join us on [Discord](https://discord.com/channels/981263820997673040/981263820997673045) for help.

## Setup
[Configuration image](https://antithesis.com/docs/getting_started/setup.html#create-a-configuration-directory)
You’ll need a docker image containing the Ethereum Kurtosis package with all dependencies, all the way down the stack. 

This is slightly different from the normal Antithesis workflow you’re used to in that the dependencies required are not running services as we describe in the quickstart guide. With Kurtosis, the required dependencies are just Kurtosis packages. 

Because these are Kurtosis packages, not services, they should all go in the same image as the Ethereum package, not separate containers as described in the Antithesis quickstart guide. 

Kurtosis packages we know you’ll need include:
- postgres
- prometheus
- db-adminer
- redis

### Mapping dependencies

To enable easier integration between Kurtosis and Antithesis, Kurtosis has built a feature that will enable the system to determine its entire dependency graph at interpretation time and allow users to pull all dependencies at once. This will go live in the near future (once [this PR merges](https://github.com/kurtosis-tech/kurtosis/pull/2518)), and Kurtosis will announce its availability.

Prior to this feature going live, you can map the dependency graph and pull dependencies as follows:
1. Run Ethereum package locally without network access. It should fail due to missing external dependencies. 
2. Pull and add any necessary Kurtosis packages to your config image. 
3. Repeat to retrieve the next layer of dependencies until the Ethereum package runs.  

### Client team images 

We recommend using “Minimal” client images.

You’ll also need an Assertoor image built with any test files you want to run. The Assertoor image contains some default tests but you can add others if desired.
- Each test in the Assertoor image needs to be specified in the configuration yaml file in the Ethereum package 
- Test files can contain instructions for starting themselves, OR they can be invoked by adding a script to the Assertoor image
  - We recommend these invocation scripts be named as follows, as this will enable future compatibility with the Antithesis test composer if desired: singleton_driver_<test_script_name>

## Starting a test

To start a test, simply call the relevant endpoint. 

```
curl --fail -u 'user:password' -X POST https://ethereum.antithesis.com/api/v1/launch_experiment/ethereum -d '{"params": {"custom.duration":"0.5", "antithesis.images":"docker.io/ethpandaops/geth:v1.13.13;docker.io/chainsafe/lodestar:v1.22.0;docker.io/ethpandaops/assertoor:latest"}}' 
```

(For the username:password please reach out to “luis.marcano@antithesis.com” or send us a message in the [Discord channel](https://discord.com/channels/981263820997673040/981263820997673045))

**Endpoint name**: ethereum

**Custom parameters**: (if not specified uses the default)
- Custom.configuration
  - **Default**: ‘.github/tests/minimal.yaml’
- Custom.duration (exploration in hours)(min 0.5 hours)
  - **Default**: 1

**Antithesis Parameters**:
- [Documentation](https://antithesis.com/docs/using_antithesis/webhook_reference.html#webhook-parameters)
- Required:
  - antithesis.report.recipients
  - antithesis.images
  - antithesis.config_image

## Results and debugging

Test results and reports will be exactly as described in our [online documentation](https://antithesis.com/docs/reports/triage.html).

Debugging using Antithesis’ multiverse debugger will also be exactly as [documented online](https://antithesis.com/docs/multiverse_debugging/overview.html).

