# SCADYS.io

**SCADYS.io develops intelligent marine electronics for sailing vessels and marine systems.** We create embedded systems that integrate seamlessly with NMEA 2000 networks, legacy marine protocols, and modern digital ecosystems. Our products are designed for both professional marine operations and cruising sailors who demand reliability in remote environments.

## Our Products

### MDD400 Marine Data Display
A touchscreen marine data display designed for NMEA 2000 integration. The MDD400 combines a responsive user interface with galvanically isolated CAN-bus communication, multi-protocol support (NMEA 2000, NMEA 0183, SeaTalk), and robust environmental sealing for offshore use. Built around an ESP32 microcontroller with comprehensive monitoring and configuration via mobile app.

**Use cases:** Primary navigation display, engine monitoring dashboard, integrated data hub for marine electronics.

### WTI400 Analog Wind Transducer Interface  
A specialized interface that bridges analog wind transducers to digital NMEA 2000 networks. The WTI400 digitizes analog wind speed and direction inputs, making legacy wind instrumentation compatible with modern boat electronics and navigation systems.

**Use cases:** Wind monitoring integration, legacy instrument migration, multi-source wind data aggregation.

## For Developers & Makers

We provide firmware, schematics, and technical documentation to support integration, customization, and experimentation:

- **Prototypes** – Engineering samples for evaluation and feedback
- **PCB Kits** – For makers who want to build their own enclosure
- **Build Kits** – Complete kits for hands-on assembly
- **Retail Units** – Fully certified, production-ready devices

All firmware is developed with the ESP-IDF framework and organized around reusable, modular components. Source code and technical documentation are available on GitHub to support the developer community.

## What You'll Find Here

- **Firmware repositories** _(coming soon)_ – Open-source embedded software for SCADYS.io products, built on ESP-IDF
- **Reusable components** _(coming soon)_ – Modular ESP-IDF components for marine electronics development (CAN-bus, NMEA, sensor drivers, etc.)
- **Technical documentation** – Product architecture, integration guides, compliance information, and API references (see [docs.scadys.io](https://docs.scadys.io))
- **Examples & tools** _(coming soon)_ – Sample applications, firmware build templates, and development utilities
- **Developer guidelines** – The design principles every component follows: [Component architecture & design principles](../docs/component-architecture.md); and how to consume our components across ESP-IDF, PlatformIO, and Arduino, including using a C component from a C++ app: [Consuming components](../docs/consuming-components.md)
- **Governance assets** – Organization policies and standards

## Design Philosophy

SCADYS.io firmware prioritizes:

- **reliability** – robust error handling and fail-safe behavior in remote marine environments
- **interoperability** – seamless integration with marine standards and third-party systems
- **modularity** – reusable components that can be shared across products
- **openness** – transparent architecture and accessible technical documentation

## Open Source & Proprietary Boundaries

We believe in open firmware to support the marine developer community. Our approach:

- **Open:** Firmware source code, technical documentation, interface specifications, and developer examples
- **Proprietary:** Hardware design files, manufacturing data, mechanical CAD, UI source assets, and production information

This boundary supports firmware customization and community contributions while protecting our hardware IP and manufacturing partnerships.

## Organization & Background

SCADYS.io is the current brand for marine electronics products developed by **GM Consult Pty Ltd**. We're consolidating legacy firmware and hardware documentation from earlier projects (SixSense Marine, GM Consult repositories) into a unified SCADYS.io ecosystem. Over time, legacy repositories will be archived as the migration completes.

For more information, visit [SCADYS.io](https://scadys.io) or check out our [technical documentation site](https://docs.scadys.io).