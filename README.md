# gke-exec-credential

A helper program allowing the `ExecCredential` system to be used to authenticate to Google Kubernetes Engine.

Traditionally, following instructions
[here](https://cloud.google.com/kubernetes-engine/docs/tutorials/authenticating-to-cloud-platform#create_a_container_cluster)
or via the **Connect** button in a GCP console page such as
`https://console.cloud.google.com/kubernetes/list?project=…&organizationId=…`,
you would run a command such as

```sh
gcloud container clusters get-credentials … --zone … --project …
```

which would rewrite your `~/.kube/config` to include an authentication section such as

```yaml
users:
- name: …
  user:
    auth-provider:
      name: gcp
      config:
        access-token: …
        cmd-args: config config-helper --format=json
        cmd-path: gcloud
        expiry: …
        expiry-key: '{.credential.token_expiry}'
        token-key: '{.credential.access_token}'
```

That resulted in the client (such as `kubectl`) loading a vendor-specific authentication plugin (`gcp`).
But newer versions of Kubernetes support a [vendor-neutral scheme](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#client-go-credential-plugins)
that uses external programs of your choice to authenticate.
This scheme is [increasingly supported also by non-Golang clients](https://github.com/kubernetes/kubernetes/issues/62185).

This helper makes it easy to authenticate to GKE this way.
You need to have the `gcloud` and `jq` tools installed,
and be logged in to GCP.
Now, if this tool is installed via [Krew](https://krew.dev/), your `~/.kube/config` can read:

```yaml
users:
- name: gke
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      command: kubectl
      args:
      - gke-exec-credential
```

(If you pass any additional arguments, they will be sent as is to `gcloud`.)

Note that you can use the same user to authenticate to multiple clusters in the same project.
Each will say

```yaml
contexts:
- name: …
  context:
    cluster: …
    namespace: …
    user: gke
```
