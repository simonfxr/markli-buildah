
# Build a basic container based on Debian unstable

## Container definition

```
### FILE: 00_FROM.txt
debian:unstable
```

Let's set our shell to bash
```
### FILE: 01_SHELL.txt
/bin/bash
```

Lets prepare a basic setup

```bash
### FILE: 10_RUN_setup.sh
set -euo pipefail
export DEBIAN_FRONTEND=noninteractive
echo "deb http://ftp2.de.debian.org/debian unstable main" >/etc/apt/sources.list
apt-get update && apt-get dist-upgrade --no-install-recommends -y
apt-get install -y --no-install-recommends ca-certificates locales
localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
echo 'LANG=en_US.UTF-8' >/etc/locale.conf
ln -sfr /usr/share/zoneinfo/Europe/Berlin /etc/localtime
apt-get clean -y && apt-get autoclean -y
rm -rf /var/lib/apt/lists/*
```

```
### FILE: 20_CONF_env.txt
--env LANG=en_US.UTF-8
--cmd '/bin/bash -li'
```

## "Compiled" buildah commands

When running `markli-buildah` in _dry-run_ mode, it will print all the `buildah`
commands it would execute. Running `markli-buildah -n --tmp-dir` from the
current directory, will result in this output:

```
Tag:             debian-unstable-basic
Markli output:   /tmp/markli-buildah-debian-unstable-basic.0gcZ32IO
Input files:     /home/simon/src/markli-buildah/examples/debian-unstable-basic/markli-buildah.md
Default shell:   /bin/sh

*** buildah from debian:unstable
*** SHELL is now /bin/bash
*** RUN /tmp/markli-buildah-debian-unstable-basic.0gcZ32IO/10_RUN_setup.sh
*** buildah run DRYRUN_CONTAINER -- /bin/bash
*** buildah config --env LANG=en_US.UTF-8 DRYRUN_CONTAINER
*** buildah config --cmd /bin/bash -li DRYRUN_CONTAINER
*** Committing DRYRUN_CONTAINER to debian-unstable-basic
*** buildah commit DRYRUN_CONTAINER debian-unstable-basic
```
