---
title: Debian Package Management
---

# Debian Package Management
{: .no_toc}

## Table of Contents
{: .no_toc .text-delta}

* TOC
{:toc}

## Import a Key

```sh
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys <PUBKEY>
# or
$ sudo gpg --keyserver pgpkeys.mit.edu --recv-key  <PUBKEY>

$ sudo gpg -a --export <PUBKEY> | sudo apt-key add -
$ sudo apt-get update
```

## Add a Repository from PPA

```sh
$ sudo add-apt-repository ppa:<user>/<package>
$ sudo apt-get update
$ sudo apt-get install <package>
```

## Install Packages on EOL Version

Add following to `/etc/apt/sources.list`
```
## EOL upgrade sources.list
# Required
deb http://old-releases.ubuntu.com/ubuntu/ CODENAME main restricted universe multiverse
deb http://old-releases.ubuntu.com/ubuntu/ CODENAME-updates main restricted universe multiverse
deb http://old-releases.ubuntu.com/ubuntu/ CODENAME-security main restricted universe multiverse

# Optional
#deb http://old-releases.ubuntu.com/ubuntu/ CODENAME-backports main restricted universe multiverse
```

## References

* [Ask Ubuntu](https://askubuntu.com/questions/13065/how-do-i-fix-the-gpg-error-no-pubkey)
* [Ubuntu Documentation](https://help.ubuntu.com/community/EOLUpgrades)
