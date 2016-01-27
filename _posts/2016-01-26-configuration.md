---
layout: post
title: Configuration (Continued)
---

WARNING: This post is a WORK IN PROGRESS.

In my [previous post](/2016/01/25/configuration/) I talked about general configuration best practices. In this post, I'll detail the specifics of how I've configured my latest project.

First, here are some relevant details about the project:

* It's a webapp (https://anthology.co)
* It's also a REST API for our iOS app
* JVM/Scala
* Spring IoC

Configuration is always kicked off from a shell script that lives in my `/bin` directory. All of my scripts in the `/bin` directory start with the following lines:

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

This file's main purpose is to deal with configuration. It checks that required variables are set (or that there is a sensible default) and it loads additional configuration.

Line 7 loads configuration from a `.env` file. This is default configuration (suitable for a development environment) for shell scripts. Configuration loaded in this step can be overridden by line 43 (e.g. to load production configuration values).

Lines 18-40 attempt to set and/or check the `APP_HOME` environment variable. Configuration override files are stored in the `$APP_HOME/etc` directory. This script has to deal with both development and deployed (e.g. qa or production) environments.

In development, the expectation is that there is an `APP_HOME` environment variable set to a directory that is not part of the codebase (this way, a developer's custom configuration will not be checked in). The default location for `APP_HOME` is `$HOME/opt/$APP_NAME`. So a developer can pull down the codebase and `mkdir -p $HOME/opt/$APP_NAME/etc` to get started. (I actually have a `/dev-bin/bootstrap` script that takes care of this.)

In a deployed environment, `$APP_HOME` and `$BASEDIR` must be equal. I typically deploy to `/opt/$APP_NAME`, so this means that there must be a `/opt/$APP_NAME/etc` directory.

At this point, `APP_HOME` is set and we have verified that we have a configuration directory. We've also loaded environment specific shell variable overrides (line 43 in the file above).

Now we can startup up our app. I won't show my complete startup script here (that can be the topic of another post) because the part that relates to configuration is actually pretty short.
