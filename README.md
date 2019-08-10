# home-k8s-setup-1
Vagrant initialization of home kubernetes setup.
## Before Running Locally...
``` bash
# Generate random KUBETOKEN for kubeadm join (be sure to safely store the generated value for reuse)
export KUBETOKEN="$(tr -cd 'a-z0-9' < /dev/urandom | fold -w 6 | head -n 1).$(tr -cd 'a-z0-9' < /dev/urandom | fold -w 16 | head -n 1)"
```
