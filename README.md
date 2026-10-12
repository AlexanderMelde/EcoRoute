# EcoRoute — Carbon-Aware Workload Router

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
   │        EcoRoute detects:        │
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
4. **Telemetry:** Tracks avoided CO₂ emissions and calculates the _Green Routing Ratio_.
5. **Agent rady:** Optimizes AI Workloads, Stats can be read via MCP
6. **Zero-Overhead Adoption:** Drop-in proxy requiring only endpoint configuration.
7. **Dual Incentive:** Combines measurable CO₂ reduction with direct financial savings.
8. **Tactile Hardware Demo:** Showcases dynamic carbon routing live with illuminated Raspberry Pis, solar panels, and status LEDs.

EcoRoute is expected to have the highest impact for long-running AI Inference and media transcoding workloads.
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

An open-source drop-in-solution like EcoRoute does not seem to exist yet.

## About the Project Leads

EcoRoute is led by **Alexander Melde** and **Milan Ferus-Comelo**, combining enterprise software development with award-winning IoT prototyping.

Alexander (M.Sc. in Computer Science) contributes deep expertise in full-stack development, TypeScript, and AI systems, paired with energy-sector domain knowledge building grid-sensor solutions.

Milan (B.Sc. in Embedded Systems) is an edge-hardware and IoT developer with international startup experience specializing in rapid prototyping.

## Summaries for the project submission:

**Pitch / Description:**
EcoRoute is a "follow-the-sun" carbon-aware workload router that dynamically directs latency-tolerant
compute tasks to cloud regions currently powered by surplus renewable energy. Acting as a zero-overhead,
drop-in reverse proxy, it uses real-time grid emission APIs to identify zones with the lowest carbon
intensity (gCO₂eq/kWh). By routing heavy asynchronous workloads—such as AI inference, video transcoding,
CI/CD pipelines, and data crunching—to the "greenest" regions, EcoRoute slashes carbon footprints while
lowering electricity costs. Designed to be data-sovereign (supporting geo-fencing like EU-only) and
telemetry-rich, it offers a sustainable, open-source alternative to proprietary cloud tools without
requiring invasive application rewrites.

**Goal / MVP by the End of the Hackathon:**

The hackathon MVP delivers a fully functional HTTP Reverse Proxy capable of dynamic carbon routing.
A core deliverable is a live dashboard visualizing real-time traffic rerouting and comparing CO₂
savings via a "Green Routing Ratio." To make this tangible, the team will showcase an interactive
hardware demo using Raspberry Pis equipped with solar panels and light sensors.
Shining a flashlight on a sensor will simulate a renewable energy surplus, triggering the router
to instantly redirect simulated workloads to the newly "green" node.
Additionally, the MVP will include an OpenAI-compatible LLM gateway for seamless AI integration and
an MCP server interface to expose sustainability metrics directly to AI agents.

**Long-Term Viability Post-Hackathon:**
Carbon-aware computing is a rapidly growing enterprise priority, yet current solutions force vendor
lock-in (e.g., Google/Azure) or require complex SDK integrations. EcoRoute fills this gap as a
universally adaptable, open-source tool. Post-hackathon, the architecture will expand into a
globally scalable edge computing solution (via Cloudflare/Fastly) and offer Kubernetes Service
Mesh integrations for enterprise microservices. Its dual incentive—measurably slashing CO₂ emissions
while exploiting cheaper renewable electricity rates—guarantees strong commercial appeal.
As corporate ESG regulations tighten globally, a multi-cloud, drop-in green routing solution
likeEcoRoute is positioned to become an essential, frictionless DevOps standard.
