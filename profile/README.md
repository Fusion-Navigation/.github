# Fusion Navigation

Fusion Navigation is building a navigation system that works without GNSS — no GPS, no satellites, no infrastructure dependency.

The system currently fuses two independent passive sensing modalities: solar polarization compass and stellar navigation. Future iterations will incorporate map-matching, visual odometry, and additional sensor sources as the fusion layer matures. The goal is a navigation solution that is accurate, secure, and low-cost enough to work in any environment where GNSS is unavailable, unreliable, or actively denied.

## Repositories

| Repository | Description |
|---|---|
| [`fnav-core`](https://github.com/Fusion-Navigation/fnav-core) | Shared interfaces, EKF core, and navigation state models — RPi-only |
| [`fnav-lab`](https://github.com/Fusion-Navigation/fnav-lab) | Navigation pipeline R&D: algorithms, visualizations, data processing, and tests |
| [`fnav-solar`](https://github.com/Fusion-Navigation/fnav-solar) | Polarization compass module: sky polarization-based heading estimation |
| [`fnav-stellar`](https://github.com/Fusion-Navigation/fnav-stellar) | GPS-denied celestial navigation: CNN constellation classifier + star-based position estimation |
| [`fnav-data-capture`](https://github.com/Fusion-Navigation/fnav-data-capture) | Mobile app for onboard sensor data capture |
| [`fnav-ui`](https://github.com/Fusion-Navigation/fnav-ui) | Ground station and demo UI |
| [`.github`](https://github.com/Fusion-Navigation/.github) | Org-wide code policy, contributing guidelines, and PR templates |

## Standards

All development standards — branching, pull requests, code style, commit messages, secrets policy, and module contracts — are defined in [CODE_POLICY.md](../CODE_POLICY.md).
