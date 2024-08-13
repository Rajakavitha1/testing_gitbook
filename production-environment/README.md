---
description: >-
  You can do a modular installation of various components of Seldon Enterprise
  Platform into an existing Kubernetes cluster to leverage the integrations that
  Seldon offers.
---

# Production Environment

## Cluster requirements

The minimum resource requirements for clusters running the full Seldon Enterprise Platform ecosystem are listed in this table. Depending on your usage, the resources that you require may vary.

| Cloud provider | Requirements      | Notes                                                                                                                        |
| -------------- | ----------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| GCP            | 3 x e2-standard-8 | This is 24 vCPUs and 96GB RAM. [Details](https://cloud.google.com/compute/docs/general-purpose-machines#e2\_machine\_types). |
| AWS            | 6 x t2.xlarge     | This is 24 vCPUs and 96GB RAM. [Details](https://aws.amazon.com/ec2/instance-types/t2/).                                     |
| Azure          | 3 x D8as or D8ads | This is 24 vCPUs and 96GB RAM. [Details](https://azure.microsoft.com/en-gb/pricing/details/virtual-machines/linux/#pricing). |
