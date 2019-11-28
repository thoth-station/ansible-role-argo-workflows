Ansible Role: Argo Workflows
=============================

[![Node CI](https://github.com/thoth-station/ansible-role-argo-workflows/workflows/CI/badge.svg)](https://github.com/thoth-station/ansible-role-argo-workflows/actions) &nbsp;
[![Release](https://img.shields.io/github/v/tag/thoth-station/ansible-role-argo-workflows.svg?sort=semver&label=Release)](https://github.com/thoth-station/ansible-role-argo-workflows/releases/latest)


This is an Ansible role to set up [Argo Workflows](https://argoproj.github.io/argo/) in a single  namespace.

About
-----

Check out the [blog post on Medium](https://medium.com/@marekermk/provisioning-argo-on-openshift-with-ansible-and-kustomize-340a1fda8b50) for a detailed walkthrough.


Requirements
------------

1) OpenShift cluster with cluster-admin rights.

> Why cluster-admin access is required?

The Argo creates a CRD, this operation requires (unless configured otherwise) cluster-admin priviledges.

2) Installed dependencies for the k8s ansible module

```
pip install kubernetes openshift
```

3) Installed `kubectl` (see https://kubernetes.io/docs/tasks/tools/install-kubectl/)

Install the role from Ansible Galaxy
------------------------------------

```
ansible-galaxy install thoth-station.argo_workflows
```

Role Variables
--------------

```yaml
defaults:
  # Namespace into which Argo should be provisioned
  namespace: argo

  # This assumes certain privileges and modifies the resources accordingly
  # For example, it is assumed that a developer will not be able to create
  # a CRD. These resources are therefore expected to already exist in the cluster.
  role: cluster-admin  # options: developer, cluster-admin

  # Custom overlay to be applied via kustomize to the base argo installation.
  # Overlays must be present in the [/templates/overlays/](/templates/overlays/) folder and must contain a valid `kustomization.yaml`
  overlay: ""       # options: openshift

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

  # Whether to validate kubernetes templates when submitting via kubectl
  validate: true

  # metricsConfig controls the path and port for prometheus metrics
  metricsEnabled: true
  metricsPath: /metrics
  metricsPort: 8080

  # telemetryConfig controls the path and port for prometheus telemetry
  telemetryEnabled: true
  telemetryPath: /telemetry
  telemetryPort: 8080

  # Artifacts
  # ---------

  # Kind of artifact repository to be configured (if not empty)
  # options:
  # - "s3"
  artifactRepository: ""

  # archiveLogs will archive the main container logs as an artifact
  # only applicable if artifactRepository != ""
  archiveLogs: {{ archiveLogs }}

  # s3 artifact repository configuration
  # only applicable if artifactRepository == "s3"
  AWS_S3_BUCKET_PREFIX: ""  # s3 bucket prefix
  AWS_S3_ARTIFACT_PATH: ""  # path to the artifact directory in the prefix
```

```yaml
extra:
  # Argo reference to use
  # Can be a branch, tag or a specific commit (defaults to the latest release)
  - ref

  # Allows to overwrite executor and workflow controller images
  - executor_image
  - workflow_controller_image

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
  - role: thoth-station.argo_workflows
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
  - role: thoth-station.argo_workflows
    tags:
      - argo
      - argo-workflows
    namespace: argo
    # patch the base installation to run in an OpenShift namespace
    # this also adds a Route for the argo-ui
    overlay: openshift
```

Tested on minishift, [OpenShift 3.11](https://docs.openshift.com/container-platform/3.11/welcome/index.html).

<br>

License
-------

MIT

Author Information
------------------

Marek Cermak <macermak@redhat.com>
