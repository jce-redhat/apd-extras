# APD Nginx Web Server

Deploys a shared nginx web server on OpenShift backed by a `ReadWriteMany` PVC. The PVC acts as shared content storage — other demos can write content into subdirectories on the PVC and have it immediately served by the running nginx instance without any restart or redeployment.

## Prerequisites

- OCS/ODF storage available (`ocs-external-storagecluster-cephfs` storage class) for the `ReadWriteMany` PVC
- **OpenShift Credential** — cluster-admin bearer token for the target OpenShift cluster

## Job Templates

* **Infra | Nginx | Deploy Nginx web server on OpenShift** — creates the namespace, PVC, Deployment, Service, primary Route, and alias Route. Idempotent; safe to re-run to apply configuration changes.

* **Infra | Nginx | Remove Nginx from OpenShift** — deletes the `apd-nginx` namespace and all resources within it, including the PVC and all web content.

## Configuration

| Parameter | Value |
|---|---|
| Namespace | `apd-nginx` |
| PVC name | `apd-nginx-web-content` |
| PVC size | `1Gi` |
| Storage class | `ocs-external-storagecluster-cephfs` (ReadWriteMany) |
| Access mode | `ReadWriteMany` |
| nginx image | `docker.io/nginxinc/nginx-unprivileged:stable-alpine` |
| Document root | `/usr/share/nginx/html` |
| Service port | `8080` |
| Primary URL | `https://apd-nginx-apd-nginx.<cluster-domain>` |
| Alias URL | `https://apd-content.<cluster-domain>` |

## Accessing the Web Server

After deployment, two URLs are both valid entry points to the same nginx instance:

- **Primary Route** — `https://apd-nginx-apd-nginx.<cluster-domain>` (auto-generated OpenShift Route name)
- **Alias Route** — `https://apd-content.<cluster-domain>` (friendly alias for use in demos)

Both URLs are printed in the job output at the end of the deploy run.

A default landing page is written to the PVC on first deployment. It is not overwritten on subsequent runs or pod restarts.

## Writing Content from Other Demos

Any demo that needs to publish web content via this server should:

1. Write files to a local staging path in the AAP execution environment (e.g. `/tmp/<demo-name>/`)
2. Use `kubernetes.core.k8s_cp` to copy the staging directory into the nginx pod under `/usr/share/nginx/html/<demo-name>/`

The content is immediately accessible at `https://apd-content.<cluster-domain>/<demo-name>/`.

**Example tasks:**

```yaml
- name: Get nginx pod name
  kubernetes.core.k8s_info:
    api_version: v1
    kind: Pod
    namespace: apd-nginx
    label_selectors:
      - app=apd-nginx
  register: _nginx_pod

- name: Write content locally
  ansible.builtin.copy:
    dest: /tmp/my-demo/index.html
    content: '<h1>My Demo</h1>'

- name: Copy content into nginx pod
  kubernetes.core.k8s_cp:
    namespace: apd-nginx
    pod: '{{ _nginx_pod.resources[0].metadata.name }}'
    container: nginx
    local_path: /tmp/my-demo
    remote_path: /usr/share/nginx/html/my-demo
    state: to_pod
```

To remove content, use `kubernetes.core.k8s_exec` with a plain string command:

```yaml
- name: Remove content
  kubernetes.core.k8s_exec:
    namespace: apd-nginx
    pod: '{{ _nginx_pod.resources[0].metadata.name }}'
    command: 'rm -rf /usr/share/nginx/html/my-demo'
```

## Notes

- **Directory autoindex** — nginx is configured with `autoindex on`, so browsing a directory that has no `index.html` returns a file listing. This makes it easy to navigate demo content directories in a browser without needing to know exact filenames.
- **nginx-unprivileged** — the standard `nginx` image requires root to bind port 80. `nginx-unprivileged` runs as a non-root user on port 8080 and works with OpenShift's default restricted SCC without any additional permissions.
- **ReadWriteMany PVC** — CephFS is used instead of Ceph RBD because RBD only supports `ReadWriteOnce`. RWX is required so that the running nginx pod holds the PVC mounted while other workloads write content via `k8s_cp` without needing to unmount and remount.
- **container: nginx** — the pod spec includes an initContainer (`init-content`) that writes the default landing page on first deployment. Because the pod has more than one container, the `container` parameter must be specified explicitly when using `k8s_cp` or `k8s_exec`.
- **Alias Route** — `apd-content.<cluster-domain>` is a second OpenShift Route in the `apd-nginx` namespace pointing at the same Service. No DNS changes are needed; the cluster's wildcard DNS record covers all subdomains under `apps.<cluster-domain>`. The alias hostname is controlled by the `_alias_name` variable in `deploy-nginx.yml`.
- **k8s_exec command format** — pass `command` as a plain string (not a YAML list) to avoid a `shlex` parsing issue in the `kubernetes.core` collection where list items are stringified rather than passed as separate exec arguments.
