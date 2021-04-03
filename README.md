
# Semantics of version number

jade components will share the same version number: `<major>.<section 1>.<section 2>.<section 3>.<section 4>,<section 5>,<section 6>,<section 7>`

where:
1. `<major>` denoting the release version number
2. `<section 1>` denoting the version number of `jadesdk`
3. `<section 2>` denoting the version number of `jade-go`
4. `<section 3>` denoting the version number of `jade-ui`
5. `<section 4>` denoting the version number of `jade-devops`
6. `<section 5>` denoting the version number of `plankton`
6. `<section 6>` denoting the version number of `jade-tests`
6. `<section 7>` denoting the times of trying under current version

# Bulid docker images

### prerequisites for building images on remote hosts

1. setup ssh trust between remote host and local host, or the proxy host
2. make sure **omit password** for `sudo` on remote host
3. make sure **Docker** is setup on remote host
4. make sure **GoLang** library is setup on remote host
5. make sure `GOPATH` and `GOROOT` environment variables are setup

+ reference: [how to omit password when sudo](http://jonmoore.duckdns.org/index.php/linux-articles/58-remove-sudo-password-prompt)

### remote building

Above all, pack the source code in `jade-go`, `jadesdk`, `plankton`
```bash
rm -f jade-go/app plankton/plankton jadelet.source.tar.gz
tar czf jadelet.source.tar.gz jade-go jadesdk plankton
```

Then use the `remote-build.sh` tool in `jade-go/devops` to build for target platforms
example 1: build for the cluster with amd64 architecture from remote, e.g., your laptop
```bash
./jade-go/devops/remote-build.sh \
    none none \
    aces-diamonds-ace robin \
    ./jadelet.source.tar.gz \
    build-and-push \
    robin98 <jadelet version>-amd64 \
    <docker registry password>
```

example 2: build for the Raspberry Pi device with arm32 architecture from remote, e.g., your laptop
```bash
./jade-go/devops/remote-build.sh \
    aces-diamonds-ace robin \
    aces-pi-11 pi \
    ./jadelet.source.tar.gz \
    build-and-push \
    robin98 <jadelet version>-arm32 \
    <docker registry password>
```

example 3: build for the Raspberry Pi device with arm32 architecture on cluster
```bash
./jade-go/devops/remote-build.sh \
    none none \
    aces-pi-11 pi \
    ./jadelet.source.tar.gz \
    build-and-push \
    robin98 0.4.0-arm32 \
    <docker registry password>

this tool accepts 8 parameters:
1. proxy host address, `none` for no-proxy mode
2. proxy host account, `none` for no-proxy mode
3. remote host address
4. remote host account
5. command, or the source package file name. the available commands include:
    + `reuse`: do not send the source package, but do everything else
    + `build`: only build execution files on remote host, do not transfer the source package
    + `push`: only build the docker images and push to the registry
6. docker registry
7. tag of the docker image
8. password of docker registry, to login before pushing images

### local building 

use the tool `speed-build.sh` in `jade-go/devops`

format: `./jade-go/devops/speed-build <tag> jadelet <registry>`

example:
```bash
./jade-go/devops/speed-build 0.4.0-amd64 jadelet robin98
```

# Deploy JADE system

Use tool `speed-deploy.sh` in `jade-go/devops`, if in homogeneous platform envrionment, i.e., all hosts running on the same system architecture, e.g., amd64.

However, the environment usually is in heteogeneous architecture, like the cluster of Raspberry Pi devices. It's better to deploy each platform with the core tool of deployment `k8s-deployer.py` in `jade-go/devops`. Or use `speed-deploy.sh` separately for each kind of architecture.

for lab cluster, there is an already configured env settings, just use the command to deploy:

CAUTION: make sure the env settings are updated

```bash
./jade-go/devops/speed-deploy-by-config.py \
    --master ./jade-go/devops/deployments/lab/master-node.json \
    --agents \
        ./jade-go/devops/deployments/lab/agent-nodes-1.json \
        ./jade-go/devops/deployments/lab/agent-nodes-2.json \
        ./jade-go/devops/deployments/lab/agent-nodes-3.json \
    --env-dir ./jade-go/devops/deployments/lab/env \
    --version <version>
```

# Remove JADE system

Since currently JADE is deployed via K3S, to remove JADE is to remove all pods on K3S, with/without its services.

**CAUTION**: Do NOT delete JADE services unless for a complete removal of JADE system.

```bash
# delete JADE pods:
kubectl get pods|grep jadelet|awk '{print $1}'|xargs kubectl delete pods

# delete JADE services:
kubectl get services|grep jadelet|awk '{print $1}'|xargs kubectl delete services
```




