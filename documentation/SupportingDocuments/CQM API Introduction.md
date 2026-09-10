# CAMARA CQM APIs — Understanding Connectivity Quality Management

## 1. Executive Summary

Most applications consume mobile connectivity on a best-effort basis: the network tries to deliver the best possible performance at a given moment without an application explicitly requesting a defined connectivity quality. In practice, that performance may vary depending on factors such as network deployment, coverage conditions, location, mobility and current network usage.

For some usage contexts, an API consumer — such as an Application Service Provider (ASP) — may need defined connectivity quality for a specific service, device, time or location. The CAMARA Connectivity Quality Management (CQM) portfolio addresses these needs through a set of specialised APIs.

This document explains differences of these APIs at product and concept level; it is not a normative specification, a Connectivity Service Provider (CSP) implementation roadmap or a strict API selection guide, and actual API availability and supported capabilities vary by CSP and market.

The APIs within the **Connectivity Quality Management** domain are:

| API Name | API Family | Description | Preview Only |
| --- | --- | --- | --- |
| `qos-profiles` | Quality On Demand | Discovery API that exposes the catalogue of QoS profiles (throughput, latency, priority) a CSP offers, including which profiles apply to a given device, so the right profile can be selected and referenced by the other CQM APIs. |  |
| `quality-on-demand` | Quality On Demand | Requests that a QoD Session (with a QoS profile) be created and applied immediately, for a bounded duration, to one or more application data flows tied to a device — the "apply now, for a limited time" tool. |  |
| `qos-provisioning` | Quality On Demand | Assigns a QoS profile to a device on a persistent basis, applied automatically whenever that device connects, until the assignment is explicitly removed. |  |
| `qos-booking` | QoS Booking | Reserves one QoS profile for one device over a defined time window (immediate or future) and service area, giving confidence — pending CSP confirmation — that the requested quality will be usable. |  |
| `qos-booking-and-assignment` | QoS Booking | Books connectivity for a time window, profile and service area while keeping device assignment separate, so devices can be added, removed, or swapped over the booking's lifetime. |  |
| `dedicated-network` | Dedicated Networks | Reserves a connectivity environment — potentially spanning multiple QoS profiles — for a time window and service area, without binding devices at booking time; device access is managed separately. |  |
| `dedicated-network-accesses` | Dedicated Networks | Companion API to `dedicated-network` that grants, changes, or revokes which devices may use a reserved connectivity environment. |  |
| `dedicated-network-profiles` | Dedicated Networks | Discovery API for the catalogue of network profiles (predefined Dedicated Networks configurations) a CSP offers. | |
| `dedicated-network-areas` | Dedicated Networks | Discovery API that exposes eligible service areas and the QoS/network profiles supported within each, so a consumer can select a valid area before reserving. | Yes |

This document focuses on the CQM APIs listed above. Other CAMARA APIs in the broader [Communication Quality](https://camaraproject.org/api-overview/)  area are related to connectivity-quality use cases, but are currently outside the main scope of this document and are briefly referenced in Appendix B for context.

## 2. Introduction — Why Connectivity Quality Management Matters

Mobile connectivity is typically delivered on a **best-effort** basis. The network aims to deliver the best performance at a given moment, but the actual connectivity quality experience can fluctuate depending on factors such as coverage conditions, geographical location, device movement and current network usage.

For many applications, this level of service is perfectly adequate. Other applications require a more consistent and reliable connectivity quality (particularly in terms of throughput and availability) under well-defined conditions. Examples include a live video uplink during a media event, point-of-sale terminals at a temporary store, several media production devices operating at one venue, or enterprise connectivity required for a defined period and location.

For these applications, connectivity quality that is rejected, unavailable, or performing below expectations can have a direct impact on business outcomes: the API consumer may be unable to launch its operations, sustain them throughout the required period, or achieve the intended business objectives.

Connectivity Quality Management exposes API-based tools that allow API consumers to discover, request, configure, reserve or manage connectivity quality in a standardised way.

API consumers work with simplified abstractions of the network connectivity such as QoS profiles, QoS sessions, connectivity bookings, service areas, device assignments and dedicated connectivity environments.

The mobile network can therefore be seen as a black box which provides sufficient information and offers relevant controls. The APIs expose the information that API consumers need: available capabilities, operations to request or manage them, and status information describing the outcome of those operations.

A successfully processed API request does not always mean that the requested connectivity quality is available. Depending on the API, the CSP offering, the device location and potentially the current network utilization, the resulting resource may indicate that the requested service is available, pending or unavailable. A later status change may also indicate that the requested connectivity quality cannot or no longer be fulfilled. These service-availability outcomes are distinct from technical API errors.

This document is intended for API consumers, developers, technical product people and non-telco readers who need to understand the purpose of each CQM API without examining endpoint-level specifications or internal mobile network architecture.

## 3. Core CQM Concepts in Plain Language

The following concepts describe what an API consumer sees and works with, rather than how the operator implements them internally.

| Concept | Plain-language meaning | Why it matters in CQM |
| --- | --- | --- |
| Connectivity quality | The level of connectivity experience needed for a given usage context. It may be described through connectivity-performance characteristics such as throughput, latency, priority or jitter. | It is the API consumer-facing concept that explains what the consumer wants to obtain when best-effort connectivity is not sufficient. In CQM, such connectivity-performance characteristics are typically defined as part of a `qos-profile`. |
| QoS profile | A reusable description of offered connectivity-performance characteristics. | It is the common reference used across all CQM APIs when requesting, assigning or reserving defined connectivity quality. The timing of its activation — immediately or at a future point — depends on which API family is used. |
| Discovery of capabilities | The ability to retrieve information about the offered QoS profiles, network profiles or eligible service areas before requesting or reserving them. | It allows the CSP to make its available capabilities visible to API consumers, who can then use them without needing to know internal network topology. |
| QoS session | A time-bounded instantiation of a QoS profile to one or more application flows associated with a device. | It explains temporary QoS treatment, either requested On-Demand (using the `quality-on-demand` API) or in-future (using one of the reservation APIs). |
| Provisioned QoS assignment | A QoS profile associated with a device and applied to all its traffic whenever the device connects to the access network, until the assignment is revoked. | It explains `qos-provisioning`: persistent device-level configuration (like a _premium subscription_) rather than a time-bounded session requested at the moment of use. |
| Connectivity quality booking | Provides confidence that the QoS Profile is usable by the requesting API consumer for a given time window and, where the API allows, a defined service area. This confidence is strongest for a future-dated booking, where the CSP has lead time to confirm availability ahead of use; a booking requested for immediate start has little or no such lead time and behaves closer to an on-demand request. | It explains `qos-booking`, `qos-booking-and-assignment` and `dedicated-network`. Through the booking, the API consumer gets confidence that the QoS Profile is usable. The CSP decides whether it can be confirmed. The minimum time between a reservation request and its confirmation/activation is defined by the CSP. Device assignment may be included in the booking or managed separately, depending on the API. |
| Service area | The geographic area where requested or booked connectivity quality is expected to apply. | It gives the API consumer an understandable geographic abstraction while keeping operator-internal topology hidden. |
| Device assignment | The act of linking one or more devices for the actual usage of the connectivity quality capabilities. | It explains the distinction between booking connectivity quality and determining which devices may use it. Depending on the API, device assignment can be done in the same step as the booking, or managed separately — first booking connectivity quality without binding devices, then assigning them later. |

The minimum time between a connectivity quality booking request and its confirmation or activation is defined by the CSP. In practice, this may reflect the time needed to assess whether the requested connectivity quality can be supported for the given conditions, and the time needed to prepare the network for activation. For an immediate-start booking, this minimum time may be effectively zero, in which case the confidence benefit over an on-demand request is limited. API consumers should account for these aspects when planning their connectivity quality needs. The API documentation of the network provider will clarify details.

The key differences between the three **connectivity quality booking APIs** are around capabilities like targeted devices and targeted QoS profiles:

| API | # Devices | # QoS Profile |
| ---- | ---- | ---- |
| QOS Booking | One Device | One QoS Profile |
| QOS Booking and Assignment | One or more Devices | One QoS Profile |
| Dedicated Networks | One or more Devices | One or more QoS Profiles |

## 4. Common Journey: Live Event Connectivity

To make the portfolio easier to understand, this document uses one illustrative journey: **connectivity quality around a live event**.

A live-event context can create different connectivity needs depending on the production setup. These needs do not all apply to every event, and they are not steps in a mandatory sequence. The examples below illustrate potentially applicable CQM tools under different usage conditions. They are not exhaustive. Furthermore, listing a tool as illustrative does not imply it is the only way to fulfil the need — depending on the CSP implementation, an equivalent outcome may be reached through other mechanisms.

![CQM Live Event Journey](../images/section4_live_event_journey.svg)
*Figure 1 — Illustrative CQM API usage across a live event timeline. This diagram is not a strict API selection guide; applicable tools depend on CSP offering and market context.*

| Illustrative live-event situation | Concrete API consumer need | Illustrative applicable CQM tool(s) | What matters to the API consumer |
| --- | --- | --- | --- |
| Before the event | Planning of needed connectivity quality, i.e. which QoS profiles, network profiles or eligible service areas the CSP exposes | `qos-profiles`, `dedicated-network-profiles` and, where exposed, service-area discovery | Understand and decide which connectivity options can be referenced before requesting or reserving them, reducing the risk for errors or rejections. |
| A reporter or contributor starts an unplanned live uplink | Request defined connectivity quality immediately, or obtain immediate / near-term visibility that the needed QoS profile can be supported at the current device location | `quality-on-demand` for an immediate time-bounded QoS session bound to specific application data flows; `qos-provisioning` if the device already has a persistent QoS assignment in place; `qos-booking` or `qos-booking-and-assignment` where the CSP supports an immediate booking, followed by QoS session establishment; `dedicated-network` by creating a network with an available profile and granting device access immediately, followed by QoS session establishment | Obtain a QoS session or a confirmed immediate booking for the selected QoS profile under the current conditions. The request may not succeed, or may later be withdrawn, if the network cannot fulfil the requested quality. |
| A broadcaster regularly uses the same field equipment across productions | Keep a QoS profile associated with a device whenever it connects to the access network | `qos-provisioning` for a persistent device-level QoS assignment active whenever the device connects; `dedicated-network` with `dedicated-network-accesses` where the broadcaster operates within a dedicated network environment; `qos-booking` or `qos-booking-and-assignment` where connectivity is pre-booked per production rather than persistently assigned; `quality-on-demand` where a new time-bounded session per use is acceptable | Apply a persistent device-level QoS assignment without creating a new time-bounded session for every use. Persistence of the assignment is not a universal guarantee that identical measured performance will be available at every time and location. |
| A single-camera contribution is scheduled in advance at a known venue | Planning of the needed connectivity quality for the one device during a future time window and service area | `qos-booking`, `qos-booking-and-assignment` or `dedicated-network` to book the QoS profile for a specific time window and service area in advance; `quality-on-demand`, where supported by the CSP offering, to establish the QoS session on the day once the booking is confirmed | Confidence that the connectivity quality for a known time and place is available. |
| A small outside broadcast uses several devices that may be replaced during the event | Reserve connectivity first and assign or re-assign devices later | `qos-booking-and-assignment` to book connectivity for a time window and service area and manage device assignment or re-assignment within the booking; `dedicated-network` with `dedicated-network-accesses` to book connectivity for a time window and service area and grant or revoke device access as devices are replaced; `quality-on-demand`, where supported by the CSP offering, to establish the QoS session between each assigned or re-assigned device and the application server | Decouple the reservation from the final device list. The relevant tool depends on whether the scenario requires device assignment only or the management of multiple devices and/or QoS profiles. |
| A complex production uses live video, intercom, preview feeds and operational traffic in the same venue | Use a reserved connectivity environment where several QoS profiles and device-access rules may be available | `dedicated-network` with `dedicated-network-profiles` and `dedicated-network-accesses` to reserve a multi-profile connectivity environment and manage device access; `dedicated-network-areas` where service area management within the venue is needed; `quality-on-demand` to activate QoS sessions per device and application flow within the environment | Manage a reserved connectivity environment with multiple QoS profiles and related conditions. Where supported by the CSP offering, `quality-on-demand` may activate QoS sessions dynamically within that environment. |

The same logic can be transposed to adjacent scenarios such as a festival, a pop-up store, temporary point-of-sale terminals or short-term enterprise connectivity at a site.

This journey is illustrative. It does **not** imply that every scenario uses every API, that the APIs must be invoked together, or that there is a universal decision tree for selecting an API.

## 5. Scope and Portfolio Grouping

Table: Overview of the CQM APIs

| API Name | API Family | Description | Preview Only |
| --- | --- | --- | --- |
| `qos-profiles` | Quality On Demand | Discovery API that exposes the catalogue of QoS profiles (throughput, latency, priority) a CSP offers, including which profiles apply to a given device, so the right profile can be selected and referenced by the other CQM APIs. |  |
| `quality-on-demand` | Quality On Demand | Requests that a QoD Session (with a QoS profile) be created and applied immediately, for a bounded duration, to one or more application data flows tied to a device — the "apply now, for a limited time" tool. |  |
| `qos-provisioning` | Quality On Demand | Assigns a QoS profile to a device on a persistent basis, applied automatically whenever that device connects, until the assignment is explicitly removed. |  |
| `qos-booking` | QoS Booking | Reserves one QoS profile for one device over a defined time window (immediate or future) and service area, giving confidence — pending CSP confirmation — that the requested quality will be usable. |  |
| `qos-booking-and-assignment` | QoS Booking | Books connectivity for a time window, profile and service area while keeping device assignment separate, so devices can be added, removed, or swapped over the booking's lifetime. |  |
| `dedicated-network` | Dedicated Networks | Reserves a connectivity environment — potentially spanning multiple QoS profiles — for a time window and service area, without binding devices at booking time; device access is managed separately. |  |
| `dedicated-network-accesses` | Dedicated Networks | Companion API to `dedicated-network` that grants, changes, or revokes which devices may use a reserved connectivity environment. |  |
| `dedicated-network-profiles` | Dedicated Networks | Discovery API for the catalogue of network profiles (predefined Dedicated Networks configurations) a CSP offers. | |
| `dedicated-network-areas` | Dedicated Networks | Discovery API that exposes eligible service areas and the QoS/network profiles supported within each, so a consumer can select a valid area before reserving. | Yes |
The CQM APIs can be grouped by purpose:

![CQM Portfolio Grouping by Purpose](../images/section5_portfolio_grouping.svg)
*Figure 2 — CQM APIs grouped by purpose. Preview capabilities are explicitly labelled. Actual API availability depends on CSP offering and market context.*

- **Discovery / support APIs** — expose information about what the CSP makes available without changing connectivity behaviour:
  - `qos-profiles`
  - `dedicated-network-profiles`
  - service-area discovery (`dedicated-network-areas`, preview capability)

- **On-demand QoS session establishment** — request a defined QoS session for immediate, time-bounded use:
  - `quality-on-demand`

- **Provisioned QoS assignment** — associate a QoS profile with a device and apply it to all its traffic whenever the device connects, until the assignment is revoked:
  - `qos-provisioning`

- **Connectivity quality booking** — to get confidence that the QoS Profile is usable for a given time window, which can start immediately or at a future point, and where the API allows, a defined service area:
  - `qos-booking`
  - `qos-booking-and-assignment`
  - `dedicated-network`

- **Device assignment** — link one or more devices to booked connectivity quality. For `qos-booking-and-assignment` and `dedicated-network` (via `dedicated-network-accesses`), device assignment can be managed in separate steps after the booking.
  - `qos-booking-and-assignment`
  - `dedicated-network-accesses`

The core portfolio description is based on published CAMARA specifications. Preview capabilities are explicitly labelled and should not be interpreted as generally available CSP offerings.

## 6. API Deep Dives

Each API is explained through the need it addresses, what it controls or exposes, the API consumer takeaway, and what it should not be confused with. For a more detailed description on API Endpoint- and schema-level, please refer to the individual API descriptions and / or definitions.

### 6.1 Discovery / support APIs

#### `qos-profiles` — Discovering available QoS profiles

| Aspect | Explanation |
| --- | --- |
| Need addressed | The API consumer needs to know which QoS profiles are available and how they are identified before requesting, assigning or reserving connectivity quality. |
| What it exposes | Information about QoS profiles and the expected connectivity performance. |
| API consumer takeaway | Treat `qos-profiles` as a supporting catalogue. It does not change connectivity behaviour directly. |
| Not to be confused with | Activating a QoS Session, reserving QoS, provisioning a device or creating a dedicated network. |

#### `dedicated-network-profiles` — Discovering available network profiles

| Aspect | Explanation |
| --- | --- |
| Need addressed | The API consumer needs to understand which network profiles the CSP exposes before creating a dedicated network booking. |
| What it exposes | A catalogue of network profiles describing supported Dedicated Networks configurations. |
| API consumer takeaway | This is the discovery API for network profiles within the Dedicated Networks family. |
| Not to be confused with | Creating a dedicated network booking or managing device access. |

#### Service-area discovery (`dedicated-network-areas`, preview capability)

| Aspect | Explanation |
| --- | --- |
| Need addressed | The API consumer needs to identify eligible service areas and understand which QoS profiles and/or network profiles are supported in each area before creating a reservation. |
| What it exposes | A catalogue of service areas, including their geographic definition and supported QoS profiles and/or network profiles. |
| API consumer takeaway | This optional preview capability helps the API consumer select an eligible service area without understanding internal network topology. |
| Not to be confused with | A generic coverage map, a guarantee that any arbitrary area can support the requested profile, or the creation of a booking or dedicated network. |

### 6.2 On-demand QoS session establishment

#### `quality-on-demand` — Immediate, time-bounded QoS sessions

| Aspect | Explanation |
| --- | --- |
| Need addressed | The API consumer needs defined connectivity quality immediately for one or more application data flows between a device and application server(s). |
| What it controls | A QoS session associated with a QoS profile and a defined duration. |
| API consumer takeaway | This is the immediate “apply a QoS profile now, for a bounded duration” tool. |
| Not to be confused with | Future reservations, persistent QoS assignment, multi-device assignment or reserved connectivity environments. |

A QoS session may not become available, or may later become unavailable, if the network cannot fulfil the requested QoS profile under the current conditions.

### 6.3 Provisioned QoS assignment

#### `qos-provisioning` — Long-lived device-level QoS assignment

| Aspect | Explanation |
| --- | --- |
| Need addressed | A device needs to remain associated with a QoS profile beyond a time-bounded session. |
| What it controls | A QoS-profile assignment configured in advance and applied whenever the device connects to the access network, until the assignment is revoked. |
| API consumer takeaway | The key distinction from `quality-on-demand` is persistence: the API consumer does not need to create a new QoS session every time the device reconnects. |
| Not to be confused with | A universal guarantee that identical measured performance will be available at every time and location, or with a future time-and-area reservation. |

### 6.4 Reservation-based connectivity quality

#### `qos-booking` — Connectivity booking for a time window and service area

| Aspect | Explanation |
| --- | --- |
| Need addressed | The API consumer needs defined connectivity quality for a specific time window — which can start immediately or at a future point — and a defined service area. |
| What it controls | A connectivity booking for one QoS profile, one time window, one service area and one device. |
| API consumer takeaway | The essential difference from `quality-on-demand` is that a future-dated booking provides confidence that the QoS Profile is usable by the requesting API consumer, obtained through CSP confirmation ahead of use. A booking requested for immediate start does not offer this lead time and behaves closer to an on-demand request. The essential difference from `qos-provisioning` is that the booking is bounded by a time window and service area. |
| Not to be confused with | Multi-device assignment, network profiles or reserved connectivity environments. |

A successfully processed request may still indicate that the requested reservation is unavailable. The API consumer should rely on the booking status to understand the service-availability outcome.

#### `qos-booking-and-assignment` — Connectivity booking with device assignment

| Aspect | Explanation |
| --- | --- |
| Need addressed | Connectivity needs to be booked for a time window — which can start immediately or at a future point — and a service area, while the devices that will use it are not all known at booking time or may change. |
| What it controls | A booking for one QoS profile across one or more devices, together with the ability to assign and release device assignments during its lifecycle. |
| API consumer takeaway | The booking confirmation gives the API consumer confidence that the QoS Profile will be usable for the requested number of devices, not just for the QoS profile itself. The additional value is separating the booking from the final device list, allowing devices to be added, removed or replaced without recreating the booking. As with `qos-booking`, this confidence is strongest when the booking targets a future start time. |
| Not to be confused with | A simple per-device booking or a reserved connectivity environment that exposes multiple QoS profiles. |

#### `dedicated-network` — Reserved connectivity environment

In the Dedicated Networks family, a network profile describes the capabilities for the reserved connectivity environment offered by the CSP, potentially including multiple QoS profiles and related limits. The booking of the connectivity environment is done using the `dedicated-network` API.

| Aspect | Explanation |
| --- | --- |
| Need addressed | The API consumer needs to book connectivity quality — for immediate or future use — without binding devices at the time of the booking. This can be done for a scenario that may require one or multiple QoS profiles. |
| What it controls | A connectivity booking based on a network profile, within which one or more QoS profiles can be made available. Device assignment is managed separately via the `dedicated-network-accesses` API. |
| API consumer takeaway | Like other booking APIs, `dedicated-network` provides confidence that the QoS Profile is usable by the requesting API consumer. It is relevant when the scenario requires not to bind devices at booking time — allowing separation between the connectivity booking and the devices that will use it — and when the scenario may require one or multiple QoS profiles within a single booking. As with `qos-booking`, this confidence is strongest when the booking targets a future start time.|
| Not to be confused with | A single-profile booking or an on-demand QoS session. |

Where supported by the CSP offering, `quality-on-demand` may be used once the dedicated network is active to create QoS sessions dynamically using the QoS profiles made available by the relevant network profile.

#### `dedicated-network-accesses` — Device-access management for a reserved connectivity environment

| Aspect | Explanation |
| --- | --- |
| Need addressed | Devices need to be granted, changed or revoked access to a reserved connectivity environment. |
| What it controls | Device accesses associated with a `dedicated-network`. |
| API consumer takeaway | This is the device-access companion to `dedicated-network`: it manages which devices may use the reserved connectivity environment. |
| Not to be confused with | Discovering network profiles, creating a connectivity booking or creating the reserved connectivity environment itself. |

## 7. Comparative Matrix and Purpose-Oriented Navigation

| API or capability | Category | Timing | Service area | Device model | QoS / profile model |
| --- | --- | --- | --- | --- | --- |
| `qos-profiles` | Discovery / support | n/a | n/a | Optional device filter | Catalogue of QoS profiles |
| `dedicated-network-profiles` | Discovery / support | n/a | n/a | n/a | Catalogue of network profiles |
| Service-area discovery (`dedicated-network-areas`, preview capability) | Discovery / support | n/a | Catalogue of eligible service areas | n/a | Associates areas with supported QoS profiles and/or network profiles |
| `quality-on-demand` | On-demand QoS session | Immediate, session duration | Not an explicit request dimension | Application flows associated with a device | One QoS profile per session |
| `qos-provisioning` | Provisioned QoS assignment | Persists until revoked | No explicit service area in the API contract | Device | One QoS profile assignment |
| `qos-booking` | Reservation-based | Time window (immediate or future)* | Defined service area | One device per booking | One QoS profile per booking |
| `qos-booking-and-assignment` | Reservation-based | Time window (immediate or future)* | Defined service area | Requested number of devices assigned to or released from a booking | One QoS profile per booking |
| `dedicated-network` | Reservation-based | Time window (immediate or future)* | Defined service area | Devices managed separately via `dedicated-network-accesses` | One or multiple QoS profiles per booking |
| `dedicated-network-accesses` | Device-access management | Follows the dedicated network booking lifecycle | Inherited from the dedicated network | Device-access management | Inherited from the dedicated network |

For reservation-based APIs (`qos-booking`, `qos-booking-and-assignment`, `dedicated-network`), the booking allows the API consumer to get confidence that the QoS Profile is usable, and to obtain confirmation from the CSP. _*Confidence from CSP confirmation is most meaningful for a future start time; an immediate-start booking has limited lead time for confirmation and behaves closer to an on-demand request._

This matrix makes portfolio roles visible. It is **not** a strict API selection guide and does not guarantee CSP support for any specific combination.

## 8. Key Takeaways, Guardrails and Sources

### 8.1 Key takeaways

The CQM portfolio is best understood as a set of API tools for different connectivity-quality needs. The main dimensions that distinguish them are:

- whether the need is immediate, long-lived or planned for a future window
- whether a service area is part of the request
- whether one device or several devices are involved
- whether device assignment must be managed over time
- whether one QoS profile per booking is sufficient or the scenario requires multiple QoS profiles within a single booking / booking of a reserved connectivity environment

Discovery APIs help API consumers understand what is available. On-demand and long-lived QoS tools apply a QoS profile dynamically or persistently. Reservation-based connectivity quality tools allow the API consumer to obtain confidence for the needed connectivity quality for a defined time window. This confidence is most meaningful for a future-dated booking, where the CSP can confirm availability ahead of use; a booking requested for immediate use has less lead time for confirmation and behaves more like an on-demand request. The QoS Booking and Assignment API extends this with the ability to manage multiple device assignments within a booking, and the Dedicated Networks family further extends this with multiple QoS profiles within a reserved environment.

### 8.2 Guardrails for external readers

1. **Do not assume operator topology.** API consumers work with external-facing abstractions such as QoS profiles, QoS sessions, connectivity bookings, service areas, device assignments and network profiles. They do not need to reason about internal network topology.

2. **Do not assume universal CSP support.** CAMARA specification availability and CSP availability are not the same thing. API consumers should check which APIs and capabilities their CSP exposes.

3. **Do not turn the portfolio into a strict decision tree.** The same scenario may be addressed differently depending on the API consumer need, CSP offering and market context.

4. **Match the API tool to the actual need**. Some needs require a QoS session; others require a long-lived assignment, a planned booking, device assignment or a reserved multi-profile environment.

5. **Distinguish API processing from service availability.** A successfully processed API request does not necessarily mean that the requested connectivity quality is available. API consumers should rely on resource status and status changes to understand the outcome.

6. **Describe optional mechanisms only when they are explicitly supported.** Capacity pre-checks, confidence indications or prediction mechanisms should not be presented as standard CQM behaviour unless they are explicitly supported by the API or CSP offering.

### 8.3 Sources and version notes

This document is based on the published CAMARA specifications for the in-scope CQM APIs and on public or shareable CQM explanatory material.

The CAMARA API overview also lists additional APIs under the broader Communication Quality area. These APIs are referenced in Appendix B only to clarify the boundary of this document.

Service-area discovery through `dedicated-network-areas` is presented as a preview capability and should not be interpreted as a generally available CSP offering.

The illustrative journey is informed by event, media production and temporary retail scenarios. These examples explain portfolio roles; they are not a normative mapping or API selection guide.

## Appendix

### Appendix A — Selected GSMA Use-Case References

The live-event journey is informed by selected GSMA Open Gateway use cases. These references are used only as explanatory anchors for the CQM portfolio. They are **not** a normative mapping between use cases and CQM APIs, and they should not be read as a strict API selection guide.

### Enterprise / B2B

- [Remote Journalist](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/remote-journalist/)
- [Remote Live Broadcast](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/remote-live-broadcast/)
- [Enhanced connectivity for retail stores](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/enhanced-connectivity-for-retail-stores/)
- [QoD for PoS transactions at events](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/qod-for-pos-transactions-at-events/)
- [Connection of remote venues](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/connection-of-remote-venues/)
- [Large Event Planning and Delivery](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/large-event-planning-and-delivery/)
- [Smart Video Surveillance](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/smart-video-surveillance/)

### Consumer / B2C

- [Reliable network for social media and gaming](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/reliable-network-for-social-media-and-gaming/)
- [Immersive large events](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/immersive-large-events/)

### Why planned connectivity may matter

The business-critical operations of many of the use cases listed above can be made more robust when connectivity quality is planned or reserved in advance.

For business-critical operations, many ASPs prefer **planned stability**: a practical expectation that the requested connectivity quality can be available when and where it is needed. This should not be interpreted as a standardised confidence-level feature or as an unconditional guarantee. It simply reflects the API consumer’s need to understand the expected service-availability outcome.

ASPs may be concerned about:

- the impact of **not obtaining** the requested connectivity service, such as a QoS session, booking or reserved connectivity environment, at the required time and place;
- the impact of **not maintaining** the requested connectivity quality during the execution of the service.

### Use-case references and CQM aspects illustrated

| GSMA use case | Why it is relevant for this document | CQM aspects illustrated | Notes / assumptions |
| --- | --- | --- | --- |
| [Remote Journalist](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/remote-journalist/) | Time-sensitive live contribution from a field location. | Immediate QoS, planned reservation, service-availability outcomes. | Connectivity quality is only available when the relevant request, session or reservation can be fulfilled by the CSP. |
| [Remote Live Broadcast](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/remote-live-broadcast/) | Media contribution with predictable uplink needs. | QoS profiles, immediate QoS, booking, multi-device scenarios. | The scenario can illustrate both immediate and planned connectivity-quality needs, depending on whether the production is spontaneous or scheduled. |
| [Enhanced connectivity for retail stores](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/enhanced-connectivity-for-retail-stores/) | Temporary or semi-stable enterprise connectivity at a known site. | Long-lived QoS assignment, booking, service area, reserved connectivity environment. | Dedicated Networks may be relevant where the CSP offering involves multiple QoS profiles, device-access management or a broader reserved connectivity environment. |
| [QoD for PoS transactions at events](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/qod-for-pos-transactions-at-events/) | Temporary event operations with potential business impact if connectivity quality is unavailable. | Immediate QoS, event reliability, service-availability outcomes. | This is a useful example to explain that a successfully processed API request does not always mean that the requested connectivity quality is available. |
| [Connection of remote venues](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/connection-of-remote-venues/) | Connectivity for a venue or site where predictable service conditions may be needed. | Service area, booking, network profile, reserved connectivity environment. | May require multiple QoS profiles or a broader reserved connectivity environment depending on the venue and service model. |
| [Large Event Planning and Delivery](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/large-event-planning-and-delivery/) | Planned venue operations involving time, area and potentially several devices. | Service area, planned reservation, device assignment, Dedicated Networks. | Useful to illustrate why pre-reservation can matter when the expected usage conditions are known in advance. |
| [Smart Video Surveillance](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/smart-video-surveillance/) | Video devices may require predictable connectivity, especially where deployment is temporary, mobile or business-critical. | Long-lived assignment, booking, device model, service area. | The exact CQM pattern depends on whether cameras are permanent, temporary, mobile or event-based. |
| [Reliable network for social media and gaming](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/reliable-network-for-social-media-and-gaming/) | Consumer or creator scenarios may require better connectivity quality under specific usage conditions. | Immediate QoS, possible planned reservation, service-availability outcomes. | The use case is broad. Social media consumption, creator live streaming and gaming sessions may lead to different CQM needs. |
| [Immersive large events](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/immersive-large-events/) | Dense-event environments can create high demand for predictable connectivity quality. | Service area, planned reservation, possible Dedicated Networks. | The connectivity purpose should be clarified before mapping it to a CQM tool. For highly congested event scenarios, planned reservation may be more appropriate than relying only on on-demand availability. |

A detailed mapping between GSMA use cases and potentially applicable CQM APIs may be maintained separately as internal analysis. It is intentionally not included in this external-facing document.

### Appendix B — Related CAMARA Communication Quality APIs outside this document’s scope

The CAMARA API overview includes additional APIs under the broader **Communication Quality** area. Some of them may be relevant to connectivity-quality use cases, but they are not part of the CQM portfolio explained in this document.

This appendix provides a short contextual reference only. It is **not** intended to extend the scope of the document, compare all Communication Quality APIs, or provide a selection guide.

| Related API | Relationship with CQM | Why it is outside the main scope of this document |
| --- | --- | --- |
| [Application Profiles](https://camaraproject.org/application-profiles/) | Describes application-level requirements or thresholds that may help other APIs understand what an application needs. | It supports application requirement description, but it is not itself a CQM control, assignment or reservation API. |
| [Connectivity Insights](https://camaraproject.org/connectivity-insights/) | Provides visibility into whether an application’s network requirements can be met for a given user session. | It is primarily an insight / assessment API, not an API for requesting, assigning or reserving connectivity quality. |
| [Connectivity Insights Subscriptions](https://camaraproject.org/connectivity-insights-subscriptions/) | Provides event-based updates when network quality changes or when requirements may no longer be met. | It supports monitoring and notifications, not direct connectivity-quality control or reservation. |
| [Predictive Connectivity Data](https://camaraproject.org/predictive-connectivity-data/) | Provides estimated future connectivity information for a geographic area or volume. | It supports planning and prediction, but does not reserve or assign QoS resources. |
| [Session Insights](https://camaraproject.org/session-insights/) | Allows application KPIs such as latency, jitter, packet loss and bitrate to be shared and assessed for a session. | It is focused on session monitoring and insight, not on CQM reservation or QoS assignment. |
| [Network Slice Booking](https://camaraproject.org/network-slice-booking/) | Allows reservation and management of network slice resources for designated areas and time periods. | It is related to reservation-based connectivity quality, but follows a network-slice-specific model rather than the CQM QoS / Dedicated Networks portfolio described in this document. |
| [Network Slice Assignment](https://camaraproject.org/network-slice-assignment/) | Allows devices or end users to be allocated to pre-applied network slice resources. | It is related to device allocation for slice resources, but remains part of the network-slice-specific API model rather than the CQM portfolio covered here. |

These APIs may be considered in broader Communication Quality discussions. However, the main body of this document remains focused on the APIs within the Connectivity Quality Management domain (`qos-profiles`, `quality-on-demand`, `qos-provisioning`, `qos-booking`, `qos-booking-and-assignment`, `dedicated-network`, `dedicated-network-accesses`, `dedicated-network-profiles`, `dedicated-network-areas`).
