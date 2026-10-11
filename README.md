# EcoRoute — Carbon-Aware Workload Router

> **Follow-the-Sun Routing für asynchrone Compute-Workloads**  
> Routet rechenintensive Jobs (KI-Inferenz, CI/CD, CMS-Image-Processing, Video-Transcoding) dynamisch in Regionen mit maximalem erneuerbaren Stromüberschuss (niedrigste gCO₂eq/kWh).

---

## 1. Problem & Pain Point

- **Exponentielles Compute-Wachstum:** KI-Inferenz (LLMs, RAG, autonome Agenten), CI/CD und Background-Jobs fressen gigantische Strommengen.
- **Fluktuierender Strommix:** Der Anteil erneuerbarer Energien in Rechenzentren schwankt je nach Sonnenstand, Wind und Region extrem.
- **Inflexible Infrastruktur:** Bisherige Architekturen routen statisch auf feste Server/Regionen, unabhängig vom aktuellen Energiemix.
- **Falsche Priorisierung:** Asynchrone Jobs (Latenztoleranz: 0,5s bis Minuten) werden unnötig mit Kohlestrom verarbeitet.

---

## 2. Die Lösung: EcoRoute

Ein intelligenter, universeller **Reverse Proxy & Gateway**, der asynchrone HTTP-Requests und Background-Jobs analysiert und anhand von Echtzeit-Emissionsdaten dorthin dispatcht, wo im Moment sprichwörtlich die Sonne scheint.

```
                  ┌───────────────────────────────┐
                  │        Incoming Request       │
                  │ (LLM Call, Image Job, CI/CD)  │
                  └───────────────┬───────────────┘
                                  │
                                  ▼
                  ┌───────────────────────────────┐
                  │           EcoRoute            │
                  │   - Carbon Aware Engine       │
                  │   - Compliance Filter (EU-only)│
                  │   - Universeller HTTP / OpenAI │
                  └───────┬───────────────┬───────┘
                          │               │
     Lowest gCO2/kWh      │               │  Fallback / Low Green
   (High Solar/Wind)      ▼               ▼
                 ┌─────────────────┐    ┌─────────────────┐
                 │ Cloud Region A  │    │ Cloud Region B  │
                 │ (z.B. Norwegen) │    │ (z.B. Frankfurt)│
                 │  ~25 gCO2/kWh   │    │  ~380 gCO2/kWh  │
                 └─────────────────┘    └─────────────────┘
```

### Kernfunktionen & Integration

1. **Universeller HTTP Reverse Proxy & Job-Dispatcher:**
   - **Beliebige Workloads:** Routet beliebige asynchrone HTTP-Requests (z. B. `POST /api/v1/jobs/image-transcode`, `POST /ci/runner`, `POST /video/convert`) transparent an Worker in der grünsten Region weiter.
   - **Zero-Code OpenAI-Profil:** Für KI-Workloads fungiert EcoRoute als Drop-In Base-URL-Replacement für beliebige OpenAI-kompatible Clients.
2. **Dynamic Carbon Routing:** Fragt Grid-Daten (z. B. Electricity Maps / WattTime) ab und ermittelt die Region mit der geringsten Kohlenstoffintensität (`gCO2eq/kWh`).
3. **Compliance, Sovereign Infrastructure & Geofencing:**
   - **Self-Hosted Enterprise (Infrastruktur-Perimeter):** Unternehmen konfigurieren in `config.yaml` / ENV nur freigegebene Zielregionen (z. B. `ALLOWED_REGIONS=["no-south", "de-central", "fr-north"]`). Workloads verlassen niemals die EU/DSGVO-Zone; EcoRoute optimiert den Strommix rein _innerhalb_ dieses souveänen Perimeters.
   - **Multi-Tenant / Edge-Ebene:** Dynamische Steuerung pro Request via Subdomains (z. B. `eu.ecoroute.io`), Header (`X-EcoRoute-Zone: EU-only`) oder URL-Pfad.
4. **Live-Telemetry:** Erfasst eingesparte CO₂-Emissionen und berechnet die _Green Routing Ratio_.
5. **Optional: MCP Server Add-on:** Bietet Agenten (z. B. Claude Desktop) optionale Resources (`ecoroute://stats`), um den eigenen CO₂-Footprint transparent abzufragen.

---

## 3. Architektur- & Deployment-Modelle im Vergleich

| Modell                                  | Funktionsweise                                                                                                                                                               | Latenz & Ausfallsicherheit                                                              | Hackathon-Eignung                                                                                  | Production-Eignung                                                       |
| :-------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------- |
| **Edge Computing (Empfohlen für Prod)** | Globales Edge-Skript (Cloudflare Workers, Fastly Compute). Request trifft physisch nächsten Edge-Point, prüft gecachten Strom-Index und leitet direkt an Ziel-Region weiter. | **Kein SPOF, minimale Latenz** (kein zentraler Flaschenhals).                           | **Sehr hoch:** Wenige Zeilen Code, blitzschnell deployt.                                           | **Höchste Skalierbarkeit & Ausfallsicherheit.**                          |
| **Klassischer Reverse Proxy**           | Lokaler/Zentraler Server (Node.js/Fastify, Rust/Axum oder Go) als Gateway/Middleware.                                                                                        | **SPOF-Gefahr & Doppelter Hop** in globaler Prod, aber vernachlässigbar im lokalen LAN. | **Optimal für den Demo-Tisch:** Steuert direkt Pis, LEDs & Dashboard im lokalen Subnetz.           | Eher für On-Premise oder Gateway-Cluster.                                |
| **Client-Side SDK**                     | Library (`npm`/`pip`), die regelmäßig einen schlanken Carbon-Index (JSON) cacht und Requests clientseitig direkt an die grünste Region schickt.                              | **Kein Proxy-Hop**, maximale Dezentralisierung.                                         | **Mittel:** Dezentrale Telemetrie für ein Live-Pitch-Dashboard erfordert extra Reporting-Endpunkt. | **Sehr hoch:** Perfekt für Serverless/Backend-Apps ohne Zwischenstation. |
| **DNS / Anycast Routing**               | Dynamisches GeoDNS (z. B. Route53 / NS1), das Domain-Auflösung basierend auf gCO₂eq-Gewichtungen vornimmt.                                                                   | **Null Latenz-Overhead** auf HTTP-Ebene.                                                | **Gering:** DNS-Caching (TTL) ist zu träge für minütliche Solar-Taschenlampen-Demos.               | **Hoch:** Für grobgranulare Lastverteilung zwischen RZs.                 |
| **Service Mesh / Sidecar**              | Envoy-Filter / Istio / Traefik-Plugin im Kubernetes-Cluster für Egress-Traffic.                                                                                              | Hochgradig optimiert innerhalb von Cloud-Clustern.                                      | **Gering:** Zu hohe Setup-Komplexität für einen Hackathon.                                         | **Industrie-Standard** für Enterprise-Microservices.                     |

> **Strategie für den Hackathon:** Hybrid!
>
> - **Für die haptische Live-Demo:** Ein lokaler Router (Node.js oder Rust), der direkt mit den Pis, den LEDs und dem Dashboard im Messe-WLAN/LAN spricht.
> - **Für den Pitch & Skalierbarkeits-Nachweis:** Edge-Architektur (Cloudflare Worker) & Client-SDK als bewiesene Architekturpfade präsentieren.

---

## 4. Workload-Matrix (Eignungsprofil)

Ideal für alle **asynchronen, rechenintensiven** Workloads mit Latenzfenster $\ge 0,5\,\text{s}$:

| Workload-Typ                     | Beispiel-Szenario                                                | Latenztoleranz      | Carbon-Impact       |
| :------------------------------- | :--------------------------------------------------------------- | :------------------ | :------------------ |
| **KI-Inferenz (Batch / Agents)** | RAG-Embedding, Dokument-Analysen, Auto-Tagging, LLM-Agent-Loops  | Hoch (1s – 60s)     | **Extrem hoch**     |
| **CMS Background Jobs**          | WordPress WebP-Konvertierung, Thumbnail-Generierung (50+ Bilder) | Mittel (1s – 5s)    | **Hoch**            |
| **CI/CD Pipelines**              | Automated Testing, Docker Image Builds, Code Compilation         | Sehr hoch (Minuten) | **Hoch**            |
| **Media Transcoding**            | Video-Encoding in HLS/DASH Streaming-Formate                     | Hoch (Minuten)      | **Extrem hoch**     |
| **Data Crunching & Backup**      | Tägliche DB-Snapshots, ETL-Pipelines, Analytics-Aggregation      | Sehr hoch (Stunden) | **Mittel bis Hoch** |

---

## 5. Hackathon MVP-Scope

### Komponente 1: Core Router (Software)

- **Tech-Stack Optionen:**
  - **Option A: TypeScript / Node.js (Fastify):** Schnelle Entwicklungsgeschwindigkeit im Hackathon, nativer Support für das offizielle Model Context Protocol (MCP) TypeScript SDK, unkompliziertes Streaming von OpenAI-Responses & nahtlose WebSocket-Anbindung ans Dashboard.
  - **Option B: Rust (Axum / Actix-web):** Maximale Laufzeit-Effizienz, minimaler Memory-Footprint am Edge, unterstreicht die "Sovereign, efficient infrastructure"-Story mit nativer Performance.
- **API-Schnittstellen:**
  - **Universeller HTTP Reverse Proxy:** Dynamisches Forwarding für beliebige asynchrone REST-Workloads (Image-Processing, CI/CD-Pipelines, Batch-Jobs) an die grünste Ziel-Region.
  - **OpenAI-kompatibles Gateway Profil:** Drop-In Proxy (`/v1/chat/completions`) für nahtlose KI- und Agenten-Einbindung.
  - **Optionales MCP Server Interface:** Für direkte Tool-Aufrufe (`ecoroute://stats`).
- **Routing-Engine & Carbon-Datenmodell (Hybrid-Ansatz):**
  - **Basis-Emissionsdaten:** Realistische Grid-Daten (z. B. via Electricity Maps API oder Fallback-Dataset für Norwegen ~25 g, Frankreich ~60 g, Deutschland ~380 g, Polen ~650 g CO₂eq/kWh).
  - **Dynamischer Sensor-Override (Taschenlampen-Effekt):** Detektiert der I2C-Sensor an einem Pi Licht-Einstrahlung (Solarüberschuss), überschreibt ein mathematischer Faktor die Intensität dieser Region dynamisch bis auf ~0 gCO₂eq/kWh.
  - **Geofencing & Constraint-Check:** Filterung nach Regionen (`EU-only`) vor dem Ranking.
  - **Dispatch:** Request geht an die Region mit dem niedrigsten resultierenden Score.

### Komponente 2: Live-Dashboard (Visualisierung & Pitch-Zentrum)

- **Design-Konzept (Dual-View):**
  - **Interaktive Map / Region-Cluster:** Visualisiert die globalen Standorte (z. B. Norwegen, Deutschland, Spanien). Wenn ein Pi-Solarpanel angestrahlt wird, leuchtet der entsprechende Node im Dashboard auf ("Solar Surge Detected"). Animierte Datenfluss-Partikel zeigen das dynamische Routing in Echtzeit.
  - **Side-by-Side Live-Vergleich ("Mit vs. Ohne EcoRoute"):**
    - _Default-Routing (Ohne EcoRoute):_ Feste Zielregion (z. B. lokales RZ mit hohem Kohlestrom-Anteil) $\rightarrow$ kumulierte Emissionen steigen schnell.
    - _EcoRoute (Mit dynamischem Routing):_ Grüne Zielregion $\rightarrow$ berechnet das direkte CO₂-Delta in Gramm und visualisiert die _Green Routing Ratio_ sowie greifbare Äquivalente (z. B. eingesparte Smartphone-Akkuladungen).
- **Tech-Stack:** Modernes Web-Frontend (Vite / React / Tailwind) mit bidirektionalem WebSocket-Stream für Latenzen < 50ms.

### Komponente 3: Physische Demo (Showstopper Pitch)

- **Hardware:** 3–4 Raspberry Pis repräsentieren globale Rechenzentren (z. B. Nordics, Mitteleuropa, Südeuropa).
- **Sensorik:** Mini-Solarpanels mit I2C-Lichtsensoren auf 3D-gedruckten Dioramen.
- **Live-Interaktion:** Die Jury strahlt ein Solarpanel mit einer Taschenlampe an $\rightarrow$ EcoRoute detektiert lokalen Energieüberschuss $\rightarrow$ Status-LEDs am getroffenen Pi leuchten synchron zum Request-Traffic $\rightarrow$ Dashboard visualisiert CO₂-Arbitrage in Echtzeit.
- **Workload-Ausführung auf den Pis für die Demo:**
  - _Option 1 (Echte lokale Berechnung):_ Kleiner Service auf dem Pi für echtes Bild-Transcoding (WebP via Sharp/ImageMagick) oder lokales Mini-LLM (SmolLM/Qwen via Ollama) für echte CPU-Last und flackernde LEDs.
  - _Option 2 (Cloud Relay):_ Pi leitet an Upstream-APIs (OpenAI, Groq, Cloud-Workers) weiter und triggert hardwarenah die LEDs.
  - _Option 3 (Mock-Streamer):_ Synthetischer Responder für 100% Bühnensicherheit mit realistischer Rechenzeit-Simulation.

### Komponente 4: Dynamischer Traffic-Generator (Hands-Free Pitch Mode)

- **Autonomer Workload-Simulator:** Ein integrierter Traffic-Generator feuert Jobs in variablen Zyklen (dynamische Poisson-Verteilung statt monotonem Sekundentakt).
- **Workload-Spitzen:** Simuliert realistische Bursts (z. B. "50 WebP Thumbnail Jobs auf einmal" oder "CI/CD Test Suite Run") wechselnd mit ruhigeren Phasen.
- **Pitch-Vorteil:** Völlig "hands-free" – das Team muss während der 3-minütigen Präsentation keine Tasten drücken. Wenn die Jury die Taschenlampe bewegt, reagiert der ununterbrochene Traffic-Strom sofort sichtbar auf dem Tisch und im Dashboard.

---

## 6. Submission Textbausteine

### Pitch / Description

Rechenintensive Cloud-Workloads (KI-Inferenz, CI/CD-Pipelines, CMS-Image-Jobs, Transcoding) wachsen exponentiell, doch der Strommix der Rechenzentren fluktuiert je nach Region und Wetter.
"EcoRoute" ist ein intelligenter, universeller API Gateway, Loadbalancer bzw. Reverse Proxy, der asynchrone HTTP-Requests und rechenintensive Jobs dynamisch in jene Cloud-Region routet,
die aktuell den höchsten Anteil an erneuerbaren Energien aufweist (geringste gCO2eq/kWh).
Über APIs (wie Electricity Maps) liest der Proxy die Kohlenstoffintensität globaler Standorte aus.
Sendet ein Nutzer, CMS oder KI-Agent einen asynchronen Job, wandert dieser dorthin, wo wortwörtlich "gerade die Sonne scheint".
EcoRoute operiert latenzarm am Edge, unterstützt Compliance-Zonen (z.B. "Nur EU") und integriert sich nahtlos als Open-Source-Infrastruktur.

### Goal / MVP by end of Hackathon

1. Lauffähiger Open-Source-Router (Node.js/Rust), der asynchrone HTTP-Workloads und OpenAI-kompatible Requests annimmt und dynamisch nach CO2-Werten weiterleitet.
2. Ein interaktives Dashboard, das die eingesparten Emissionen und die "Green Routing Ratio" in Echtzeit visualisiert.
3. Die physische Demo: Wir simulieren die Cloud-Regionen mit Raspberry Pis in 3D-gedruckten Dioramen. Ausgestattet mit Mini-Solarpanels und Sensoren, verlagert der Router den echten Request-Traffic live auf den Pi, dessen Solarpanel gerade von der Jury (mit einer Taschenlampe) angestrahlt wird, sichtbar gemacht durch Traffic-LEDs.

### Long-term Viability post-Hackathon

EcoRoute wird als Open-Source-Infrastruktur (MIT License) und als standardisierter Model Context Protocol (MCP) Server veröffentlicht.
So können Entwickler nachhaltiges Routing direkt in Tools wie Claude Desktop oder WordPress-Plugins integrieren, ohne ihre Architektur zu ändern.
Nach dem Event planen wir, die Integration von dynamischen Cloud-Pricing-APIs voranzutreiben, sodass EcoRoute nicht nur das "grüne Image" fördert,
sondern Unternehmen echte Arbitrage-Kostenvorteile bei asynchronen Workloads (wie Batch-Processing, Scraping, Auto-Tagging im CMS) bietet.

---

## 7. Hard Facts & USPs

- **Universeller Scope:** Nicht auf KI limitiert – optimiert jeden asynchronen HTTP-Compute-Workload (CMS Image-Processing, Video-Transcoding, CI/CD-Builds, RAG-Indexierung).
- **Track Fit 2027:** Adressiert direkt _Sovereign, efficient infrastructure_ & _The agent-ready web_.
- **Zero Overhead Adoption:** Drop-in Proxy (Umstellen der Ziel-URL / `baseURL` in Tools & SDKs genügt).
- **Economic Arbitrage (Green = Cheap):** Erneuerbarer Stromüberfluss (Solar/Wind-Peaks) drückt Börsenstrompreise und Spot-Instanzen-Kosten. EcoRoute liefert handfeste Kostenersparnis für CFOs bei rechenintensiven Batch-Workloads.
- **Dual-Incentive:** Ökologischer Impact (CO₂-Reduktion) gekoppelt mit nativer Kosteneffizienz.
- **Haptisches Pitch-Erlebnis:** Die Taschenlampen-Demo mit Raspberry Pis, echten Solarpanels und synchron leuchtenden LEDs hebt das Projekt sofort von rein virtuellen Software-Pitches ab.
- **Sovereign Infrastructure:** Harte Trennung zwischen unverhandelbarer Compliance (`ALLOWED_REGIONS` für EU-Souveränität) und intelligenter Optimierung innerhalb der sicheren Grenzen.
