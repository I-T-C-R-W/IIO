## 12. Example Use Case – Heterogeneous Device (Front Device + Fabric)

OOI defines a model where heterogeneous compute systems behave as a single deterministic device, independent of internal topology.

---

### 12.1 System Entry Point
The system exposes a single logical device to the operating system. All interaction is performed through a front-facing device (e.g., TPU-class accelerator) implementing the OOI interface.

```text
OS → Driver → Front Device → OOI Fabric → Internal Devices
```

*Note: The operating system is not aware of internal structure or topology.*

### 12.2 Discovery
At initialization, the front device performs OOI discovery across the fabric.

**Flow:**
`Front Device` → `DISCOVER` → `Fabric` → `Devices` → `RESPONSES` → `Aggregation`

Devices compliant with OOI respond with capability descriptions. Discovered devices may include:
* **Throughput-oriented nodes:** (e.g., ALU clusters)
* **Flexible accelerators:** (e.g., GPUs)
* **Deterministic units:** (e.g., TPU pipelines)
* **Specialized resources:** Additional compute-specific hardware

### 12.3 Capability Aggregation
The front device aggregates all responses into a unified capability map. System behavior is defined entirely by this map; no static hardware configuration is required.

**Example Aggregated Map:**
```text
compute {
  deterministic_units: 8
  flexible_units: 1024
  throughput_units: 32
}

memory {
  L1: 32 GB
  L2: 32 GB
  L3: 512 GB (streaming)
}

access_modes {
  deterministic_mode: true
  streaming_mode: true
  sync_mode: true
}
```

### 12.4 OS Exposure
The driver exposes a single logical device profile. Internal devices are hidden, and no guarantees are made about individual device identity or physical execution paths.

```text
DEVICE {
  compute_profile: mixed
  memory_profile: multi-tier
  supported_modes: deterministic, streaming
}
```

### 12.5 Execution Model
Workloads are submitted through standard APIs:
`dispatch(workload)`

The front device schedules execution based on capability classes:
* **deterministic_units** → fixed pipeline execution
* **flexible_units** → dynamic or branching workloads
* **throughput_units** → bulk processing

External accelerators are treated as capability providers, not primary devices. They extend the system but do not define it.

### 12.6 Memory Model
Data is distributed across memory tiers. Memory placement is implementation-defined and not exposed to the application.

* **L1:** Active working sets
* **L2:** Shared system memory
* **L3:** Streaming and background data
* *Prefetching ensures data availability prior to execution.*

### 12.7 Execution Flow Example
A rendering workload is submitted. The front device assigns:
* **Geometry processing** to throughput units.
* **Dynamic shading** to flexible units.
* **Post-processing** to deterministic units.

Results are written to shared memory and assembled before being returned. All execution paths are abstracted from the application.

### 12.8 Properties
* **Single logical device abstraction**
* **Hidden internal topology**
* **Capability-based scheduling**
* **Multi-tier memory utilization**
* **Deterministic or near-deterministic execution**

### 12.9 Summary
OOI enables heterogeneous compute resources to operate as a unified system. Applications interact with a single device, while internal complexity is fully abstracted through capability-driven discovery and scheduling.
