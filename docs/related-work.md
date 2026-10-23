# Related Work

Carbon-aware placement is an active research area and there are several early open-source projects. EcoRoute differs mainly in its deployment model:

| Project                                                                           | What it does                                              | Difference to EcoRoute                                            |
| --------------------------------------------------------------------------------- | --------------------------------------------------------- | ----------------------------------------------------------------- |
| [GSF Carbon Aware SDK][carbon-aware-sdk]                                          | Library / Web API that returns the best time and location | Gives a recommendation; does not route or queue                   |
| [Cloudflare Green Compute][cf-green-compute]                                      | Runs Cron Triggers only in renewable-powered PoPs         | Cloudflare-only, cron only, based on annual renewable procurement |
| [Carbon-aware KEDA operator][keda-carbon], [Karmada-based schedulers][k8s-survey] | Kubernetes scaling / placement by carbon                  | Requires Kubernetes; not request-level                            |
| [CASPER][casper] (research)                                                       | Weights Traefik across regions by carbon and SLO          | Research prototype, not packaged                                  |
| [CarbonRoute][carbonroute], [CarbonServe][carbonserve] (GitHub)                   | Route LLM requests between cloud regions                  | LLM-only, hyperscaler-oriented, early stage                       |
| [CodeCarbon][codecarbon], [Kepler][kepler]                                        | Measure emissions                                         | Measurement only, no shifting                                     |

EcoRoute's niche: an open-source, drop-in router that works at request level, independent of any cloud vendor, with sovereignty constraints and optional time-shifting.

Research on 123 cloud regions ([Sukprasert et al., EuroSys 2024](https://arxiv.org/abs/2306.06502)) found that simple policies capture most of the achievable savings.

[carbon-aware-sdk]: https://github.com/Green-Software-Foundation/carbon-aware-sdk
[cf-green-compute]: https://blog.cloudflare.com/announcing-green-compute/
[keda-carbon]: https://github.com/Azure/carbon-aware-keda-operator
[k8s-survey]: https://arxiv.org/abs/2508.05949
[casper]: https://arxiv.org/abs/2403.14792
[carbonroute]: https://github.com/dheerajeshwar32/carbonroute-api
[carbonserve]: https://github.com/RavaniRoshan/carbonserve
[codecarbon]: https://github.com/mlco2/codecarbon
[kepler]: https://github.com/sustainable-computing-io/kepler
