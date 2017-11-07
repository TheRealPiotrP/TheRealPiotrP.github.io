---
layout: post
title: Installing dotnet CLI on Travis CI via apt-get
---

### TL;DR
We can use Travis CI's [APT addon](https://docs.travis-ci.com/user/installing-dependencies/#Installing-Packages-with-the-APT-Addon) to install the `dotnet CLI` for use during a build.

### .travis.yml
```
...
addons:
  apt:
    sources:
      - sourceline: 'deb https://packages.microsoft.com/repos/microsoft-ubuntu-trusty-prod trusty main'
        key_url: 'https://packages.microsoft.com/keys/microsoft.asc'
    packages:
      - dotnet-sdk-2.0.2
...
```

This `.travis.yml` file will install .NET CLI on Ubuntu Trusty in Travis CI:
![dotnet CLI installing in Travis CI](https://github.com/TheRealPiotrP/TheRealPiotrP.github.io/blob/master/images/Screen%20Shot%202017-11-06%20at%2011.55.35%20PM.png)

You can modify the example to support other distros as well as other CLI versions.

#### sources
The .NET team publishes debian packages for the CLI to feeds under https://packages.microsoft.com. The feeds are distro-specific so you want to pick the right feed. You can find a full listing [here](https://www.microsoft.com/net/learn/get-started/linuxubuntu), just pick the right feed for your distro. Once you have it you can set:
 - sourceline: the feed URL you chose above. I had to remove the `[arch=amd64]`
 - key_url: 'https://packages.microsoft.com/keys/microsoft.asc'. This one seems consistent for all of the feeds.

#### packages
Every release of the .NET CLI is published to the package feeds. You can find a listing of released versions [here].
