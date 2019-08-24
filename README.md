Argo Workflows
=========

This is an Ansible role to set up [Argo Workflows](https://argoproj.github.io/argo/) in an OpenShift namespace.

**NOTE**: The role is currently in development phase and assumes local cluster OpenShift compatible environment (i.e., [minishift](https://www.okd.io/minishift/)). With a slight configuration, it should also be possible to run it on a Kubernetes cluster.

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

  # Custom overlay to be applied via kustomize to the base argo installation.
  # Overlays must be present in the [/templates/overlays/](/templates/overlays/) folder and must contain a valid `kustomization.yaml`
  #
  # options:
  # - "openshift"
  overlay: ""

  # Kind of artifact repository to be configured (if not empty)
  # options:
  # - "s3"
  ARTIFACT_REPOSITORY: ""

  # s3 artifact repository configuration
  AWS_S3_BUCKET_PREFIX: ""  # s3 bucket prefix
  AWS_S3_ARTIFACT_PATH: ""  # path to the artifact directory in the prefix
```

```yaml
extra:
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
```

<br>

License
-------

MIT

Author Information
------------------

Marek Cermak <macermak@redhat.com>