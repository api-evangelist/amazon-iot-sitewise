# Amazon IoT SiteWise (amazon-iot-sitewise)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

AWS IoT SiteWise is a managed service that makes it easy to collect, store, organize, and monitor industrial data at scale. It provides tools to create asset models representing your industrial operations and analyze equipment performance across your facilities.

**URL:** [Visit APIs.json URL](https://raw.githubusercontent.com/api-evangelist/amazon-iot-sitewise/refs/heads/main/apis.yml)

**Run:** [Capabilities Using Naftiko](https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=company-api-evangelist&utm_content=repo)

## Tags:

 - AWS, Asset Management, Industrial IoT, IoT, Time Series Data

## Timestamps

- **Created:** 2026-03-16
- **Modified:** 2026-04-19

## APIs

### AWS IoT SiteWise API
The AWS IoT SiteWise API provides access to asset model management, asset data ingestion, time-series data queries, portals, and dashboards for industrial IoT monitoring.

**Human URL:** [https://aws.amazon.com/iot-sitewise/](https://aws.amazon.com/iot-sitewise/)

#### Tags:

 - Asset Management, Industrial IoT, IoT, Time Series

#### Properties

- [Documentation](https://docs.aws.amazon.com/iot-sitewise/latest/APIReference/)
- [OpenAPI](openapi/amazon-iot-sitewise-openapi-original.yml)
- [GettingStarted](https://docs.aws.amazon.com/iot-sitewise/latest/userguide/getting-started.html)
- [Pricing](https://aws.amazon.com/iot-sitewise/pricing/)
- [FAQ](https://aws.amazon.com/iot-sitewise/faqs/)

## Common Properties

- [Portal](https://aws.amazon.com/iot-sitewise/)
- [Website](https://aws.amazon.com/iot-sitewise/)
- [Documentation](https://docs.aws.amazon.com/iot-sitewise/)
- [TermsOfService](https://aws.amazon.com/service-terms/)
- [PrivacyPolicy](https://aws.amazon.com/privacy/)
- [Support](https://aws.amazon.com/premiumsupport/)
- [Blog](https://aws.amazon.com/blogs/iot/tag/aws-iot-sitewise/)
- [GitHubOrganization](https://github.com/aws)
- [Console](https://console.aws.amazon.com/iotsitewise/)
- [SignUp](https://portal.aws.amazon.com/billing/signup)
- [Login](https://signin.aws.amazon.com/)
- [StatusPage](https://health.aws.amazon.com/health/status)
- [Contact](https://aws.amazon.com/contact-us/)

## Features

| Name | Description |
|------|-------------|
| Asset Modeling | Create hierarchical asset models that represent your industrial equipment and processes. |
| Time-Series Data Storage | Ingest and store industrial sensor data with automatic data quality classification. |
| SiteWise Monitor | Build no-code dashboards for industrial operations visualization. |
| Edge Processing | Process data locally at industrial sites using SiteWise Edge. |
| Computed Properties | Define metrics and transforms on asset data using built-in formula engine. |

## Use Cases

| Name | Description |
|------|-------------|
| Equipment Performance Monitoring | Track OEE and equipment health across multiple manufacturing facilities. |
| Energy Management | Monitor and optimize energy consumption across industrial sites. |
| Process Optimization | Analyze production line data to identify bottlenecks and inefficiencies. |

## Integrations

| Name | Description |
|------|-------------|
| AWS IoT Greengrass | Collects industrial data from OPC-UA, Modbus, and Ethernet/IP sources at the edge. |
| Amazon Kinesis | Streams asset property data for real-time analytics. |
| Amazon QuickSight | Visualizes SiteWise industrial data in business dashboards. |

## Artifacts

Machine-readable API specifications organized by format.

### OpenAPI

- [AWS IoT SiteWise API](openapi/amazon-iot-sitewise-openapi-original.yml)

### JSON Schema

200 schema files covering key resources and operations.

### JSON Structure

200 JSON Structure files converted from JSON Schema.

### JSON-LD

- [Amazon IoT SiteWise Context](json-ld/amazon-iot-sitewise-context.jsonld)

### Examples

200 example JSON files generated from schemas.

## Capabilities

Naftiko capabilities organized as shared per-API definitions composed into customer-facing workflows.

### Shared Per-API Definitions

- [AWS IoT SiteWise API](capabilities/shared/iot-sitewise.yaml) — operations for amazon iot sitewise management

### Workflow Capabilities

| Workflow | APIs Combined | Tools | Persona |
|----------|--------------|-------|---------|
| [Industrial Asset Management](capabilities/industrial-asset-management.yaml) | Amazon IoT SiteWise | 8 | OT Engineer, Data Analyst |

## Vocabulary

- [Amazon IoT SiteWise Vocabulary](vocabulary/amazon-iot-sitewise-vocabulary.yaml) — Unified taxonomy mapping resources, actions, workflows, and personas

## Rules

- [Amazon IoT SiteWise Spectral Rules](rules/amazon-iot-sitewise-spectral-rules.yml) — 14 rules across 6 categories enforcing Amazon IoT SiteWise API conventions

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
