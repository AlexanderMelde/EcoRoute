# EcoRouter — Carbon-Aware Workload Router

**Follow-the-Sun Routing**:
Dynamically routes latency-tolerant asynchronous compute- and electricity-intensive jobs / workloads
(AI inference, CI/CD, CMS image processing, video transcoding)
to the "greenest" regions with peak surplus renewable energy (=lowest gCO₂eq/kWh).

Given dynamic electricity tariffs, those regions have cheaper electricity, helping
both the environment and your wallet.

```
       ┌────────────────────┐
       │  Incoming Request  │
       └──────────┬─────────┘
                  ▼
   ┌─────────────────────────────────┐
   │        EcoRouter detects:        │
   │ A has high Solar/Wind, B doesnt │
   └───────┬───────────────┬─────────┘
Chooses A  ▼               ▼
┌─────────────────┐    ┌─────────────────┐
│ Cloud Region A  │    │ Cloud Region B  │
│ (e.g., Norway)  │    │(e.g., Frankfurt)│
│  ~25 gCO2/kWh   │    │  ~380 gCO2/kWh  │
└─────────────────┘    └─────────────────┘
```

1. **Supports anything:** Routes any workload / HTTP request to workers in the greenest region.
2. **Dynamic Carbon Routing:**
   Queries real-time grid emissions APIs to identify the region with the lowest carbon intensity (`gCO2eq/kWh`).
3. **Compliant and Sovereign:** Target regions can be restricted to e.g. EU-only
   - **Optional Time-Deferred Routing:** Non-urgent jobs (e.g. backups) can wait for the greenest hour when no allowed region is green right now (`X-EcoRoute-Deadline: 6h`).
4. **Telemetry:** Tracks avoided CO₂ emissions and calculates the _Green Routing Ratio_.
5. **Agent rady:** Optimizes AI Workloads, Stats can be read via MCP
6. **Zero-Overhead Adoption:** Drop-in proxy requiring only endpoint configuration.
7. **Dual Incentive:** Combines measurable CO₂ reduction with direct financial savings.
8. **Tactile Hardware Demo:** Showcases dynamic carbon routing live with illuminated Raspberry Pis, solar panels, and status LEDs.

EcoRouter is expected to have the highest impact for long-running AI Inference and media transcoding workloads.
Further impact will be on CMS Background jobs like image conversion and thumbnail generation and CI/CD ppipeline steps.
It can also be used for Data Crunching (ETL pipelines, analytics aggregations) and Backup jobs (e.g. DB Snapshots).

## Architecture

We want to implement both a scalable performance-optimized edge-computing solution,
and a low-entry on-premise environment to integrate into existing infrastructure as a
drop-in proxy, avoiding architectural overhauls, SDK dependencies or application rewrites.

### Edge Computing Approach:

Scalable, Resilient Global edge worker (Cloudflare Workers, Fastly Compute).
Minimal code, minimal latency, no centralized bottleneck, deploys in seconds.
The nearest edge PoP checks the cached carbon index and forwards traffic to the target region.

Optional Add-On for serverless and microservices: Client-Side SDK Library (`npm`/`pip`)
that caches the carbon index data to send requests to the greenest region, without a proxy.
Fully decentralized (except Telemetry).

### On-Premise Router / Gateway Cluster Approach:

A local or centralized server as a Reverse Proxy / API gateway.
Single-point-of-failure risk & double-hop in production, but negligible on LAN.

### Further Options:

- **DNS / Anycast Routing**
  Dynamic GeoDNS (e.g., Route53 / NS1) domain lookup.
  Zero latency overhead at the HTTP layer, but DNS caching (TTL) might be too slow for some use cases.
- **Service Mesh / Sidecar for enterprise microservices in Kubernetes cloud clusters**
  Operate as a sidecar providing routing decisions
  Highly optimized but complex to setup Envoy filter, Istio, or Traefik plugin handling egress traffic.

Complements existing load balancers (Nginx, AWS ALB, Traefik), can sit upstream behind ingress.
SSL/TLS termination, and DDoS protection remain untouched.

## Tech Stack

- **Option A: TypeScript / Node.js (Fastify):** Rapid development, official MCP TypeScript SDK support, native OpenAI streaming, and WebSocket integration.
- **Option B: Rust (Axum / Actix-web):** Minimal memory footprint and native bare-metal performance at the edge.

## Deliverables

- **Universal HTTP Reverse Proxy API Interface:** Dynamic routing for REST workloads
- **Dashboard**: Visualizes real-time traffic rerouting (clients, requests, energy generation & regions)
  on a map and compares the real-time CO₂ savings side-by-side (_Green Routing Ratio_).
- Add-On: **OpenAI-Compatible LLM-Gateway Proxy:** Drop-in replacement for seamless LLM and agent integration (`/v1/chat/completions`).
- Add-On: **MCP Server Interface:** Exposes metrics (`ecoroute://stats`) for AI agents

## Inputs

- **Emissions Data** from real grid: Electricity Maps API, WattTime
- **Geofencing & Constraints:**
  Restricts target regions via `config` files, environment variables, region-specific subdomains,
  URL paths or request headers (`eu.ecoroute.io`, `X-EcoRoute-Zone: EU-only`, `ALLOWED_REGIONS=[...]`)

## Physical Hardware Live Demo

We plan to show 3–4 Raspberry Pis representing distributed cloud regions (Nordics, Central Europe, Southern Europe).
Each Pi is equipped with a mini solar panel and an I2C light sensor and mounted on a 3D-printed diorama.

Once we shine a flashlight onto a solar panel, we will simulate a solar surplus and lower the illuminated region's carbon intensity.

During the demo, Workloads will be generated automatically in realistic burst patterns.
The procy will route to the _greenest_ Pi, which will then execute the workload.
(e.g. image transcoding, lightweight local LLM or a mocked simulated latency for stage reliability)

The effectiveness of our routing will be shown by the developed dashboard and the Pis CPU load.

## Market Research

Carbon-aware cloud computing is a well-known and trending hype topic across enterprise architects
and hyperscalers, proving there is a massive real market.

Unlike EcoRoute, existing approaches rely on proprietary cloud ecosystems
(vendor lock-in, e.g. Google Cloud Carbon-Aware Scheduling, Azure KEDA, Cloudflare Green Compute),
require invasive code changes (GSF Carbon Aware SDK),
or only measure emissions retrospectively (CodeCarbon, Kepler).

An open-source drop-in-solution like EcoRouter does not seem to exist yet. See [Related Work](docs/related-work.md) for a detailed comparison.

## About the Project Leads

EcoRouter is led by **Alexander Melde** and **Milan Ferus-Comelo**, combining enterprise software development with award-winning IoT prototyping.

Alexander (M.Sc. in Computer Science) contributes deep expertise in full-stack development, TypeScript, and AI systems, paired with energy-sector domain knowledge building grid-sensor solutions.

Milan is an edge-AI and IoT developer with an electrical engineering background whose work has been awarded by the KI Bundesverband, the United Nations and the German Federal Ministry of Research, Technology and Space. He even won three awards at CloudFest Hackathon 2026.

## Summaries for the project submission:

**Pitch / Description:**
EcoRouter is a carbon-aware routing layer that sends heavy compute jobs wherever the sun is shining or the wind is blowing. By tracking real-time grid emissions, it dynamically forwards latency-tolerant workloads to cloud regions with the lowest carbon footprint. It acts as a zero-overhead, drop-in reverse proxy, meaning you don't need to rewrite any code to use it. Users slash CO₂ emissions and take advantage of cheaper renewable energy rates, all while keeping data sovereign via strict geofencing. It’s an open-source, frictionless way to make infrastructure instantly greener.

**Goal / MVP by the End of the Hackathon:**
Our MVP is a working HTTP reverse proxy that routes traffic based on live carbon data, paired with a dashboard tracking real-time CO₂ savings via a "Green Routing Ratio". To show exactly how it works, we’re bringing a tactile hardware demo: a cluster of Raspberry Pis hooked up to mini solar panels and light sensors. Shine a flashlight on a sensor to simulate a solar surplus, and watch EcoRouter instantly redirect the workload to that "green" node. We can extend the system by developing a drop-in OpenAI-compatible gateway and an MCP server so AI agents can monitor their own sustainability stats.

**Long-Term Viability Post-Hackathon:**
Green cloud computing is a massive enterprise trend, but it usually means getting locked into a specific cloud provider or dealing with complex code rewrites. EcoRouter solves this by being open-source, multi-cloud, and entirely frictionless. After the hackathon, we plan to scale it as an edge-computing solution and offer Kubernetes Service Mesh integrations. Because EcoRouter offers a powerful dual incentive by helping companies hit strict ESG climate goals while actively lowering electricity bills, it has huge commercial potential as a standard, drop-in DevOps tool.
