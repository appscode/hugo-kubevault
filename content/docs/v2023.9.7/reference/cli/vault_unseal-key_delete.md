---
title: Vault Unseal-Key Delete
menu:
  docs_v2023.9.7:
    identifier: vault-unseal-key-delete
    name: Vault Unseal-Key Delete
    parent: reference-cli
menu_name: docs_v2023.9.7
section_menu_id: reference
info:
  cli: v0.16.0
  installer: v2023.9.7
  operator: v0.16.0
  unsealer: v0.16.0
  version: v2023.9.7
---

## vault unseal-key delete

delete vault unseal-key

### Synopsis


$ kubectl vault unseal-key delete vaultserver <name> -n <namespace> [flags]

Examples:
 # delete the unseal-key with name set by --key-name flag
 $ kubectl vault unseal-key delete vaultserver vault -n demo --key-name <name>

 # delete the unseal-key with name set by --key-id flag
 $ kubectl vault unseal-key delete vaultserver vault -n demo --key-id <id>


```
vault unseal-key delete [flags]
```

### Options

```
  -h, --help              help for delete
      --key-id int        delete the latest unseal key with id
      --key-name string   delete unseal key with key-name
```

### Options inherited from parent commands

```
      --as string                      Username to impersonate for the operation. User could be a regular user or a service account in a namespace.
      --as-group stringArray           Group to impersonate for the operation, this flag can be repeated to specify multiple groups.
      --as-uid string                  UID to impersonate for the operation.
      --cache-dir string               Default cache directory (default "/home/runner/.kube/cache")
      --certificate-authority string   Path to a cert file for the certificate authority
      --client-certificate string      Path to a client certificate file for TLS
      --client-key string              Path to a client key file for TLS
      --cluster string                 The name of the kubeconfig cluster to use
      --context string                 The name of the kubeconfig context to use
      --insecure-skip-tls-verify       If true, the server's certificate will not be checked for validity. This will make your HTTPS connections insecure
      --kubeconfig string              Path to the kubeconfig file to use for CLI requests.
      --match-server-version           Require server version to match client version
  -n, --namespace string               If present, the namespace scope for this CLI request
      --request-timeout string         The length of time to wait before giving up on a single server request. Non-zero values should contain a corresponding time unit (e.g. 1s, 2m, 3h). A value of zero means don't timeout requests. (default "0")
  -s, --server string                  The address and port of the Kubernetes API server
      --tls-server-name string         Server name to use for server certificate validation. If it is not provided, the hostname used to contact the server is used
      --token string                   Bearer token for authentication to the API server
      --user string                    The name of the kubeconfig user to use
```

### SEE ALSO

* [vault unseal-key](/docs/v2023.9.7/reference/cli/vault_unseal-key)	 - get, set, delete, list and sync unseal-key
