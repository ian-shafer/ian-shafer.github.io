---
layout: post
title: Configuration
---

Just about any code that is deployed or shared needs to be configured. This post
will discuss configuration best practices.

All programs of any consequence will require configuration. Configuration can tell your program

* Where to find your database
* What port to bind to
* How long to wait until timing out
* Who to email when there's a problem
* What remote API server to connect to
* What domain name the web server is using
* Much more...

Early and Often
--------------------------------------------------------------------------------

In just about any project, configuration should be one of the first things you address. If you don't address this early, you will attempt to solve problems with methods other than configuration that will lead to sub-optimal functionality and code smell.

A classic consequence of putting off configuration is leaving configuration in the code. E.g. hardcoding a database connection string. When this code is shared and the connection string must be modified to work in a new environment, there is a chance that the connection string change will be committed, and will break other's development environments. This database connection string is the first of many values that should be configured. If you wait to employ configuration, you will cost you and your team valuable time.

Another cost of not setting up configuration early is that you will be missing a valuable tool. Configuration can be used to solve many important problems (e.g. deploying to QA), if this tool is missing you will have a hard time solving certain types of problems.

Configuration in Development
--------------------------------------------------------------------------------

Your codebase should come with sensible defaults. An engineer should be able to pull down the code, build it, and run it quickly with little to no custom configuration. In other words, the default configuration should always be set for a developer's environment. I recommend this for a few reasons

   * Your deployment process should take care of configuring your program in remote environments. In a developer's environment, there is no deployment, and therefore, no chance to set configuration
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

Finally, if your configuration varies per deployment/installation this can be difficult as you will have a separate file in source control for each deployment (not to mention how to determine which configuration file to use during deployment).

#### Deployment Hosts

The approach here is to have a configuration file or directory that is searched at startup time e.g. `/etc/$APP_NAME` or `/opt/$APP_NAME/etc`. 

This is a great option if you have host/deployment-specific configuration e.g. your path
to ImageMagick's `convert` executable varies on a per-host basis. The drawback here is
that your configuration is now spread out over multiple hosts and is, therefore,
harder to maintain.

Managing these configuration files can be difficult, as they will need to be maintained by some dev-ops process. 

#### Database

Using a database for configuration is a great way to share configuration across all instances of the app. This is similar to the Code Repository option in that configuration is easily maintained across all deployments. However, there's the chicken/egg problem of how does the app know what database to connect to? And there is no version control like the Code Repository solution gives.

The big advantages with database configuration are

* configuration can vary independent of the software deployment
* configuration can be maintained from a central location

When to Apply Configuration
--------------------------------------------------------------------------------

### Build Time

In the technique, build artifacts are built with the configuration specified at build time. The main advantage with built time configuration is that the build artifacts can stand on their own i.e. they do not require any specific environment to be deployed into. The disadvantage is that any configuration change requires a new build and deployment.

Built Time configuration application all but forces one to store configuration along with the code.

### Runtime

Using the runtime approach, all configuration is loaded at runtime. Configuration is not part of the deployed artifacts, but lives on the runtime hosts or a database.

Configurations updates can be applied easily by restarting the process (or one could even write some code to dynamically reload the configuration without a restart).

### Hybrid (Build Time/Runtime)

This is a mix of the two. The advantage of this is that you can choose what values are configured in what way. The disadvantage is that you have to maintain multiple configuration systems.

Code in Configuration
--------------------------------------------------------------------------------

Storing code in configuration can give you programs a new level of power and flexibility. This technique only makes sense if you are not storing configuration in your code repository. This technique comes at a cost

* It's hard to verify the correctness of code in configuration
* An infrastructure must be built around the execution of the configured code
* Debugging can be difficult as it can be difficult to know what deployed code executed with what configured code

Examples of storing code in configuration are email templates and event hooks. Email templates can be stored as configuration and allow for updates independent of deployments. Event hooks could allow for system updates that quickly address changing requirements. Code in configuration is also a solution to handling changing or unknown inputs into your program. E.g. if you're receiving data from a third party that is not well defined (the data, that is), you may want to store the code that processes the data in configuration so it can be updated without a new deployment.

This kind of configuration is never easy and should be considered carefully before employing. I would not use this unless there was a very good reason.

Sometimes, there can be external factors that might lead to using this technique

* Your deployment process is slow and you need to see changes fast
* System requirements are not well defined or are constantly changing


Command Line Overrides
--------------------------------------------------------------------------------

The ability to override configuration from the command line can be valuable as a way to quickly change configuration in both production and development environments. Configuration values should be applied in the following order

1. Hard-coded values in code
1. Config files/database values
1. Command line options


Next
--------------------------------------------------------------------------------

In my next post, I'll show the details of how I've configured my latest application.
