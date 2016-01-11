---
layout: post
title: Configuration
---

Just about any code that is deployed or shared needs to be configured. This post
will discuss configuration best practices.

Most (all?) programs of any consequence will require configuration. Configuration can tell your program

* Where to find your database
* What port to bind to
* How long to wait until timing out
* Who to email when there's a problem
* What remote API server to connect to
* What domain name the web server is using
* Much more...

Early and Often
--------------------------------------------------------------------------------

In just about any project, configuration should be one of the first things you address. Inevitably, there will be values that need to be configured, even to share your code with other developers. If you don't address this early, you will attempt to solve problems with methods other than configuration that lead to sub-optimal functionality.

A classic example of this would be to leave configuration in the code (e.g. hardcode a database connection string). When this code is shared and the connection string must be modified to work in a new environment, there is a chance that the connection string change will be committed, and will break other's development. This database connection string is the first of many values that should be configured. If you wait to employ configuration, you will cost you and your team valuable time.

Configuration in Development
--------------------------------------------------------------------------------

Your codebase should come with sensible defaults. An engineer should be able to pull down the code, build it, and run it quickly with little to no configuration. In other words, the default configuration should always be set for a developer's environment. I recommend this for a few reasons

   * Your deployment process should take care of configuring your program in remote environments. In a developer's environment, there is no deployment (usually), and therefore, no chance to set configuration
   * Money. Engineering talent is expensive and we don't want to waste time with needless configuration
   * It would not be good for a program running in a dev environment program interfere with production data (e.g. a dev program connecting to a production database)

Engineers will need the ability to modify configuration in their development environments. The approach to modifying configuration in development will be the same for all other environments. These approaches will be discussed later.

First Things
--------------------------------------------------------------------------------
There are two main considerations when thinking about configuration:

* Where to store it
* What format

### Where to Store Configuration Data

#### Code Repository

Store configuration files in your code repository. You get all the benefits of
source control, and your configuration is easy to find, examine, and
change. When I do this, I usually make `/config/{qa,prod}` directories to hold my
configuration files.

The main drawback with this approach is that updating configuration values will require a full deployment. This can be mitigated by

* creating a patch deploy process that only deploys configuration
* using a hybrid approach that also takes advantage of host-based configuration (described below)

Finally, if your configuration varies per deployment/installation this can be difficult as you will have to have a separate file in source control for each deployment (not to mention how to determine which configuration file to use during deployment).

#### Deployment Hosts

The common approach here is to have a configuration file or directory that is searched at startup time e.g. `/etc/$APP_NAME` or `/opt/$APP_NAME/etc`. 

This is a great option if you have host/deployment-specific configuration e.g. your path
to ImageMagick's `convert` varies on a per-host basis. The drawback here is
that your configuration is now spread out over multiple hosts and is, therefore,
harder to maintain.

Managing these configuration files can be difficult, as they will need to be maintained by some dev-ops process. 

#### Database

Using a database for configuration is a great way to share configuration across all instances of the app. This is similar to the Code Repository option in that configuration is easily maintained across all deployments. However, deployment-specific configuration is difficult, and then there's the chicken/egg problem of how does the app know what database to connect to? (If configuration data is stored in a database, the app must connect to the database, but how can the app know what database to connect to without configuration.)

The big advantages with database configuration are

* configuration can vary independent of the software deployment
* configuration can be maintained from a central location

When to Apply Configuration
--------------------------------------------------------------------------------

### Build Time

In the technique, build artifacts are built with the configuration specified at build time. The main advantage with built time configuration is that the build artifacts can stand on their own i.e. they do not require any specific environment to be deployed into. The disadvantage is that any configuration change requires a new build and deployment.

### Runtime

### Hybrid (Build Time/Runtime)


Code in Configuration
--------------------------------------------------------------------------------

Storing code in configuration can give you programs a new level of power and flexibility. Of course, this comes at a cost. It's hard to verify the correctness of code in configuration, and an infrastructure must be built around the execution of this code. 

Example of storing code in configuration are email templates and event hooks. Email templates could be stored as configuration and allow for "easy" updates independent of deployments. Event hooks could allow for "easy" system updates that quickly address changing requirements. Code in configuration is also a solution to handling changing or unknown inputs into your program. E.g. if you're receiving data from a third party that is not well defined (the data, that is), you may want to store the code that processes the data in configuration so it can be updated without a new deployment.

You may notice, I put "easy" in quotes above. This is because, this kind of configuration is never easy and should be considered carefully before employing. I would advise against allowing anybody who is not a professional programmer write code in production systems. This includes any kind of template languages and markup.


Responsibilities of configuration:
--------------------------------------------------------------------------------
Clarity: Configured properties tell us something about the intent of the program. Values that are configurable are expected to be changed.
Safety: Updating text in a config file is safer than updating code.
Consolidated responsibility: Don't have to hunt through source code to update database name.

dev - development. This is usually run on a developers desktop or laptop. Source code is edited and tested in the environment.
qa - Artifacts are deployed to this environment to allow for access to people other than the developer.
test - Automated tests are run in this environment. This can be a continuous build environment.
prod - End-user facing.

Why?
--------------------------------------------------------------------------------
You can run multiple instances of your program on a single host.

A good codebase will have solutions for all kinds of configuration needs. Some things to consider:

* What should the default (no-config) value be?
Code should work out-of-the-box (e.g. straight from the source repository) for development. So the default configuration should always be set for a developer's laptop or desktop environment. After installing all required software, a developer should be able to build and run the software without any special configuration. I recommend this for a few reasons
   * Your deployment process should take care of configuring your program in remote environments. In a developer's environment (dev), there is no deployment (usually), and therefore, no chance to set configuration
   * Money. Engineering talent is expensive and we don't want to waste time with needless configuration
   * It would not be good for a program running in a dev environment program interfere with production data (e.g. a dev program connect to a prod database)

* How can a program be configured in development *and* in deployed environments?
If it is the case that in dev, there is no deployment, we still need a way to configure the program in dev. The approach I recommend here is
   * There is always an app home directory
   * This directory contains an `etc` directory that contains override configuration files
   * In remote (deployed) environments the app is typically deployed into the app home directory
   * In dev, The app home dir only contains the override config

* What is the presedent order for configuration sources?

* Can I apply configuration changes at run-time, or do I need to restart the process?
This is a choice that should be driven by business needs. How often is this needed? Can you get by with full-process restarts to apply new configuration? If you neeed more flexibility, does *all* configuration need to be mutable at runtime?

* What format do my configuration files use? How do I transform configuration data to program data.
   * Java Properties
   * JSON
   * YAML
   * XML
   * Scripting language
The first four (Java Properties, JSON, YAML, XML) are all in the same class. Java Properties are strictly name/value pairs. On the JVM, Java Properties are a great choice because of their easy serialization and deserialization using the `java.util.Properties` class. JSON is great because of it's simplicity and well-known spec. For more structured data, JSON has a big leg up over Java Properties. YAML is similar to JSON. I would not recommend XML as it's most likely overkill.

Using a scripting language for configuration give the added advantage of allowing for dynamic and calculated configuration values. I have never done this, and my guess is that there aren't many cases where this would be a good choice. One place where it would be a good choice would be where responsibilities are separated. Say dev-ops only has access to the local config, they could be given more flexibility by using a scripting language for configuration.

* How to give developers unique values for shared resources E.g. S3 buckets.
There are two general directions to go here:
   * Manual
   Require the developer to specify a unique value before startup. This unique value can be stored in the filesystem and read at startup.

   At Amazon we used our phone extension. Email addresses work well here too.

   * Automatic
   Same as manual, except the unique value is automatically generated. E.g. md5 checksum of the current timestamp. The main difference between the two options (besides manual requiring developer time/thought) is that debugging may be easier with manual as the unique value may be more recognizable.

* How can I be sure test/qa/prod configuration is correct?
* Dynamically driven config. E.g. `controller.model.__key = value`


* Variable expansion in configuration
This is a nice-to-have bordering on necessary. It can cut down on duplication and make configuration updates easier (because of the same principals of splitting large functions into smaller, reusable functions). An example (using Java Properties and Spring's `PropertyPlaceholderHelper`) is:

```
domain = example.com
email = info@${domain}
```

At runtime, email would be expanded to `info@example.com`.

Configuration should live in four different places:
1. Version controlled source in the codebase. E.g. `.properties` files

   I typically put these files in a `/src/main/resources` directory whos contents end up in the root of the constructed jar. Any configuration in these files should be suitable for development. E.g. database URLs should point to localhost, S3 buckets should be named in such a way to know that they're for development (and they should probably not conflict with other developers' buckets), etc..

1. Environment-specific version controlled source. E.g. overrides for test, qa, and prod environments

   These files live in directories like `/config/qa/etc`, `/config/prod/etc`, etc.. These files are deployed, depending on on the environment being deployed to (i.e. files under `/config/prod` are deployed when deploying to production). Some common fields that are overridden here are API tokens, database URLs, S3 buckets, email addresses, timeout values, and caching policies.

1. Local non-version controlled override files. These files will exist on development laptops and desktops, and qa/test/prod servers.

   This is usually reserved for host-specific configuration and quick-fix updates.

1. Command line options. E.g. `-Dxxx.home=/opt/xxx`

   One should always be able to override any configuration parameter via the command line at JVM start time.

TODO:
How to share config between shell and runtime.

tl;dr There are at least two environments to configure: `dev` and `prod`. Configuration should be applied in the following order: version controlled source, version controlled environment-specific source, host-specific, start-time (in order of least to most prescedence).
