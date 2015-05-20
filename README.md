# docker-registry-systemd
docker registry systemd unit file


#### setup the docker to use the private registry mirror (coreos as an example)

- `touch /run/flannel_docker_opts.env`
- add `DOCKER_OPTS="--registry-mirror=http://<registry ip>:<registry port> --insecure-registry=<registry ip>:<registry port>"`
- note: need to add --insecure-registry if you run the registry without https and you want to push image into this private registry
- note: must use "=" for --insecure-registry. the offical website missed it.

### run the registry container in systemd unit

- the unit shall be put under `/etc/systemd/system/`
- it is better to attach a volume to VM (for example in openstack), then the images can be saved after VM dies. you need to mount this volume to container with `-v <your volume path>:/tmp/ `
- if you run the docker/container behind the proxy, you need to set the proxy environment for the registry contianer othewise it can't get the images from the registry-1.docker.io
- if you need to push the image to it, you need to set the environment `STANDALONE=true`
- if you need to use it as mirror, you have to set the environment `MIRROR_SOURCEE=https://registry-1.docker.io` and `MIRROR_SOURCE_INDEX=https://index.docker.io` when set `STANDALONE=true`
- in production environment with files, set environment `SETTINGS_FLAVOR=local`
- note: current only the registry v0.9.1 supports mirror.  registry v2.0 can't support mirror but private registry now. maybe registry 2.1/docker 1.7 will support mirror.

### configure docker with proxy (optional)

- `touch http-proxy.conf` in `/etc/systemd/system/docker.service.d`
- add  below content 

```
[Service]
Environment="HTTP_PROXY=http://<proxy ip>:<proxy port>" "HTTPS_PROXY=http://<proxy ip>:<proxy port>" "NO_PROXY=127.0.0.1,localhost,<registry ip>,/var/run/docker.sock" `
```

- note: Only can set one Environment
- note: NO_PROXY only can support full ip. it can't support ip with widecast


### add volume for coreos under openstack(optinal)

- in openstact web console, create a volume
- in the controller node, use nova command to attach the volume to coreso vm: `nova volume-attach <vm id> <volume id> [device name]`. note: default device name is `/dev/vdb`
- in coreos, format the volume: `mkfs.ext4 /dev/vdb`
- mount the volume with systemd unit
- note: the unit name must be <full path>.mount. the subdir in the full path is connected with `-`. For exmaple, the full path name is `/mirror/registry`, then the unit name is `mirror-registry.mount`
- the unit shall be put under `/etc/systemd/system/`
