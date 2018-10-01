# Deploying GCS with Helm

## Download Helm

Helm can be obtained from the [Helm
releases](https://github.com/helm/helm/releases) page.

## Install Helm & Tiller

Once you have downloaded Helm, you need to install it. The Helm client is
installed locally, and Tiller runs within your Kubernetes cluster.

Assuming you have RBAC installed on your cluster, you need to create a service
account for Helm to use. The provided `helm-sa.yaml` creates a service account
in the `kube-system` namespace called "tiller" and gives it cluster admin
permissions. This allows Tiller to deploy charts anywhere in the cluster.

**Note: These instructions do not set up TLS security for Helm, so it should
not be considered a secure configuration. Patches welcome.**

Install the SA:

```bash
$ kubectl apply -f helm-sa.yaml
serviceaccount "tiller" created
clusterrolebinding.rbac.authorization.k8s.io "tiller" created
```

Install Tiller & initialize local Helm state:

```bash
$ helm --kubeconfig=../deploy/kubeconfig init --service-account tiller
$HELM_HOME has been configured at /home/jstrunk/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
Happy Helming!
```

Verify it is installed:

```bash
$ helm --kubeconfig=../deploy/kubeconfig version
Client: &version.Version{SemVer:"v2.10.0", GitCommit:"9ad53aac42165a5fadc6c87be0dea6b115f93090", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.10.0", GitCommit:"9ad53aac42165a5fadc6c87be0dea6b115f93090", GitTreeState:"clean"}
```

## Configure GCS for your cluster

There isn't much that is currently configurable, but what exists is in `gluster-container-storage/values.yaml`.

## Deploy GCS

Download chart dependencies (etcd-operator):

```bash
$ helm dependency update gluster-container-storage
Hang tight while we grab the latest from your chart repositories...
...Unable to get an update from the "local" chart repository (http://127.0.0.1:8879/charts):
	Get http://127.0.0.1:8879/charts/index.yaml: dial tcp 127.0.0.1:8879: connect: connection refused
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading etcd-operator from repo https://kubernetes-charts.storage.googleapis.com
Deleting outdated charts
```

Install GCS chart:

```bash
$ helm --kubeconfig=../deploy/kubeconfig install --namespace gcs gluster-container-storage
NAME:   kindred-cricket
LAST DEPLOYED: Mon Oct  1 16:12:31 2018
NAMESPACE: gcs
STATUS: DEPLOYED

RESOURCES:
==> v1beta2/Deployment
NAME                                         DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
kindred-cricket-etcd-operator-etcd-operator  1        1        1           0          3s

==> v1beta2/EtcdCluster
NAME  AGE
etcd  3s

==> v1/Pod(related)
NAME                                                         READY  STATUS             RESTARTS  AGE
csi-nodeplugin-glusterfsplugin-ggrg7                         0/2    ContainerCreating  0         3s
csi-nodeplugin-glusterfsplugin-gjx9g                         0/2    ContainerCreating  0         3s
csi-nodeplugin-glusterfsplugin-qv4ph                         0/2    ContainerCreating  0         3s
glusterd2-cluster-8zhzn                                      0/1    ContainerCreating  0         3s
glusterd2-cluster-fghw6                                      0/1    ContainerCreating  0         3s
glusterd2-cluster-p6d6v                                      0/1    ContainerCreating  0         3s
kindred-cricket-etcd-operator-etcd-operator-959d989c9-jrnjw  0/1    ContainerCreating  0         2s
csi-provisioner-glusterfsplugin-0                            0/2    ContainerCreating  0         2s
csi-attacher-glusterfsplugin-0                               0/2    ContainerCreating  0         2s

==> v1/ServiceAccount
NAME                                         SECRETS  AGE
kindred-cricket-etcd-operator-etcd-operator  1        3s
csi-attacher                                 1        3s
csi-provisioner                              1        3s
csi-nodeplugin                               1        3s

==> v1beta1/ClusterRole
NAME                                         AGE
kindred-cricket-etcd-operator-etcd-operator  3s

==> v1/ClusterRole
external-provisioner-runner  3s
csi-nodeplugin               3s
external-attacher-runner     3s

==> v1/DaemonSet
NAME                            DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE SELECTOR  AGE
csi-nodeplugin-glusterfsplugin  3        3        0      3           0          <none>         3s
glusterd2-cluster               3        3        0      3           0          <none>         3s

==> v1/StatefulSet
NAME                             DESIRED  CURRENT  AGE
csi-provisioner-glusterfsplugin  1        1        3s
csi-attacher-glusterfsplugin     1        1        3s

==> v1/StorageClass
NAME                     PROVISIONER            AGE
glusterfs-csi (default)  org.gluster.glusterfs  3s

==> v1beta1/ClusterRoleBinding
NAME                                         AGE
kindred-cricket-etcd-operator-etcd-operator  3s

==> v1/ClusterRoleBinding
csi-attacher-role     3s
csi-nodeplugin        3s
csi-provisioner-role  3s

==> v1/Service
NAME          TYPE       CLUSTER-IP   EXTERNAL-IP  PORT(S)    AGE
gluster-mgmt  ClusterIP  10.103.3.25  <none>       24007/TCP  3s
```

It will take a few minutes for the pods to start, so check back later...

```bash
$ kubectl -n gcs get po
NAME                                                          READY     STATUS    RESTARTS   AGE
csi-attacher-glusterfsplugin-0                                2/2       Running   0          2m
csi-nodeplugin-glusterfsplugin-ggrg7                          2/2       Running   0          2m
csi-nodeplugin-glusterfsplugin-gjx9g                          2/2       Running   0          2m
csi-nodeplugin-glusterfsplugin-qv4ph                          2/2       Running   0          2m
csi-provisioner-glusterfsplugin-0                             2/2       Running   0          2m
etcd-cnstrrvxk8                                               1/1       Running   0          1m
etcd-t6t5fcpqw5                                               1/1       Running   0          2m
etcd-xhv4gkrhxx                                               1/1       Running   0          2m
glusterd2-cluster-8zhzn                                       1/1       Running   0          2m
glusterd2-cluster-fghw6                                       1/1       Running   0          2m
glusterd2-cluster-p6d6v                                       1/1       Running   0          2m
kindred-cricket-etcd-operator-etcd-operator-959d989c9-jrnjw   1/1       Running   0          2m
```

Veryfy GD2 has a good cluster:

```bash
$ kubectl -n gcs exec glusterd2-cluster-8zhzn glustercli peer list
+--------------------------------------+-------------------------+------------------+------------------+--------+-----+
|                  ID                  |          NAME           | CLIENT ADDRESSES |  PEER ADDRESSES  | ONLINE | PID |
+--------------------------------------+-------------------------+------------------+------------------+--------+-----+
| 14e9b539-d27c-48b4-872a-143445c2c775 | glusterd2-cluster-fghw6 | 127.0.0.1:24007  | 10.44.0.9:24008  | yes    |  21 |
|                                      |                         | 10.44.0.9:24007  |                  |        |     |
| 5cb6bec2-e7ce-4e21-bbea-3727ffc694f7 | glusterd2-cluster-8zhzn | 127.0.0.1:24007  | 10.42.0.11:24008 | yes    |  21 |
|                                      |                         | 10.42.0.11:24007 |                  |        |     |
| 73121ee1-21c2-4741-a297-4eb9b532a44a | glusterd2-cluster-p6d6v | 127.0.0.1:24007  | 10.36.0.8:24008  | yes    |  21 |
|                                      |                         | 10.36.0.8:24007  |                  |        |     |
+--------------------------------------+-------------------------+------------------+------------------+--------+-----+
```
