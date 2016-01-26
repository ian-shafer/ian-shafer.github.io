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

```
#!/bin/bash
source $(dirname $0)/../.include
```

The first line tells the shell to use `/bin/bash` to execute the script. The second line sources a `.include` in `/bin`'s parent directory. Here's `.include`:

```
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
    warn "This script is running from: $BASEDIR"
    warn "Your APP_HOME is set to: $APP_HOME"
    fatal "Your APP_HOME is probably misconfigured"
  fi
fi

if test -n "$APP_HOME" -a "$APP_HOME" != "$BASEDIR"; then
  if test -e $BASEDIR/etc/override.properties; then
    warn "This script is running from: $BASEDIR"
    warn "Your APP_HOME is set to: $APP_HOME"
    fatal "Your APP_HOME is probably misconfigured"
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

if test -r $APP_HOME/etc/env.sh; then
  source $APP_HOME/etc/env.sh
fi
```
