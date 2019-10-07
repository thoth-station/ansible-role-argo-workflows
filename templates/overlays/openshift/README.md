# OpenShift

This overlay contains nested overlays specific to `cluster-admin` and `developer` privileges.

Each overlay adds a specific label `as` to filter the resources that the role can (and should) deploy.

The `base/` folder contains the base for this overlay which is common to the two overlays.