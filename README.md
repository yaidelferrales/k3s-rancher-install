# _Rancher's k3s - First steps_

Lightweight, easy, fast Kubernetes distribution with a very small footprint https://k3s.io or https://k3d.io for the dockerized version of k3s.

## _Install Master_

Considerations:

- As of (november 2023) Rancher is having issues with versions > `v.27.0` therefore we will use the version `v1.26.10+k3s2`.
- K3s uses traefik as proxy manager by default but as this installation will use nginx ingress controller we will disable trefik.

After this small changes the install script is:

```
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.26.10+k3s2 sh -s - --disable traefik --write-kubeconfig-mode 644 --node-name k3s-master-01
```

## _Copy the k3s cluster config_

The config of the cluster is what allows `kubectl` to connect to k3s and perform all the actions on it. Run the following command to copy the cluster config to the user's home directory:

```
sudo cp /etc/rancher/k3s/k3s.yaml .kube/config && sudo chmod 600 .kube/config && sudo chown <user>:<user> .kube/config
```

## _Install Worker (Optional)_

Grab token from the master node to be able to add worked nodes to it:

```
cat /var/lib/rancher/k3s/server/node-token
```

Install k3s on the worker node and add it to our cluster using the same version as on the master node:

```
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.26.10+k3s2 K3S_NODE_NAME=k3s-worker-01 K3S_URL=https://<IP>:6443 K3S_TOKEN=<TOKEN> sh -
```

## _Install nginx ingress controller_

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.1/deploy/static/provider/cloud/deploy.yaml
```

### _Important note for AWS, Azure, GCP_

If the ingress controller service does not have as external ip the server ip run the following commands:

```
kubectl -n ingress-nginx edit svc ingress-nginx-controller
```

Then add your IP as list to spec.externalIPs like so:
```
  spec:
    externalIPs:
    - <YOUR_IP>
```




## _Install cert-manager_

Cert-manager is a popular way to add certificate management capabilities to the k3s cluster, to install it follow the instructions in the [official site](https://cert-manager.io/docs/installation/kubectl/).

If you want to be able to perform certificate related tasks from the terminal line consider installing `cmctl` following the documentation at the [official site](https://cert-manager.io/docs/reference/cmctl/).

## _Install helm_

Helm is a package manager for kubernetes, to install it follow the instructions in the [official site](https://helm.sh/docs/intro/install/#helm)

## _Install Rancher_

Rancher is a Kubernetes management tool to deploy and run clusters anywhere and on any provider.
Rancher can provision Kubernetes from a hosted provider, provision compute nodes and then install Kubernetes onto them, or import existing Kubernetes clusters running anywhere.

Install rancher following the instructions in the [official site](https://ranchermanager.docs.rancher.com/pages-for-subheaders/install-upgrade-on-a-kubernetes-cluster)

### _Fix rancher 404_

Even though rancher is ready you may face a 404 error due to an error on the rancher ingress controllers. To fix them run:

1. Get all the nginx ingress controllers
   ```
   kubectl -n cattle-system get ingress
   ```
2. Edit the acme-http-solver

   ```
   kubectl -n cattle-system edit ingress cm-acme-http-solver-<random-id>
   ```

3. Add the ingress class to the spec like so:
   ```
   spec:
     ingressClassName: nginx # This is the line to be added
   ```
4. Edit the main ingress controller:
   ```
   kubectl -n cattle-system edit ingress rancher
   ```
5. Add the ingress class to the spec like so:
   ```
   spec:
     ingressClassName: nginx # This is the line to be added
   ```
