# Fedora [![build status](https://gitlab.com/nvidia/container-images/driver/badges/master/build.svg)](https://gitlab.com/nvidia/driver/commits/master)

See <https://github.com/NVIDIA/nvidia-docker/wiki/Driver-containers-(Beta>)

Building and running locally:

    DRIVER_VERSION=450.51.05
    FEDORA_VERSION=32
    sudo podman build \
        --build-arg FEDORA_VERSION=$FEDORA_VERSION \
        --build-arg DRIVER_VERSION=$DRIVER_VERSION \
        -t docker.io/nvidia/driver:$DRIVER_VERSION-fedora$FEDORA_VERSION .
    sudo podman run --name nvidia-driver --privileged --pid=host \
        -v /run/nvidia:/run/nvidia:shared \
        docker.io/nvidia/driver:$DRIVER_VERSION-fedora$FEDORA_VERSION \
        --accept-license

This subdirectly is a fork of
<https://gitlab.com/nvidia/container-images/driver/-/tree/master/fedora> with some downstream
changes to support Fedora CoreOS. The Fedora CoreOS [GPU
operator](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/getting-started.html)
driver built from this repo is available at: <https://ghcr.io/stackhpc/driver>.

## Caveat

At present, the GPU operator has only worked with `selinux_mode=permissive`
label which can be on Magnum. It did not work with `selinux_mode=disabled` as
the driver installer expects to be able to run `chcon` which fails when selinux
is disabled. It also hasn't been tested with `selinux_mode=enabled` which is
the default behaviour in Magnum for Fedora CoreOS driver..

## Image

Build and push the image:

    make build
    make push

## Magnum Nodegroup

Prior to installing helm chart, make sure that you add GPU flavor nodegroup to
your cluster. To create a GPU flavor nodegroup, add `--role gpu` flag:

    openstack coe nodegroup create k8s-cluster --flavor m1.gpu --role gpu gpu-worker

When the node is ready, the nodegroup should be visible:

    kubectl get nodes -L magnum.openstack.org/role
    NAME                                   STATUS   ROLES    AGE    VERSION   ROLE
    k8s-cluster-654l4zihkwov-master-0     Ready    master   151m   v1.20.6   master
    k8s-cluster-654l4zihkwov-node-0       Ready    <none>   147m   v1.20.6   worker
    k8s-cluster-gpu-worker-kwxqc-node-0   Ready    <none>   81s    v1.20.6   gpu

The GPU operator chart does auto discovery of nodes with GPUs so unlike the
[nvidia-driver-container](https://github.com/stackhpc/nvidia-driver-container)
chart, the `gpu` role is not strictly necessary.

Additional information on nodegroups can be found at:
<https://docs.openstack.org/magnum/latest/user/#node-groups>.

## Helm Chart

To install the chart to `kube-system` namespace:

    make install

NOTE: The helm chart only installs drivers on nodegroups with
`magnum.openstack.org/role=gpu` label by default. Appropriate modifications
need to be made to `values.yml` for other scenarious.

## Development

During development, one can inspect the daemonset logs with:

    make logs

And look at the pods in `gpu-operator-resources` namespace with:

    make pods

And restart the daemonset with the latest images with:

    make restart

## Test workflow

To test the usability of GPUs after deploying both daemonsets, there is a
CUDA sample pod that runs an nbody simulation. Check the results with:

    kubectl apply -f test/cuda-sample-nbody.yaml
    kubectl logs cuda-sample-nbody
