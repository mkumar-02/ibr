# OpenG2P Integrated Beneficiary Registry (IBR)

Cross-programme view of who is enrolled where, and what benefits they have received.

OpenG2P already provides strong **Social Registries** (NSR, Farmer, Disability, and others) and **PBMS** for programme operations. What is missing is a system that answers:

- How many programmes is this beneficiary enrolled in?
- What benefits have they received, from which programme, and when?
- Is there unintended overlap or duplication across programmes?
- What is coverage across target groups and regions?

**IBR fills that gap.** It consolidates enrolment and benefit data across programmes so governments can coordinate, monitor, and analyse social protection delivery.

This product is designed to align with the [DCI Integrated Beneficiary Registry v1.0.0](https://standards.spdci.org/standards/dci-standards/wip-integrated-beneficiary-registry-v1.0.0) interoperability standards.

> **Status:** Documentation first. Implementation comes later.

## Documentation

| Document | Description |
|---|---|
| [Overview](docs/01-overview.md) | Problem, scope, and how IBR differs from Social Registry and PBMS |
| [Architecture](docs/02-architecture.md) | Shared system context and data flows |
| [Architecture options](docs/10-architecture-options.md) | **Decision brief** — A / A′ / B / B+ / C |
| [Domain model](docs/03-domain-model.md) | Registers, supporting tables, and DCI data objects |
| [DCI processes](docs/04-dci-processes.md) | PRS.IBR.01–04 use cases and OpenG2P mapping |
| [APIs](docs/05-apis.md) | DCI endpoints IBR exposes and consumes |
| [Integration](docs/06-integration.md) | PBMS publisher, registries as clients, partners |
| [Roadmap](docs/07-roadmap.md) | Phased delivery plan |
| [Open decisions](docs/08-open-decisions.md) | Choices to lock before implementation |
| [Glossary](docs/09-glossary.md) | Terms and acronyms |

**Next step for the team:** review [Architecture options](docs/10-architecture-options.md) and lock [Decision 0](docs/08-open-decisions.md#0-product-shape-where-ibr-lives).

## Spec references

- [DCI IBR v1.0.0 (WIP)](https://standards.spdci.org/standards/dci-standards/wip-integrated-beneficiary-registry-v1.0.0)
- [DCI IBR schemas](https://github.com/spdci/schemas/tree/main/extensions/ibr/v1)
- [DCI API standards](https://github.com/spdci/standards)
- [OpenG2P roadmap (IBR in 2026)](https://docs.openg2p.org/platform/roadmap)

## License

See [LICENSE](LICENSE).
