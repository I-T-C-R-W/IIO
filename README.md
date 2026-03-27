# Open Offload Interface (OOI)

[![Version](https://img.shields.io/badge/version-v0.1_Draft-blue.svg)](Spec.md)
[![Repository](https://img.shields.io/badge/github-I--T--C--R--W%2FOOI-black?logo=github)](https://github.com/I-T-C-R-W/OOI)

Welcome to the **Open Offload Interface (OOI)** repository. 

OOI defines a minimal, vendor-neutral protocol for the discovery and description of compute resources in heterogeneous systems. By describing device *behavior* rather than hardware *implementation*, OOI enables the creation of deterministic, scalable, and multi-layer compute systems.

## 🎯 Vision & Scope
Modern compute architectures consist of diverse processing units and complex memory hierarchies. OOI provides an abstraction layer that allows these heterogeneous systems to present themselves as a **single logical device** to the operating system, while internally coordinating multiple compute classes and memory tiers. Applications remain unaware of the underlying complexity, while the runtime ensures efficient and predictable execution.

## 📐 Design Principles
* **Behavior-oriented abstraction:** Focus on what a component does, not how it is built.
* **Minimal core specification:** Keep the core lightweight with clear extensibility.
* **Deterministic capability description:** Predictable latency, memory access, and execution.
* **Vendor neutrality:** Hardware agnostic design.
* **Forward compatibility:** Safe extension mechanisms for future technologies.

## 📄 Documentation

The complete technical specification can be found here:
👉 **[Read the OOI Specification (Spec.md)](Spec.md)**

### Table of Contents (Spec)
1. Scope & Design Principles
2. Discovery Flow
3. Capability Block Definition
4. Map Aggregation & OS Exposure
5. Deterministic Execution Model
6. Example Use Case (Real-Time Rendering)

## 🤝 Contributing
*(Add your contribution guidelines here for the Git release)*

## 📄 License
*(Add your chosen license here, e.g., MIT, Apache 2.0)*
