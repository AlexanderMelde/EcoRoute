# EcoRoute — Carbon-Aware Workload Router

> **Follow-the-Sun Routing for Asynchronous Compute Workloads**  
> Dynamically routes compute-intensive jobs (AI inference, CI/CD, CMS image processing, video transcoding) to regions with peak surplus renewable energy (lowest gCO₂eq/kWh).

---

## 1. Problem & Pain Points

- **Exponential Compute Growth:** AI inference (LLMs, RAG, agents), CI/CD pipelines, and background jobs consume vast amounts of electricity.
- **Fluctuating Grid Mix:** Renewable energy shares in data centers vary drastically by weather and geography.
- **Inflexible Infrastructure:** Traditional architectures statically route traffic to fixed regions, ignoring real-time grid dynamics.
- **Misaligned Priorities:** Latency-tolerant asynchronous jobs (0.5s–minutes) are needlessly powered by fossil fuels.

---

## 2. The Solution: EcoRoute

An intelligent **reverse proxy and gateway** that inspects asynchronous HTTP requests and background jobs, dynamically dispatching them to regions with peak renewable energy generation.

```
                  ┌───────────────────────────────┐
                  │        Incoming Request       │
                  │ (LLM Call, Image Job, CI/CD)  │
                  └───────────────┬───────────────┘
                                  │
                                  ▼
                  ┌───────────────────────────────┐
                  │           EcoRoute            │
                  │   - Carbon-Aware Engine       │
                  │   - Compliance Filter(EU-only)│
                  │   - Universal HTTP / OpenAI   │
                  └───────┬───────────────┬───────┘
                          │               │
     Lowest gCO2/kWh      │               │  Fallback / Low Green
   (High Solar/Wind)      ▼               ▼
                 ┌─────────────────┐    ┌─────────────────┐
                 │ Cloud Region A  │    │ Cloud Region B  │
                 │ (e.g., Norway)  │    │(e.g., Frankfurt)│
                 │  ~25 gCO2/kWh   │    │  ~380 gCO2/kWh  │
                 └─────────────────┘    └─────────────────┘
```

### Core Features & Integration

1. **Universal HTTP Reverse Proxy & Job Dispatcher:**
   - **Arbitrary Workloads:** Routes any asynchronous HTTP request (`POST /api/v1/jobs/image-transcode`, `POST /ci/runner`, `POST /video/convert`) to workers in the greenest region.
   - **Zero-Code OpenAI Profile:** Acts as a drop-in `baseURL` replacement for any OpenAI-compatible client.
2. **Dynamic Carbon Routing:** Queries real-time grid emissions data (e.g., Electricity Maps, WattTime) to identify the region with the lowest carbon intensity (`gCO2eq/kWh`).
3. **Compliance, Sovereign Infrastructure & Geofencing:**
   - **Self-Hosted Enterprise (Perimeter Control):** Restricts target regions via `config.yaml` or ENV (e.g., `ALLOWED_REGIONS=["no-south", "de-central", "fr-north"]`) to keep workloads within sovereign/GDPR boundaries.
   - **Multi-Tenant / Edge Level:** Enforces per-request routing via subdomains (e.g., `eu.ecoroute.io`), request headers (`X-EcoRoute-Zone: EU-only`), or URL paths.
4. **Live Telemetry:** Tracks avoided CO₂ emissions in real time and calculates the _Green Routing Ratio_.
5. **MCP Server Add-On:** Exposes resources (`ecoroute://stats`) for AI agents (e.g., Claude Desktop) to inspect carbon footprints directly.

---

## 3. Architecture & Deployment Models Compared

| Model                                     | How It Works                                                                                                                                            | Latency & Resilience                                             | Hackathon Suitability                                                              | Production Readiness                                    |
| :---------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------ | :--------------------------------------------------------------- | :--------------------------------------------------------------------------------- | :------------------------------------------------------ |
| **Edge Computing (Recommended for Prod)** | Global edge worker (Cloudflare Workers, Fastly Compute). The nearest edge PoP checks the cached carbon index and forwards traffic to the target region. | **No SPOF, minimal latency** (no centralized bottleneck).        | **Very High:** Minimal code, deploys in seconds.                                   | **Highest scalability & resilience.**                   |
| **Classic Reverse Proxy**                 | Local or centralized server (Node.js/Fastify, Rust/Axum, or Go) acting as an API gateway.                                                               | **SPOF risk & double-hop** in production, but negligible on LAN. | **Optimal for booth demos:** Directly controls Pis, LEDs, and local dashboards.    | Best for on-premise or gateway clusters.                |
| **Client-Side SDK**                       | Library (`npm`/`pip`) that caches a carbon index JSON and dispatches requests client-side to the greenest region.                                       | **Zero proxy hop**, maximum decentralization.                    | **Medium:** Telemetry requires a dedicated reporting endpoint for live dashboards. | **Very High:** Ideal for serverless and microservices.  |
| **DNS / Anycast Routing**                 | Dynamic GeoDNS (e.g., Route53 / NS1) resolving domains by gCO₂eq weightings.                                                                            | **Zero latency overhead** at the HTTP layer.                     | **Low:** DNS caching (TTL) is too slow for dynamic live demos.                     | **High:** Coarse traffic balancing across data centers. |
| **Service Mesh / Sidecar**                | Envoy filter, Istio, or Traefik plugin in Kubernetes handling egress traffic.                                                                           | Highly optimized within cloud clusters.                          | **Low:** Setup complexity is too high for a hackathon.                             | **Industry Standard** for enterprise microservices.     |

> **Hackathon Strategy: Hybrid**
>
> - **Live Demo:** Local router (Node.js or Rust) communicating directly with Raspberry Pis, LEDs, and the dashboard over event Wi-Fi/LAN.
> - **Pitch & Scalability:** Highlight Edge architecture (Cloudflare Workers) and Client SDK as production pathways.

---

## 4. Workload Matrix (Fit Profile)

Optimized for **asynchronous, compute-heavy** workloads with latency tolerance $\ge 0.5\,\text{s}$:

| Workload Type                     | Example Scenario                                             | Latency Tolerance   | Carbon Impact      |
| :-------------------------------- | :----------------------------------------------------------- | :------------------ | :----------------- |
| **AI Inference (Batch / Agents)** | RAG embeddings, document analysis, auto-tagging, agent loops | High (1s–60s)       | **Extremely High** |
| **CMS Background Jobs**           | WebP conversion, thumbnail generation (50+ images)           | Medium (1s–5s)      | **High**           |
| **CI/CD Pipelines**               | Automated testing, Docker image builds, code compilation     | Very High (minutes) | **High**           |
| **Media Transcoding**             | Video encoding into HLS/DASH streaming formats               | High (minutes)      | **Extremely High** |
| **Data Crunching & Backup**       | Daily DB snapshots, ETL pipelines, analytics aggregations    | Very High (hours)   | **Medium to High** |

---

## 5. Hackathon MVP Scope

### Component 1: Core Router (Software)

- **Tech Stack Options:**
  - **Option A: TypeScript / Node.js (Fastify):** Rapid development, official MCP TypeScript SDK support, native OpenAI streaming, and WebSocket integration.
  - **Option B: Rust (Axum / Actix-web):** Minimal memory footprint and native bare-metal performance at the edge.
- **API Interfaces:**
  - **Universal HTTP Reverse Proxy:** Dynamic routing for asynchronous REST workloads (image processing, CI/CD, batch jobs) to the greenest region.
  - **OpenAI-Compatible Gateway Profile:** Drop-in proxy (`/v1/chat/completions`) for seamless LLM and agent integration.
  - **MCP Server Interface:** Exposes metrics (`ecoroute://stats`) for AI agents.
- **Routing Engine & Carbon Data Model:**
  - **Baseline Emissions Data:** Real grid data (Electricity Maps API or fallback dataset: Norway ~25 g, France ~60 g, Germany ~380 g, Poland ~650 g CO₂eq/kWh).
  - **Dynamic Sensor Override ("Flashlight Effect"):** When an I2C light sensor detects illumination (simulating solar surplus), it dynamically overrides that region's carbon intensity down to ~0 gCO₂eq/kWh.
  - **Geofencing & Constraints:** Validates regional boundaries (e.g., `EU-only`) before ranking.
  - **Dispatch:** Routes requests to the region with the lowest resulting score.

### Component 2: Live Dashboard (Visualization & Pitch Centerpiece)

- **Design Concept (Dual View):**
  - **Interactive Map / Region Cluster:** Visualizes regions (Norway, Germany, Spain). When a Pi detects light, its node illuminates ("Solar Surge Detected") with animated particles showing real-time traffic rerouting.
  - **Side-by-Side Live Comparison ("With vs. Without EcoRoute"):**
    - _Default Routing (Without EcoRoute):_ Fixed carbon-heavy region $\rightarrow$ cumulative emissions spike.
    - _EcoRoute (With Dynamic Routing):_ Green target region $\rightarrow$ displays real-time CO₂ savings in grams, the _Green Routing Ratio_, and equivalent smartphone charges saved.
- **Tech Stack:** Modern frontend (Vite / React / Tailwind) with bidirectional WebSockets for sub-50ms updates.

### Component 3: Physical Hardware Demo (Showstopper Pitch)

- **Hardware:** 3–4 Raspberry Pis representing distributed cloud regions (Nordics, Central Europe, Southern Europe).
- **Sensors:** Mini solar panels paired with I2C light sensors mounted on 3D-printed dioramas.
- **Live Interaction:** Judges shine a flashlight onto a solar panel $\rightarrow$ EcoRoute detects clean energy surplus $\rightarrow$ status LEDs on the target Pi light up synchronously with incoming traffic $\rightarrow$ dashboard reflects carbon arbitrage in real time.
- **Workload Execution Options on the Pis:**
  - _Option 1 (Local Compute):_ Real image transcoding (WebP via Sharp/ImageMagick) or a lightweight local LLM (SmolLM/Qwen via Ollama) producing real CPU load and LED activity.
  - _Option 2 (Cloud Relay):_ Pi proxies upstream calls (OpenAI, Groq, Cloudflare Workers) while driving hardware status LEDs.
  - _Option 3 (Mock Streamer):_ Synthetic responder for 100% stage reliability with simulated latency.

### Component 4: Dynamic Traffic Generator (Hands-Free Pitch Mode)

- **Autonomous Workload Simulator:** Dispatches jobs using a Poisson distribution rather than a static 1-second pulse.
- **Workload Bursts:** Simulates burst patterns (e.g., 50 batch image conversions or a CI/CD run) alternating with idle periods.
- **Pitch Advantage:** Completely hands-free operation. As judges illuminate different panels, traffic visibly shifts across the Pis and dashboard in real time.

---

## 6. Hackathon Submission Copy

### Pitch / Description

Compute-heavy workloads (AI inference, CI/CD, image processing, video transcoding) are growing rapidly, while data center renewable energy shares fluctuate constantly by weather and geography.
EcoRoute is a universal reverse proxy and API gateway that dynamically routes asynchronous HTTP requests and batch jobs to cloud regions with the highest share of clean energy (lowest gCO₂eq/kWh).
Using real-time grid APIs (such as Electricity Maps), EcoRoute monitors carbon intensity globally, dispatching tasks to regions with active renewable generation.
Operating at the edge with low latency, EcoRoute enforces regional compliance boundaries (e.g., "EU-only") and integrates into existing stacks without code changes.

### Goal / MVP by the End of the Hackathon

1. An open-source router (Node.js/Rust) dynamically dispatching asynchronous HTTP and OpenAI-compatible workloads based on carbon intensity.
2. A live dashboard visualizing avoided emissions and the Green Routing Ratio in real time.
3. A physical hardware demo using Raspberry Pis in 3D-printed dioramas with mini solar panels: shine a flashlight on a panel, traffic dynamically reroutes to that Pi, confirmed by synchronized status LEDs.

### Long-Term Viability Post-Hackathon

EcoRoute will be published open source alongside a standardized Model Context Protocol (MCP) server, allowing developers to embed carbon-aware routing into tools like Claude Desktop or CMS plugins.
Future updates will integrate dynamic cloud spot pricing APIs, turning EcoRoute into a cost-saving engine that delivers financial arbitrage alongside carbon reduction.

---

## 7. Integration Flexibility

EcoRoute integrates into existing infrastructure without architectural overhauls:

- **Zero-Config Drop-In:** Functions as a reverse proxy requiring no code changes or SDKs. Applications simply update their base URL (e.g., changing `api.openai.com` to `ecoroute.local/openai` or pointing CMS workers to the EcoRoute gateway).
- **Architectural Extension:** Complements existing load balancers (Nginx, AWS ALB, Traefik). EcoRoute can sit upstream behind ingress to intercept asynchronous workloads (e.g., `/jobs`) or operate as a sidecar providing routing decisions. Core infrastructure, SSL/TLS termination, and DDoS protection remain untouched.

---

## 8. Competitive Differentiation

Existing approaches rely on proprietary cloud ecosystems (Google Cloud Carbon-Aware Scheduling, Azure KEDA, Cloudflare Green Compute),
require invasive code integration (GSF Carbon Aware SDK), or only measure emissions retrospectively (CodeCarbon, Kepler).

EcoRoute differentiates through:

- **Provider-Agnostic:** Dynamically balances traffic across heterogeneous cloud providers without vendor lock-in.
- **Zero-Config Drop-In:** Operates as a reverse proxy via base URL updates, avoiding SDK dependencies, application rewrites, or infrastructure re-architecting.

---

## 9. Key Highlights & USPs

- **Universal Scope:** Optimizes any asynchronous compute workload (AI inference, CMS image processing, video transcoding, CI/CD).
- **Track Fit:** Directly aligns with _Sovereign, efficient infrastructure_ and _The agent-ready web_.
- **Zero-Overhead Adoption:** Drop-in proxy requiring only endpoint or `baseURL` configuration.
- **Economic Arbitrage:** Surplus renewable generation lowers wholesale electricity and cloud spot prices, reducing compute costs.
- **Dual Incentive:** Combines measurable CO₂ reduction with direct financial savings.
- **Tactile Hardware Demo:** Showcases dynamic carbon routing live with illuminated Raspberry Pis, solar panels, and status LEDs.
- **Sovereign Infrastructure:** Enforces strict compliance boundaries (`ALLOWED_REGIONS` / GDPR) while optimizing within trusted zones.

---

## 10. About the Project Leads

EcoRoute is led by **Alexander Melde** and **Milan Ferus-Comelo**, combining enterprise software development with award-winning IoT prototyping.

Alexander (M.Sc. in Computer Science) contributes deep expertise in full-stack development, TypeScript, and AI systems, paired with energy-sector domain knowledge building grid-sensor solutions.

Milan (B.Sc. in Embedded Systems) is an edge-hardware and IoT developer with international startup experience specializing in rapid prototyping.
