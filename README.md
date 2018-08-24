# Demo

## Pre Requisites

- https://www.virtualbox.org/wiki/Downloads
- https://www.vagrantup.com/downloads.html
- `brew install coreos-ct`

## Quick Start

On a master control plane (v1.10.x) create a new token:

```
kubeadm token create --print-join-command
```

Clone this repository and use output of the command above to join

```
vagrant plugin install vagrant-ignition
git clone https://github.com/sandromello/kube-box.git
cd kube-box
ct --platform=vagrant-virtualbox < cl.conf > config.ign
vagrant up
vagrant ssh

kubeadm join (...)
```

Allow communication between api-server and kubelet (exec, logs, port-forward, ...)

```
ssh -R 10250:localhost:10250 serveo.net
```

> The `./manifests` directory contain the templates used on the master.
> Also the kubeadm.cfg used to bootstrap the control plane

# Kubernetes Bare Metal Box

- DNS must be deployed as daemonset, it should be configurable to access the external api server:

```yaml
# coredns daemonset
(...)
        env:
        - name: KUBERNETES_SERVICE_HOST
          value: <api-server-external-dns>
        - name: KUBERNETES_SERVICE_PORT
          value: "443"
(...)
```

- Configure DNS service with Session Affinity and 10 seconds of timeout

```yaml
# coredns service
# it will allow fail fast and query the proper coredns endpoint
(...)
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10
(...)
```

- Calico Node must reach an external api server, customize a kubeconfig

```yaml
(...)
            env:
            - name: KUBECONFIG
              value: /etc/cni/net.d/calico-kubeconfig
(...)
```
