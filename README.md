# OAI FlexRIC 5G Testbed

This repository provides comprehensive step-by-step instructions for integrating the OAI RAN, Core, and FlexRIC near-RT RIC to deploy an end-to-end 5G Standalone testbed for developing and testing xApps.

## Table of Contents

- [Overview](#overview)
- [Requirements](#requirements)
- [Repository Structure](#repository-structure)
- [Installation](#installation)
- [Usage](#usage)
- [References](#references)

## Overview

This testbed deployment integrates:

- OAI 5G RAN with E2 agent
- OAI 5G Core Network
- FlexRIC near-RT RIC
- Multi-UE simulation
- Traffic simulation
- Data collection for AI/ML

## Requirements

- Operating System: Ubuntu 22.04 LTS
- CPU: 8 cores x86_64 @ 3.5 GHz
- RAM: 32GB
- Storage: 16GB

## Repository Structure

```bash
OpenAirInterface-FlexRIC-Deployment-Guide/
├── Code/
├── Docs/
├── Installation Guide.md
├── Usage Guide.md
├── README.md
└── LICENSE
```

## Installation

For detailed setup instructions, see [Installation Guide](Installation%20Guide.md).

## Usage

For detailed usage instructions, see [Usage Guide](Usage%20Guide.md).

## References

If you use this setup for research, reference the OAI and FlexRIC projects.

- OAI RAN: [link](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/openair2/E2AP/README.md)  
- FlexRIC: [link](https://gitlab.eurecom.fr/mosaic5g/flexric#22-build-flexric)  
- OAI Core: [link](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_CN5G.md)  
- Multi‑UE: [link](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_multi_UE.md?ref_type=heads#run-multiple-ues-in-rfsimulator)
- OAM Workshop: [link](https://gitlab.eurecom.fr/oai/trainings/oai-workshops/-/blob/main/oam/README.md)  

## Acknowledgements

[NGNLab](https://ngnlab.org/), Department of Computer Technology, Anna University, MIT Campus
