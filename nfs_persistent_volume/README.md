# Kubernetes NFS-Client Provisioner

**nfs-client** is an automatic provisioner that use your *existing and already configured* NFS server to support dynamic provisioning of Kubernetes Persistent Volumes via Persistent Volume Claims. Persistent volumes are provisioned as ``${namespace}-${pvcName}-${pvName}``.
Refer to https://github.com/kubernetes-incubator/external-storage/tree/master/nfs-client

# Steps to deploy nfs-client to our cluster.

**Step 1: Create Filestore instance**. Create a preminum Google Cloud Filestore instance in the same network and region as that of the GKE cluster

**Step 2: Get connection information for the Filestore instance**. Get the IP address of the Google Cloud Filestore instance we created in the previous step

**Step 3: Setup authorization**.

```
$ kubectl create namespace cicd
$ kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value account)
$ kubectl create -f rbac.yaml
```

**Step 4: Configure the NFS-Client provisioner**.

Edit the nfs-client-provisioner.yaml and update the connection details (IP address) at two places in the file

```
$ kubectl create -f nfs-client-provisioner.yaml
$ kubectl create -f storage-class.yaml
```

**Step 5: Test the setup**.

Create a test Persistent Volume Claim and test pod that uses the Persistent Volume Claim

```sh
$ kubectl create -f test-claim.yaml -f test-pod.yaml
```

Exec in to the container and write a test file on the Persistent Volume, and delete the test pod

```sh
$ kubectl get pods
$ kubectl exec -it test-pod -- /bin/sh
# touch /mnt/testfile.txt
# echo success > /mnt/testfile.txt
# exit
$ kubectl delete pods test-pod
```

Recreate the test pod and check if the test file content has persisted

```sh
$ kubectl create -f test-pod.yaml
$ kubectl get pods
$ kubectl exec -it test-pod -- /bin/sh
# cat /mnt/testfile.txt
success
# exit
$ kubectl delete -f test-pod.yaml -f test-claim.yaml
```
