---
description: >-
  You can install various components of the Seldon Enterprise Platform modularly
  into an existing Kubernetes cluster to take advantage of Seldon's
  integrations.
---

# Production Environment

## Cluster requirements

This table lists the minimum resource requirements for clusters running the full Seldon Enterprise Platform ecosystem. Your actual resource needs may vary depending on usage.

<table><thead><tr><th width="158">Cloud provider</th><th width="167">Requirements</th><th>Notes</th></tr></thead><tbody><tr><td>GCP</td><td>3 x e2-standard-8</td><td>This is 24 vCPUs and 96GB RAM. <a href="https://cloud.google.com/compute/docs/general-purpose-machines#e2_machine_types">Details</a>.</td></tr><tr><td>AWS</td><td>6 x t2.xlarge</td><td>This is 24 vCPUs and 96GB RAM. <a href="https://aws.amazon.com/ec2/instance-types/t2/">Details</a>.</td></tr><tr><td>Azure</td><td>3 x D8as or D8ads</td><td>This is 24 vCPUs and 96GB RAM. <a href="https://azure.microsoft.com/en-gb/pricing/details/virtual-machines/linux/#pricing">Details</a>.</td></tr></tbody></table>

## Recommended installation sequence

1. [Seldon Enterprise Platform](seldon-enterprise-platform.md)
2. [Ingress Controller](ingress-controller/)
3. [PostgreSQL](postgresql/)
4. Observability and Alerting with Prometheus
5. Seldon Core v2 & Kafka integration
6. Elasticsearch / OpenSearch
7. ML data and events logging with Elasticsearch
