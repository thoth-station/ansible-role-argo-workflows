Ansible Role: Argo Workflows
=============================

This is an Ansible role to set up [Argo Workflows](https://argoproj.github.io/argo/) in a single  namespace.

**NOTE**: The role is currently in development phase and has been tested on local OpenShift-compatible cluster (i.e., [minishift](https://www.okd.io/minishift/)).

Tested on minishift, [OpenShift 3.11](https://docs.openshift.com/container-platform/3.11/welcome/index.html). 

Requirements
------------

1) OpenShift cluster with cluster-admin rights.

> Why cluster-admin access is required?

The Argo creates a CRD, this operation requires (unless configured otherwise) cluster-admin priviledges.

2) Installed Python `kubernetes` and `openshift` libraries

```bash
pip3 install kubernetes openshift
```

> Why?

This ansible role uses the `k8s` ansible plugin which requires the above libraries to be installed.


Role Variables
--------------

```yaml
defaults:
  # Namespace into which Argo should be provisioned
  namespace: argo

  # This assumes certain privileges and modifies the resources accordingly
  # For example, it is assumed that a developer will not be able to create
  # a CRD. These resources are therefore expected to already exist in the cluster.
  as: cluster-admin  # options: developer, cluster-admin

  # Custom overlay to be applied via kustomize to the base argo installation.
  # Overlays must be present in the [/templates/overlays/](/templates/overlays/) folder and must contain a valid `kustomization.yaml`
  overlay: ""       # options: openshift

  # Kind of artifact repository to be configured (if not empty)
  # options:
  # - "s3"
  ARTIFACT_REPOSITORY: ""

  # Argo container runtime executor
  #
  #     Docker

  # + supports all workflow examples
  # + most reliable and well tested
  # + very scalable. communicates to docker daemon for heavy lifting
  # - least secure. requires docker.sock of host to be mounted (often rejected by OPA)
  # 
  #     Kubelet
  # 
  # + secure. cannot escape privileges of pod's service account
  # + medium scalability - log retrieval and container polling is done against kubelet
  # - additional kubelet configuration may be required
  # - can only save params/artifacts in volumes (e.g. emptyDir), and not the base image layer (e.g. /tmp)
  # 
  #     K8s API
  # 
  # + secure. cannot escape privileges of pod's service account
  # + no extra configuration
  # - least scalable - log retrieval and container polling is done against k8s API server
  # - can only save params/artifacts in volumes (e.g. emptyDir), and not the base image layer (e.g. /tmp)
  # 
  #     PNS
  # 
  # + secure. cannot escape privileges of service account
  # + artifact collection can be collected from base image layer
  # + scalable - process polling is done over procfs and not kubelet/k8s API
  # - processes will no longer run with pid 1
  # - artifact collection from base image may fail for containers which complete too fast
  # - cannot capture artifact directories from base image layer which has a volume mounted under it
  # - immature
  executor: docker   # options: docker, kubelet, k8sapi, pns

  # s3 artifact repository configuration
  AWS_S3_BUCKET_PREFIX: ""  # s3 bucket prefix
  AWS_S3_ARTIFACT_PATH: ""  # path to the artifact directory in the prefix
```

```yaml
extra:
  # Argo reference to use
  # Can be a branch, tag or a specific commit (defaults to the latest release)
  - ref 

  # If s3 artifact repository is selected, a host (endpoint) and credentials are  required
  - AWS_S3_HOST
  - AWS_S3_BUCKET_NAME
  - AWS_S3_ACCESS_KEY_ID
  - AWS_S3_SECRET_ACCESS_KEY
```

Example Playbook
----------------

```yaml
---
- name: "A basic Play to provision Argo into a single namespace."

  hosts: localhost
  connection: local

  roles:
  - role: cermakm.argo-workflows
    tags:
      - argo
      - argo-workflows
    namespace: argo
    ref: v2.4.0
```

The role provides an option to add custom overlays via [Kustomize](https://kustomize.io/), currently there is an existing overlay for OpenShift environment. Use as such:

```yaml
---
- name: "A basic Play to provision Argo into a single namespace."

  hosts: localhost
  connection: local

  roles:
  - role: cermakm.argo-workflows
    tags:
      - argo
      - argo-workflows
    namespace: argo
    # patch the base installation to run in an OpenShift namespace
    # this also adds a Route for the argo-ui
    overlay: openshift
```

<br>

License
-------

MIT

Author Information
------------------

Marek Cermak <macermak@redhat.com>