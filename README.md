# EcoRoute — Carbon-Aware Workload Router

> **Follow-the-Sun Routing for Asynchronous Compute Workloads**  
> Dynamically routes compute-intensive jobs (AI inference, CI/CD, CMS image processing, video transcoding) to regions with peak surplus renewable energy (lowest gCO₂eq/kWh).

---

## 1. Problem & Pain Points

- **Exponential Compute Growth:** AI inference (LLMs, RAG, autonomous agents), CI/CD pipelines, and background processing consume immense amounts of electricity.
- **Fluctuating Power Grid Mix:** The share of renewable energy in data centers varies drastically depending on solar radiation, wind speeds, and geography.
- **Inflexible Infrastructure:** Legacy architectures statically route traffic to fixed servers or regions, completely blind to real-time energy grid dynamics.
- **Misaligned Priorities:** Latency-tolerant asynchronous jobs (tolerating 0.5s to several minutes) are needlessly powered by carbon-heavy fossil fuels.

---

## 2. The Solution: EcoRoute

An intelligent, universal **reverse proxy & gateway** that inspects asynchronous HTTP requests and background jobs, dynamically dispatching them to wherever the sun is literally shining and the wind is blowing.

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
   - **Arbitrary Workloads:** Transparently routes any asynchronous HTTP request (e.g., `POST /api/v1/jobs/image-transcode`, `POST /ci/runner`, `POST /video/convert`) to workers in the greenest region.
   - **Zero-Code OpenAI Profile:** For AI workloads, EcoRoute acts as a drop-in `baseURL` replacement for any OpenAI-compatible client.
2. **Dynamic Carbon Routing:** Queries live grid emissions data (e.g., Electricity Maps / WattTime) and identifies the region with the lowest carbon intensity (`gCO2eq/kWh`).
3. **Compliance, Sovereign Infrastructure & Geofencing:**
   - **Self-Hosted Enterprise (Infrastructure Perimeter):** Organizations configure approved target regions in `config.yaml` / ENV (e.g., `ALLOWED_REGIONS=["no-south", "de-central", "fr-north"]`). Workloads never leave the sovereign/GDPR zone; EcoRoute optimizes the power mix strictly _within_ this trusted boundary.
   - **Multi-Tenant / Edge Level:** Dynamic per-request routing via subdomains (e.g., `eu.ecoroute.io`), request headers (`X-EcoRoute-Zone: EU-only`), or URL paths.
4. **Live Telemetry:** Tracks avoided CO₂ emissions in real time and calculates the _Green Routing Ratio_.
5. **Optional: MCP Server Add-On:** Exposes resources (`ecoroute://stats`) for AI agents (e.g., Claude Desktop) to query their carbon footprint transparently.

---

## 3. Architecture & Deployment Models Compared

| Model                                     | How It Works                                                                                                                                                                               | Latency & Resilience                                                            | Hackathon Suitability                                                                                  | Production Readiness                                                                   |
| :---------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------ | :----------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------- |
| **Edge Computing (Recommended for Prod)** | Global edge worker (Cloudflare Workers, Fastly Compute). The request hits the geographically closest edge PoP, checks the cached carbon index, and forwards directly to the target region. | **No SPOF, minimal latency** (no centralized bottleneck).                       | **Very High:** Few lines of code, deployed in seconds.                                                 | **Highest scalability & resilience.**                                                  |
| **Classic Reverse Proxy**                 | Local or centralized server (Node.js/Fastify, Rust/Axum, or Go) acting as an API gateway/middleware.                                                                                       | **SPOF risk & double hop** in global production, but negligible on a local LAN. | **Optimal for booth demos:** Directly controls Pis, LEDs, and the dashboard on a local subnet.         | Best suited for on-premise or gateway clusters.                                        |
| **Client-Side SDK**                       | Library (`npm`/`pip`) that periodically caches a lightweight carbon index (JSON) and dispatches requests client-side directly to the greenest region.                                      | **Zero proxy hop**, maximum decentralization.                                   | **Medium:** Decentralized telemetry requires a separate reporting endpoint for a live pitch dashboard. | **Very High:** Perfect for serverless/backend microservices without intermediary hops. |
| **DNS / Anycast Routing**                 | Dynamic GeoDNS (e.g., Route53 / NS1) resolving domains based on gCO₂eq weightings.                                                                                                         | **Zero latency overhead** at the HTTP layer.                                    | **Low:** DNS caching (TTL) is too sluggish for minute-by-minute flashlight demos.                      | **High:** For coarse-grained traffic balancing across data centers.                    |
| **Service Mesh / Sidecar**                | Envoy filter / Istio / Traefik plugin within a Kubernetes cluster handling egress traffic.                                                                                                 | Highly optimized within cloud clusters.                                         | **Low:** Setup complexity is too high for a hackathon.                                                 | **Industry Standard** for enterprise microservices.                                    |

> **Hackathon Strategy: Hybrid!**
>
> - **For the tactile live demo:** A local router (Node.js or Rust) communicating directly with the Raspberry Pis, LEDs, and the dashboard over the local event Wi-Fi/LAN.
> - **For the pitch & scalability story:** Showcase Edge architecture (Cloudflare Workers) & Client SDK as proven production paths.

---

## 4. Workload Matrix (Fit Profile)

Ideal for all **asynchronous, compute-heavy** workloads with a latency tolerance window $\ge 0.5\,\text{s}$:

| Workload Type                     | Example Scenario                                                 | Latency Tolerance   | Carbon Impact      |
| :-------------------------------- | :--------------------------------------------------------------- | :------------------ | :----------------- |
| **AI Inference (Batch / Agents)** | RAG embeddings, document analysis, auto-tagging, LLM agent loops | High (1s – 60s)     | **Extremely High** |
| **CMS Background Jobs**           | WordPress WebP conversion, thumbnail generation (50+ images)     | Medium (1s – 5s)    | **High**           |
| **CI/CD Pipelines**               | Automated testing, Docker image builds, code compilation         | Very High (minutes) | **High**           |
| **Media Transcoding**             | Video encoding into HLS/DASH streaming formats                   | High (minutes)      | **Extremely High** |
| **Data Crunching & Backup**       | Daily DB snapshots, ETL pipelines, analytics aggregations        | Very High (hours)   | **Medium to High** |

---

## 5. Hackathon MVP Scope

### Component 1: Core Router (Software)

- **Tech Stack Options:**
  - **Option A: TypeScript / Node.js (Fastify):** Rapid development velocity during the hackathon, native support for the official Model Context Protocol (MCP) TypeScript SDK, hassle-free OpenAI response streaming, and seamless WebSocket integration with the dashboard.
  - **Option B: Rust (Axum / Actix-web):** Maximum runtime efficiency, minimal memory footprint at the edge, reinforcing the "sovereign, efficient infrastructure" story with native bare-metal performance.
- **API Interfaces:**
  - **Universal HTTP Reverse Proxy:** Dynamic forwarding for any asynchronous REST workload (image processing, CI/CD pipelines, batch jobs) to the greenest target region.
  - **OpenAI-Compatible Gateway Profile:** Drop-in proxy (`/v1/chat/completions`) for seamless AI and agent integration.
  - **Optional MCP Server Interface:** For direct tool execution (`ecoroute://stats`).
- **Routing Engine & Carbon Data Model (Hybrid Approach):**
  - **Baseline Emissions Data:** Realistic grid data (e.g., via Electricity Maps API or fallback dataset: Norway ~25 g, France ~60 g, Germany ~380 g, Poland ~650 g CO₂eq/kWh).
  - **Dynamic Sensor Override ("Flashlight Effect"):** When an I2C sensor on a Raspberry Pi detects light exposure (simulating a local solar surplus), a mathematical scaling factor dynamically overrides that region's carbon intensity down to ~0 gCO₂eq/kWh.
  - **Geofencing & Constraint Validation:** Region filtering (`EU-only`) prior to ranking.
  - **Dispatch:** Request routes to the region with the lowest resulting score.

### Component 2: Live Dashboard (Visualization & Pitch Centerpiece)

- **Design Concept (Dual View):**
  - **Interactive Map / Region Cluster:** Visualizes global locations (e.g., Norway, Germany, Spain). When a Pi's solar panel is illuminated, the corresponding dashboard node lights up ("Solar Surge Detected"). Animated data flow particles show dynamic traffic rerouting in real time.
  - **Side-by-Side Live Comparison ("With vs. Without EcoRoute"):**
    - _Default Routing (Without EcoRoute):_ Fixed target region (e.g., local data center running heavily on coal) $\rightarrow$ cumulative emissions spike rapidly.
    - _EcoRoute (With Dynamic Routing):_ Green target region $\rightarrow$ computes real-time CO₂ savings in grams, displaying the _Green Routing Ratio_ alongside tangible equivalents (e.g., smartphone charges saved).
- **Tech Stack:** Modern web frontend (Vite / React / Tailwind) with a bidirectional WebSocket stream for sub-50ms latency.

### Component 3: Physical Hardware Demo (Showstopper Pitch)

- **Hardware:** 3–4 Raspberry Pis representing distributed cloud regions (e.g., Nordics, Central Europe, Southern Europe).
- **Sensors:** Mini solar panels paired with I2C light sensors mounted on 3D-printed dioramas.
- **Live Interaction:** The jury shines a flashlight onto a solar panel $\rightarrow$ EcoRoute detects local surplus clean energy $\rightarrow$ status LEDs on the targeted Pi light up synchronously with incoming request traffic $\rightarrow$ the dashboard visualizes the carbon arbitrage in real time.
- **Workload Execution on the Pis for the Demo:**
  - _Option 1 (Real Local Computation):_ A lightweight service on the Pi performing real image transcoding (WebP via Sharp/ImageMagick) or running a small local LLM (SmolLM/Qwen via Ollama) to produce real CPU load and flickering activity LEDs.
  - _Option 2 (Cloud Relay):_ Pi proxies calls to upstream APIs (OpenAI, Groq, Cloudflare Workers) while triggering hardware-level status LEDs.
  - _Option 3 (Mock Streamer):_ Synthetic responder for 100% stage reliability with realistic latency simulation.

### Component 4: Dynamic Traffic Generator (Hands-Free Pitch Mode)

- **Autonomous Workload Simulator:** A built-in traffic generator dispatches jobs in variable cycles (dynamic Poisson distribution rather than a static 1-second pulse).
- **Workload Bursts:** Simulates realistic burst traffic (e.g., "50 WebP thumbnail jobs at once" or "CI/CD test suite run") alternating with quieter periods.
- **Pitch Advantage:** Completely hands-free—the team doesn't have to touch a keyboard during the 3-minute pitch. As the judges move the flashlight, the continuous stream of traffic visibly shifts across the table and on the dashboard immediately.

---

## 6. Hackathon Submission Copy

### Pitch / Description

Compute-heavy cloud workloads (AI inference, CI/CD pipelines, CMS image processing, video transcoding) are growing exponentially, yet the renewable energy mix across data centers fluctuates constantly by region and weather.
"EcoRoute" is an intelligent, universal API gateway, load balancer, and reverse proxy that dynamically routes asynchronous HTTP requests and heavy jobs to the cloud region currently powered by the highest share of clean energy (lowest gCO₂eq/kWh).
Using real-time grid APIs (such as Electricity Maps), the proxy monitors carbon intensity worldwide.
Whenever a user, CMS, or AI agent dispatches an asynchronous task, it travels to wherever the sun is literally shining.
EcoRoute runs at low latency on the edge, enforces compliance boundaries (such as "EU-only"), and integrates seamlessly into any stack as open-source infrastructure.

### Goal / MVP by the End of the Hackathon

1. A working open-source router (Node.js/Rust) that accepts asynchronous HTTP workloads and OpenAI-compatible requests, dynamically dispatching them based on carbon intensity.
2. An interactive live dashboard visualizing avoided emissions and the "Green Routing Ratio" in real time.
3. The physical hardware demo: Simulating cloud regions using Raspberry Pis housed in 3D-printed dioramas. Equipped with mini solar panels and sensors, the router steers live request traffic in real time to whichever Pi the judges illuminate with a flashlight, confirmed by synchronized traffic LEDs.

### Long-Term Viability Post-Hackathon

EcoRoute will be published as open-source infrastructure (MIT License) alongside a standardized Model Context Protocol (MCP) server.
This enables developers to embed carbon-aware routing directly into environments like Claude Desktop or CMS plugins without altering underlying application architecture.
Post-hackathon, we plan to integrate dynamic cloud spot pricing APIs—transforming EcoRoute from an environmental initiative into a direct cost-saving engine, delivering tangible arbitrage on asynchronous workloads (batch processing, web scraping, CMS auto-tagging).

---

## 7. Integration Flexibility

EcoRoute is designed to integrate seamlessly and unobtrusively into existing infrastructure:

- **Zero-Config Drop-In:** Developers do not need to write new code or integrate complex SDKs. EcoRoute operates as a reverse proxy—existing applications simply point to a new base URL (e.g., swapping `api.openai.com` for `ecoroute.local/openai`, or updating a CMS worker endpoint to the EcoRoute gateway).
- **Architectural Extension:** If load balancers (such as Nginx, AWS ALB, or Traefik) are already in place, EcoRoute does not need to replace them. It can be slotted in gracefully as an "upstream router" behind the primary load balancer to intercept specific asynchronous workloads (e.g., matching the `/jobs` path). Alternatively, EcoRoute can be configured as a sidecar service that provides dynamic routing decisions back to the primary ingress layer. The existing core infrastructure—including SSL/TLS termination and DDoS protection—remains completely untouched.

---

## 8. Competitive Differentiation

Existing approaches either lock users into proprietary cloud ecosystems (e.g., Google Cloud Carbon-Aware Scheduling, Azure KEDA, Cloudflare Green Compute),
demand extensive code-level integration (such as the GSF Carbon Aware SDK), or merely measure emissions retrospectively after execution (CodeCarbon, Kepler).
EcoRoute differentiates itself through two core pillars:

- **Provider-Agnostic:** Dynamic routing balances workloads across heterogeneous cloud providers without vendor lock-in.
- **Zero-Config Drop-In:** Integration requires no proprietary SDKs, infrastructure overhauls, or application code rewrites.
  Operating as a reverse proxy, EcoRoute only requires swapping the base URL (e.g., within an existing load balancer or LLM client configuration).

---

## 9. Key Highlights & USPs

- **Universal Scope:** Not restricted to AI—optimizes any asynchronous compute workload (CMS image optimization, video transcoding, CI/CD builds, RAG indexing).
- **Track Fit 2027:** Directly targets _Sovereign, efficient infrastructure_ & _The agent-ready web_.
- **Zero-Overhead Adoption:** Drop-in proxy (simply update the endpoint / `baseURL` in tools & SDKs).
- **Economic Arbitrage (Green = Cheap):** Surplus renewable power (solar/wind spikes) drives down wholesale electricity prices and cloud spot instances. EcoRoute delivers bottom-line cost savings for CFOs on heavy batch workloads.
- **Dual Incentive:** Measurable environmental impact (CO₂ reduction) paired with natural financial efficiency.
- **Tactile Pitch Experience:** The interactive flashlight demo featuring Raspberry Pis, real solar cells, and synced LED indicators instantly sets the project apart from purely virtual software pitches.
- **Sovereign Infrastructure:** Strict division between non-negotiable compliance (`ALLOWED_REGIONS` for EU data sovereignty) and intelligent optimization within secure, compliant borders.
