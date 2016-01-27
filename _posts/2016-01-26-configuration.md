---
layout: post
title: Configuration, Part 2
---

In my [previous post](/2016/01/25/configuration/) I talked about general configuration best practices. In this post, I'll detail the specifics of how I've configured my latest project.

First, here are some relevant details about the project:

* It's a webapp [Anthology](https://anthology.co)
* It's also a REST API for our iOS app
* JVM/Scala
* Spring IoC

Processes are always kicked off from a shell script that lives in the `$APP_HOME/bin` directory. All of my scripts in the `$APP_HOME/bin` directory start with the following lines:

{% highlight bash linenos %}
#!/bin/bash
source $(dirname $0)/../.include
{% endhighlight %}

Line 1 tells the shell to use `/bin/bash` to execute the script. Line 2 sources `.include`. Here's `.include`:

{% highlight bash linenos %}
#!/bin/bash

cd $(dirname $0)/..
BASEDIR=$PWD
cd - >/dev/null

source $BASEDIR/.env
source $BASEDIR/.functions

version=$(cat $BASEDIR/.version)

if test -r $BASEDIR/.source-version-hash; then
  source_version_hash=$(cat $BASEDIR/.source-version-hash)
else
  source_version_hash=NONE
fi

if test -e $BASEDIR/etc/override.properties; then
  if test -n "$APP_HOME" -a "$APP_HOME" != "$BASEDIR"; then
    fatal "Your APP_HOME is misconfigured"
  fi
fi

if test -n "$APP_HOME" -a "$APP_HOME" != "$BASEDIR"; then
  if test -e $BASEDIR/etc/override.properties; then
    fatal "Your APP_HOME is misconfigured"
  fi
fi

if test -z "$APP_HOME"; then
  if test -e $BASEDIR/etc/override.properties; then
    info "Setting APP_HOME to $BASEDIR"
    export APP_HOME=$BASEDIR
  elif test -e $HOME/opt/$APP_NAME/etc/override.properties; then
    info "Setting APP_HOME to $HOME/opt/$APP_NAME"
    export APP_HOME=$HOME/opt/$APP_NAME
  else
    fatal "Please set APP_HOME"
  fi
fi

if test -r $APP_HOME/etc/env; then
  source $APP_HOME/etc/env
fi
{% endhighlight %}

This file's main purpose is to setup configuration. It checks that required variables are set (or that there is a sensible default) and it loads additional configuration.

Line 7 loads configuration from a `.env` file. This is default configuration (suitable for a development environment) for shell scripts. Configuration loaded in this step can be overridden by line 43 (e.g. to load production configuration values).

Lines 18-40 attempt to set and/or check the `APP_HOME` environment variable. Configuration override files are stored in the `$APP_HOME/etc` directory. This script has to deal with both development and deployed (e.g. qa or production) environments.

In development, the expectation is that there is an `APP_HOME` environment variable set to a directory that is not part of the codebase (this way, a developer's custom configuration will not be checked in). The default location for `APP_HOME` is `$HOME/opt/$APP_NAME`. So a developer can pull down the codebase and `mkdir -p $HOME/opt/$APP_NAME/etc` to get started. (I actually have a `/dev-bin/bootstrap` script that takes care of this.)

In a deployed environment, `$APP_HOME` and `$BASEDIR` must be equal. I typically deploy to `/opt/$APP_NAME`, so this means that there must be a `/opt/$APP_NAME/etc` directory.

After `.include` is sourced `APP_HOME` is set and we have verified that we have a configuration directory. We've also loaded environment specific shell variable overrides (line 43 in the file above).

Now we can start up our app. I won't show my complete startup script here (that can be the topic of another post) because the part that relates to configuration is actually pretty short.

{% highlight bash linenos %}
java -classpath $CP -Dapp.home=$APP_HOME co.anthology.common.spring.Main "$@"
{% endhighlight %}

You can ignore the `$CP` variable. The main thing to notice here is that `$APP_HOME` is passed to the JVM as a system property using a `-D` option.

The final part of configuring is handled by Spring. Here's a snippet from a Spring context file:

{% highlight xml linenos %}
<bean
    class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer"
    p:system-properties-mode-name="SYSTEM_PROPERTIES_MODE_OVERRIDE"
    p:ignore-unresolvable-placeholders="true"
    p:ignore-resource-not-found="true"
    >
  <property name="locations">
    <list>
      <value>classpath:common.properties</value>
      <value>file://${app.home}/etc/override.properties</value>
      <value>file://${app.home}/etc/local-override.properties</value>
    </list>
  </property>
</bean>
{% endhighlight %}

Configuration values are applied in these Spring context files like this

{% highlight xml linenos %}
<bean
    id="my-id"
    class="my.class.Name"
    p:property-1="${common.property-1}"
    />
{% endhighlight %}

In the above example, `property-1` will get the configured value of `common.property-1`. (Have a look at the Spring docs for more about this.)

The value of `common.property-1` is set in the `common.properties` file (line 9 above). It can be overridden by `override.properties` (line 10), `local-override.properties` (line 11), and finally by a system property (line 3).

`common.properties` is a source controlled default configuration file (all defaults should be dev friendly). It is deployed in a jar file.

`override.properties` is a deployed configuration file that is also under source control. There is one of these for each deployed environment (e.g. qa and prod). I typically store these files under `/config/{qa,prod}` in my codebase.

`local-override.properties` is an optional file (technically they're all optional because of the `ignore-resource-not-found="true"`) that lives on the deployment hosts. I don't use this, but if I ever needed a way to configure a host differently than the others, this is how I would do it.

Finally, I can set a system property at startup time e.g. `-Dcommon.property-1=xyz` to override all previous configuration values.

Okay, that's a short overview of how I'm configuring my latest project. Configuration may not be the most sexy topic, but it's definitely an important piece of any project.
