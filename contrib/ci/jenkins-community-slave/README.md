# Gluon CI using Jenkins

## Requirements
- Linux system
  - with docker installed
  - with Hardware Virtualisation (KVM Support)
    - Verify using: `lscpu | grep vmx`
    - If machine is virtualized host needs to load `kvm_intel` with `nested=1` option and cpuflags need to include `vmx`

## Architecture

![Screenshot from 2019-09-24 00-20-32](https://user-images.githubusercontent.com/601153/65468827-9edf2c80-de65-11e9-9fe0-56c3487719c3.png)

## Installation
You can support the gluon CI with your infrastructure:
1. You need to query @lemoer (freifunk@irrelefant.net) for credentials.
2. He will give you a `SLAVE_NAME` and a `SLAVE_SECRET` for your host.
3. Then go to your docker host and substitute the values for  `SLAVE_NAME` and a `SLAVE_SECRET` in the following statements:
``` shell
git clone https://github.com/freifunk-gluon/gluon/
cd gluon/contrib/ci/jenkins-community-slave/
docker build -t gluon-jenkins .
mkdir /var/cache/openwrt_dl_cache/
chown 1000:1000 /var/cache/openwrt_dl_cache
echo "z /dev/kvm 0666 - kvm -" > /etc/tmpfiles.d/kvm.conf
systemd-tmpfiles --create
docker run --detach --restart always \
    --env "SLAVE_NAME=whoareyou" \
    --env "SLAVE_SECRET=changeme" \
    --device /dev/kvm:/dev/kvm \
    --volume /var/cache/openwrt_dl_cache/:/dl_cache \
    gluon-jenkins
```
4. Check whether the instance is running correctly:
   - Your node should appear [here](https://build.ffh.zone/label/gluon-docker/).
   - When clicking on it, Jenkins should state "Agent is connected." like here: 
![Screenshot from 2019-09-24 01-00-52](https://user-images.githubusercontent.com/601153/65469209-dac6c180-de66-11e9-9d62-0d1c3b6b940b.png)
5. **Your docker container needs to be rebuilt, when the build dependencies of gluon change. As soon as build dependencies have changed, the build dependency api level has to be raised.** After you rebuilt your docker container, notify @lemoer, so he can bump the versioning number.

## Backoff
- If @lemoer is not reachable, please be patient at first if possible. Otherwise contact info@hannover.freifunk.net or join the channel `#freifunkh` on hackint.
