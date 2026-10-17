# ModelServe

A small Kubernetes design for an internal ML inference API. I used this as a way to explore how the app-facing resources fit together in Meshery: ingress, service discovery, workload health, scaling, and a couple of basic guardrails.

## What is in the design?

- `Namespace` keeps the resources grouped.
- `Deployment` runs two replicas and defines resource requests/limits, probes, and a restricted container security context.
- `Service` gives the replicas a stable in-cluster endpoint.
- `Ingress` shows the external HTTP route. The hostname is an example, not a live endpoint.
- `HorizontalPodAutoscaler` scales between two and eight replicas based on CPU utilization.
- `PodDisruptionBudget` keeps at least one replica available during voluntary disruptions.
- `NetworkPolicy` limits ingress to pods in the `ingress-nginx` namespace.
- `ConfigMap` holds non-secret runtime settings, and the `ServiceAccount` avoids automatically mounting a token.

## Files

- `modelserve-meshery-design.yaml` — Kubernetes manifest to import or use as the source for the design.
- `architecture.mmd` — Mermaid diagram of the request path and supporting resources.

## Importing into Meshery

1. Open [Meshery Playground](https://playground.meshery.io/).
2. Use the design import workflow and select the Kubernetes manifest option if it is available in your current Playground version.
3. Import `modelserve-meshery-design.yaml`, inspect the resulting components and relationships, and save the design.
4. If your task expects a native Meshery Design export rather than a manifest, export the saved design from Meshery and include that exported file as well.

## Before deploying

The container image is intentionally a placeholder. NGINX is not an ML inference server. Replace it with your own FastAPI, TorchServe, Triton, or ONNX Runtime image and update the port, probes, and runtime configuration to match. The example also assumes an ingress controller exists in the cluster. Add TLS, secrets management, model artifact storage, and metrics before treating this as a production deployment.

## Design trade-offs

I kept the example small enough to inspect in one sitting. CPU-based scaling is simple to demonstrate, but real inference services may need request-rate, queue-depth, latency, or GPU-utilization metrics. The ingress rule is HTTP-only to keep the topology clear; a real deployment should use TLS and a managed hostname.
