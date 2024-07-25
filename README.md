# FlightCTL POC 
This repo just contains steps take to test FlightCtl bits


FlightCTL [UI](https://ui.flightctl.poc-01.edge-devices.net/)

```yaml
Credentials: 
    user: poc01
    pass: *****
```

On MacOS Podman must be running in rootful mode. Follow steps below

```sh
podman machine stop
podman machine set --rootful
podman machine start
```

Install QEMU

```
brew install qemu
```

Pull the base image to use to create disk image

```
sudo podman pull quay.io/flightctl/flightctl-agent-centos:poc-01
```

Create a config.toml file to include a default user

```toml
[[customizations.user]]
name = "rprakashg"
password = "R3dh@t!"
key = "ssh-rsa AAA ... user@email.com"
groups = ["wheel"]
```

Buid disk image

```
sudo podman run \
    --rm \
    -it \
    --privileged \
    --pull=newer \
    --security-opt label=type:unconfined_t \
    -v $(pwd)/config.toml:/config.toml:ro \
    -v $(pwd)/output:/output \
    -v /var/lib/containers/storage:/var/lib/containers/storage \
    quay.io/centos-bootc/bootc-image-builder:latest \
    --type qcow2 \
    --local \
    quay.io/flightctl/flightctl-agent-centos:poc-01
```

```
qemu-system-x86_64 \
    -m 4096 \
    -cpu host \
    -smp 2 \
    -hda output/qcow2/disk.qcow2 \
    -boot c \
    -netdev user,id=n1,hostfwd=tcp::2222-:22 \
    -device virtio-net,netdev=n1 \
    -nographic -M accel=hvf -name "vm01,guest=edge-device-test-ram" \
    -name "vm01,guest=edge-device-ram-test"
```

SSH into device

```
ssh -p 2222 rprakashg@localhost
```
