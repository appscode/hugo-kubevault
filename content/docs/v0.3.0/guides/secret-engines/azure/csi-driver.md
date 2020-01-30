---
title: Mount Azure Secrets into Kubernetes a Pod using CSI Driver
menu:
  docs_v0.3.0:
    identifier: csi-driver-azure
    name: CSI Driver
    parent: azure-secret-engines
    weight: 15
menu_name: docs_v0.3.0
section_menu_id: guides
info:
  version: v0.3.0
---

> New to KubeVault? Please start [here](/docs/v0.3.0/concepts/README).

# Mount Azure Secrets using CSI Driver

At first, you need to have a Kubernetes 1.14 or later cluster, and the kubectl command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/). To check the version of your cluster, run:

```console
$ kubectl version --short
Client Version: v1.16.2
Server Version: v1.14.0
```

Before you begin:

- Install KubeVault operator in your cluster from [here](/docs/v0.3.0/setup/operator/install).
- Install KubeVault CSI driver in your cluster from [here](/docs/v0.3.0/setup/csi-driver/install).

To keep things isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

> Note: YAML files used in this tutorial stored in [examples](/docs/v0.3.0/examples/guides/secret-engines/azure) folder in GitHub repository [KubeVault/docs](https://github.com/kubevault/docs)

## Vault Server

If you don't have a Vault Server, you can deploy it by using the KubeVault operator.

- [Deploy Vault Server](/docs/v0.3.0/guides/vault-server/vault-server)

The KubeVault operator can manage policies and secret engines of Vault servers which are not provisioned by the KubeVault operator. You need to configure both the Vault server and the cluster so that the KubeVault operator can communicate with your Vault server.

- [Configure cluster and Vault server](/docs/v0.3.0/guides/vault-server/external-vault-sever#configuration)

Now, we have the [AppBinding](/docs/v0.3.0/concepts/vault-server-crds/auth-methods/appbinding) that contains connection and authentication information about the Vault server. And we also have the service account that the Vault server can authenticate.

```console
$ kubectl get serviceaccounts -n demo
NAME                       SECRETS   AGE
vault                      1         20h

$ kubectl get appbinding -n demo
NAME    AGE
vault   50m

$ kubectl get appbinding -n demo vault -o yaml
apiVersion: appcatalog.appscode.com/v1alpha1
kind: AppBinding
metadata:
  name: vault
  namespace: demo
spec:
  clientConfig:
    caBundle: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN1RENDQWFDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFOTVFzd0NRWURWUVFERXdKallUQWUKRncweE9URXhNVEl3T1RFMU5EQmFGdzB5T1RFeE1Ea3dPVEUxTkRCYU1BMHhDekFKQmdOVkJBTVRBbU5oTUlJQgpJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBUThBTUlJQkNnS0NBUUVBdFZFZmtic2c2T085dnM2d1Z6bTlPQ1FYClBtYzBYTjlCWjNMbXZRTG0zdzZGaWF2aUlSS3VDVk1hN1NRSGo2L2YvOHZPeWhqNEpMcHhCM0hCYVFPZ3RrM2QKeEFDbHppU1lEd3dDbGEwSThxdklGVENLWndreXQzdHVQb0xybkppRFdTS2xJait6aFZDTHZ0enB4MDE3SEZadApmZEdhUUtlSXREUVdyNUV1QWlCMjhhSVF4WXREaVN6Y0h3OUdEMnkrblRMUEd4UXlxUlhua0d1UlIvR1B3R3lLClJ5cTQ5NmpFTmFjOE8wVERYRkIydWJQSFNza2xOU1VwSUN3S1IvR3BobnhGak1rWm4yRGJFZW9GWDE5UnhzUmcKSW94TFBhWDkrRVZxZU5jMlczN2MwQlhBSGwyMHVJUWQrVytIWDhnOVBVVXRVZW9uYnlHMDMvampvNERJRHdJRApBUUFCb3lNd0lUQU9CZ05WSFE4QkFmOEVCQU1DQXFRd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBTkJna3Foa2lHCjl3MEJBUXNGQUFPQ0FRRUFabHRFN0M3a3ZCeTNzeldHY0J0SkpBTHZXY3ZFeUdxYUdCYmFUbGlVbWJHTW9QWXoKbnVqMUVrY1I1Qlg2YnkxZk15M0ZtZkJXL2E0NU9HcDU3U0RMWTVuc2w0S1RlUDdGZkFYZFBNZGxrV0lQZGpnNAptOVlyOUxnTThkOGVrWUJmN0paUkNzcEorYkpDU1A2a2p1V3l6MUtlYzBOdCtIU0psaTF3dXIrMWVyMUprRUdWClBQMzFoeTQ2RTJKeFlvbnRQc0d5akxlQ1NhTlk0UWdWK3ZneWJmSlFEMVYxbDZ4UlVlMzk2YkJ3aS94VGkzN0oKNWxTVklmb1kxcUlBaGJPbjBUWHp2YzBRRXBKUExaRDM2VDBZcEtJSVhjZUVGYXNxZzVWb1pINGx1Uk50SStBUAp0blg4S1JZU0xGOWlCNEJXd0N0aGFhZzZFZVFqYWpQNWlxZnZoUT09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    service:
      name: vault
      port: 8200
      scheme: HTTPS
  parameters:
    apiVersion: config.kubevault.com/v1alpha1
    kind: VaultServerConfiguration
    path: kubernetes
    vaultRole: vault-policy-controller
    kubernetes:
      serviceAccountName: vault
      tokenReviewerServiceAccountName: vault-k8s-token-reviewer
      usePodServiceAccountForCSIDriver: true
```

## Enable and Configure Azure Secret Engine

The following steps are required to enable and configure the Azure secrets engine in the Vault server.

There are two ways to configure the Vault server. You can either use the `KubeVault operator` or the  `Vault CLI` to manually configure a Vault server.

<ul class="nav nav-tabs" id="conceptsTab" role="tablist">
  <li class="nav-item">
    <a class="nav-link active" id="operator-tab" data-toggle="tab" href="#operator" role="tab" aria-controls="operator" aria-selected="true">Using KubeVault operator</a>
  </li>
  <li class="nav-item">
    <a class="nav-link" id="csi-driver-tab" data-toggle="tab" href="#csi-driver" role="tab" aria-controls="csi-driver" aria-selected="false">Using Vault CLI</a>
  </li>
</ul>
<div class="tab-content" id="conceptsTabContent">
  <div open class="tab-pane fade show active" id="operator" role="tabpanel" aria-labelledby="operator-tab">

### Using KubeVault operator

You need to be familiar with the following CRDs:

- [AppBinding](/docs/v0.3.0/concepts/vault-server-crds/auth-methods/appbinding)
- [SecretEngine](/docs/v0.3.0/concepts/secret-engine-crds/secretengine)
- [AzureRole](/docs/v0.3.0/concepts/secret-engine-crds/azure-secret-engine/azurerole)

Let's enable and configure Azure secret engine by deploying the following `SecretEngine` yaml:

```yaml
apiVersion: engine.kubevault.com/v1alpha1
kind: SecretEngine
metadata:
  name: azure-engine
  namespace: demo
spec:
  vaultRef:
    name: vault
  azure:
    credentialSecret: azure-cred
```

To configure the Azure secret engine, you need to provide azure credentials through a Kubernetes secret.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: azure-cred
  namespace: demo
data:
  client-secret: TU1hRjdRZWVzTG...=
  subscription-id: MWJmYzlmNjYt....
  client-id: MmI4NzFkNGEtNzU3......
  tenant-id: NzcyMjY4ZTUtZDk0MC....
```

Let's deploy SecretEngine:

```console
$ kubectl apply -f docs/examples/guides/secret-engines/azure/azureCred.yaml
secret/azure-cred created

$ kubectl apply -f docs/examples/guides/secret-engines/azure/azureSecretEngine.yaml
secretengine.engine.kubevault.com/azure-engine created
```

Wait till the status become `Success`:

```console
$ kubectl get secretengines -n demo
NAME           STATUS
azure-engine   Success
```

A sample AzureRole object is given below:

```yaml
apiVersion: engine.kubevault.com/v1alpha1
kind: AzureRole
metadata:
  name: azure-role
  namespace: demo
spec:
  vaultRef:
    name: vault
  applicationObjectID: c1cb042d-96d7-423a-8dba-243c2e5010d3
  ttl: 1h
```

Let's deploy AzureRole:

```console
$ kubectl apply -f docs/examples/guides/secret-engines/azure/azureRole.yaml
azurerole.engine.kubevault.com/azure-role created

$ kubectl get azureroles -n demo
NAME         STATUS
azure-role   Success
```

You can also check from Vault that the role is created.
To resolve the naming conflict, name of the role in Vault will follow this format: `k8s.{clusterName}.{metadata.namespace}.{metadata.name}`.

> Don't have Vault CLI? Download and configure it as described [here](/docs/v0.3.0/guides/vault-server/vault-server#enable-vault-cli)

```console
$ vault list azure/roles
Keys
----
k8s.-.demo.azure-role

$ vault read azure/roles/k8s.-.demo.azure-role
Key                      Value
---                      -----
application_object_id    c1cb042d-96d7-423a-8dba-243c2e5010d3
azure_roles              <nil>
max_ttl                  0s
ttl                      1h
```

For more detailed explanation visit [Vault official website](https://www.vaultproject.io/docs/secrets/azure/index.html#setup)

</div>
<div class="tab-pane fade" id="csi-driver" role="tabpanel" aria-labelledby="csi-driver-tab">

### Using Vault CLI

You can also use [Vault CLI](https://www.vaultproject.io/docs/commands/) to [enable and configure](https://www.vaultproject.io/docs/secrets/azure/index.html#setup) the Azure secret engine.

> Don't have Vault CLI? Download and configure it as described [here](/docs/v0.3.0/guides/vault-server/vault-server#enable-vault-cli)

To generate secret from the Azure secret engine, you need to perform the following steps.

- **Enable Azure Secret Engine:** To enable Azure secret engine run the following command.

```console
$ vault secrets enable azure
Success! Enabled the azure secrets engine at: azure/
```

- **Configure the secrets engine:** Configure the secrets engine with account credentials

```console
$ vault write azure/config \
  subscription_id=$AZURE_SUBSCRIPTION_ID \
  tenant_id=$AZURE_TENANT_ID \
  client_id=$AZURE_CLIENT_ID \
  client_secret=$AZURE_CLIENT_SECRET

Success! Data written to: azure/config
```

- **Configure a role:** Configure a role called "my-role" with an existing service principal:

```console
$ vault write azure/roles/k8s.-.demo.azure-role \
                  application_object_id=c1cb042d-96d7-423a-8dba-243c2e5010d3 \
                  ttl=1h
Success! Data written to: azure/roles/k8s.-.demo.azure-role
```

- **Read the role:**

```console
$ vault list azure/roles
Keys
----
k8s.-.demo.azure-role

$ vault read azure/roles/k8s.-.demo.azure-role
Key                      Value
---                      -----
application_object_id    c1cb042d-96d7-423a-8dba-243c2e5010d3
azure_roles              <nil>
max_ttl                  0s
ttl                      1h
```

If you use Vault CLI to enable and configure the Azure secret engine then you need to **update the vault policy** for the service account 'vault' created during vault server configuration and add the permission to read at "azure/roles/*" with previous permissions. That is why it is recommended to use the KubeVault operator because the operator updates the policies automatically when needed.

Find how to update the policy for service account in [here](/docs/v0.3.0/guides/secret-engines/kv/csi-driver#update-vault-policy).

For more detailed explanation visit [Vault official website](https://www.vaultproject.io/docs/secrets/azure/index.html#setup)

## Mount secrets into a Kubernetes pod

Since Kubernetes 1.14, `storage.k8s.io/v1beta1` `CSINode` and `CSIDriver` objects were introduced. Let's check [CSIDriver](https://kubernetes-csi.github.io/docs/csi-driver-object.html) and [CSINode](https://kubernetes-csi.github.io/docs/csi-node-object.html) are available or not.

```console
$ kubectl get csidrivers
NAME                        CREATED AT
secrets.csi.kubevault.com   2019-12-09T04:32:50Z

$ kubectl get csinodes
NAME             CREATED AT
2gb-pool-57jj7   2019-12-09T04:32:52Z
2gb-pool-jrvtj   2019-12-09T04:32:58Z
```

So, we can create `StorageClass` now.

### Create StorageClass

Create `StorageClass` object with the following content:

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: vault-azure-storage
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"
provisioner: secrets.csi.kubevault.com
parameters:
  ref: demo/vault # namespace/AppBinding, we created vault server configuration
  engine: Azure # vault engine name
  role: k8s.-.demo.azure-role # role name on vault which you want get access
  path: azure # specify the secret engine path, default is azure
```

```console
$ kubectl apply -f docs/examples/guides/secret-engines/azure/storageClass.yaml
storageclass.storage.k8s.io/vault-azure-storage created
```

## Test & Verify

Let's create a separate namespace called `trial` for testing purpose.

```console
$ kubectl create ns trail
namespace/trail created
```

### Create PVC

Create a `PersistentVolumeClaim` with the following data. This makes sure a volume will be created and provisioned on your behalf.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: csi-pvc-azure
  namespace: trial
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
  storageClassName: vault-azure-storage
```

```console
$ kubectl apply -f docs/examples/guides/secret-engines/azure/pvc.yaml
persistentvolumeclaim/csi-pvc-azure created
```

### Create VaultPolicy and VaultPolicyBinding for Pod's Service Account

Let's say pod's service account name is `pod-sa` located in `trial` namespace. We need to create a [VaultPolicy](/docs/v0.3.0/concepts/policy-crds/vaultpolicy) and a [VaultPolicyBinding](/docs/v0.3.0/concepts/policy-crds/vaultpolicybinding) so that the pod has access to read secrets from the Vault server.

```yaml
apiVersion: policy.kubevault.com/v1alpha1
kind: VaultPolicy
metadata:
  name: azure-se-policy
  namespace: demo
spec:
  vaultRef:
    name: vault
  # Here, azure secret engine is enabled at "azure".
  # If the path was "demo-se", policy should be like
  # path "demo-se/*" {}.
  policyDocument: |
    path "azure/*" {
      capabilities = ["create", "read"]
    }
---
apiVersion: policy.kubevault.com/v1alpha1
kind: VaultPolicyBinding
metadata:
  name: azure-se-role
  namespace: demo
spec:
  vaultRef:
    name: vault
  policies:
  - ref: azure-se-policy
  subjectRef:
    kubernetes:
      serviceAccountNames:
      - "pod-sa"
      serviceAccountNamespaces:
      - "trial"
```

Let's create VaultPolicy and VaultPolicyBinding:

```console
$ kubectl apply -f docs/examples/guides/secret-engines/azure/vaultPolicy.yaml
vaultpolicy.policy.kubevault.com/azure-se-policy created

$ kubectl apply -f docs/examples/guides/secret-engines/azure/vaultPolicyBinding.yaml
vaultpolicybinding.policy.kubevault.com/azure-se-role created
```

Check if the VaultPolicy and the VaultPolicyBinding are successfully registered to the Vault server:

```console
$ kubectl get vaultpolicy -n demo
NAME                           STATUS    AGE
azure-se-policy                Success   8s

$ kubectl get vaultpolicybinding -n demo
NAME                           STATUS    AGE
azure-se-role                  Success   10s
```

### Create Service Account for Pod

Let's create the service account `pod-sa` which was used in VaultPolicyBinding. When a VaultPolicyBinding object is created, the KubeVault operator create an auth role in the Vault server. The role name is generated by the following naming format: `k8s.(clusterName or -).namespace.name`. Here, it is `k8s.-.demo.azure-se-role`. We need to provide the auth role name as service account `annotations` while creating the service account. If the annotation `secrets.csi.kubevault.com/vault-role` is not provided, the CSI driver will not be able to perform authentication to the Vault.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: pod-sa
  namespace: trial
  annotations:
    secrets.csi.kubevault.com/vault-role: k8s.-.demo.azure-se-role
```

```console
$ kubectl apply -f docs/examples/guides/secret-engines/azure/podServiceAccount.yaml
serviceaccount/pod-sa created
```

### Create Pod

Now we can create a Pod which refers to this volume. When the Pod is created, the volume will be attached, formatted and mounted to the specific container.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  namespace: trial
spec:
  containers:
  - name: mypod
    image: busybox
    command:
    - sleep
    - "3600"
    volumeMounts:
    - name: my-vault-volume
      mountPath: "/etc/azure"
      readOnly: true
  serviceAccountName: pod-sa # service account that was created
  volumes:
  - name: my-vault-volume
    persistentVolumeClaim:
      claimName: csi-pvc-azure
```

```console
$ kubectl apply -f docs/examples/guides/secret-engines/azure/pod.yaml
pod/mypod created
```

Check if the Pod is running successfully, by running:

```console
$ kubectl get pods -n trial
NAME                    READY   STATUS    RESTARTS   AGE
mypod                   1/1     Running   0          27s
```

### Verify Secret

If the Pod is running successfully, then check inside the app container by running

```console
$ kubectl exec -it -n trial  mypod sh
/ # ls /etc/azure/
client_id      client_secret

/ # cat /etc/azure/client_id
2b871d4a-757e-4b2f-bc78*************/ #

/ # cat /etc/azure/client_secret
9d7ce30a-4fa5-a*********************/ #

/ # exit
```

So, we can see that the secret `client_id` and `client_secret` are mounted into the pod, where the secret key is mounted as file and the value is the content of that file.

## Cleaning up

To clean up the Kubernetes resources created by this tutorial, run:

```console
$ kubectl delete ns demo
namespace "demo" deleted

$ $ kubectl delete ns trial
namespace "trial" deleted
```