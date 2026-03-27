# Open Offload Interface (OOI) Discovery and Capability Specification
**Version:** Draft v0.1

---

## 1. Scope
This specification defines a minimal protocol for the discovery and description of compute resources in heterogeneous systems. 

The goal is to provide a vendor-neutral abstraction layer that describes device behavior rather than hardware implementation, enabling deterministic and scalable multi-layer compute systems.

## 2. Design Principles
* **Behavior-oriented abstraction**
* **Minimal core specification with extensibility**
* **Deterministic capability description**
* **Vendor neutrality**
* **Forward compatibility**

---

## 3. Discovery Flow
The discovery lifecycle follows a strict sequence: 
`INIT` → `DISCOVER` → `RESPONSE` → `CAPABILITY MAP` → `DRIVER INITIALIZATION` → `OS EXPOSURE`

### 3.1 INIT
The controller initializes the interconnect and prepares for device discovery.

### 3.2 DISCOVER
The controller broadcasts a discovery request to all connected devices:

```text
DISCOVER_REQUEST { 
    version 
    requested_fields 
}
```

### 3.3 RESPONSE
Each device responds with its identity and capability block:

```text
DISCOVER_RESPONSE { 
    device_id 
    device_class 
    capability_block 
}
```

---

## 4. Capability Block

### 4.1 Compute Classes
```text
compute { 
    deterministic_units: integer  # Compute elements with predictable latency and behavior
    flexible_units: integer       # Compute elements optimized for dynamic workloads
    throughput_units: integer     # Compute elements optimized for high-volume workloads
}
```

### 4.2 Memory Model
Latency and bandwidth are expressed as abstract classes rather than fixed values.

```text
memory { 
    L1 { 
        size 
        latency_class 
    } 
    L2 { 
        size 
        bandwidth_class 
    } 
    L3 { 
        size 
        streaming: boolean 
    } 
}
```

### 4.3 Access Modes
```text
access_modes { 
    deterministic_mode: boolean 
    streaming_mode: boolean 
    sync_mode: boolean 
}
```

### 4.4 Scheduling Capabilities
```text
scheduling { 
    partitioning: boolean 
    prefetch: boolean 
    reduction: boolean 
    priority_levels: integer 
}
```

### 4.5 Offload Interface
```text
offload { 
    accepts_jobs: boolean 
    preferred_job_size: small | medium | large 
    latency_profile: low | medium | high 
}
```

### 4.6 Performance Envelope
```text
performance { 
    compute_class: low | mid | high 
    efficiency_class: low | mid | high 
}
```

---

## 5. Capability Map Aggregation
The controller aggregates all device responses into a unified capability map:

```text
CAPABILITY_MAP { 
    global_compute 
    global_memory 
    available_modes 
    device_groups 
}
```

## 6. Driver Responsibilities
The driver is responsible for:
* Interpreting the capability map.
* Grouping compute resources by behavior.
* Selecting scheduling strategies.
* Exposing a unified device abstraction to the operating system.

## 7. OS Exposure Model
The operating system is presented with a single logical device. Internal topology and individual nodes are not exposed.

```text
DEVICE { 
    compute_profile 
    memory_profile 
    supported_modes 
}
```

## 8. Deterministic Execution Model
A device is considered deterministic when:
* Execution modes are explicitly defined.
* Memory access patterns are controlled and predictable.
* Scheduling avoids unbounded dynamic behavior.

## 9. Extension Mechanism
Unknown fields must be safely ignored by the controller and driver.

```text
extensions { 
    vendor_specific: optional 
    future_fields: allowed 
}
```

## 10. Versioning
Format: `version: major.minor`
* **Major versions:** Introduce breaking changes.
* **Minor versions:** Introduce backward-compatible extensions.

---

## 11. Example Use Case: Real-Time Rendering

### 11.1 System Overview
A heterogeneous compute device consists of:
* A front-facing accelerator (TPU-class device) acting as the primary interface.
* Multiple compute nodes providing throughput-oriented execution.
* Optional external accelerators (e.g., discrete GPUs).
* A multi-tier memory system (L1: local, L2: shared, L3: streaming).

### 11.2 Discovery Phase
Example aggregated capability after discovery:

```text
compute { 
    deterministic_units: 8 
    flexible_units: 1024 
    throughput_units: 32 
}

memory { 
    L1: 32 GB                  # fastram 
    L2: 32 GB                  # slowram 
    L3: 512 GB (streaming)     # nvme 
}

access_modes { 
    deterministic_mode: true 
    streaming_mode: true 
    sync_mode: true 
}
```

### 11.3 OS Exposure
The driver exposes a single logical device:

```text
DEVICE { 
    compute_profile: mixed 
    memory_profile: multi-tier 
    supported_modes: deterministic, streaming 
}
```

### 11.4 Workload Execution
1. **Resource Allocation:** Driver distributes data across L1, L2, and L3.
2. **Workload Partitioning:** Tasks assigned to Deterministic, Flexible, or Throughput units.
3. **Execution:** System operates in deterministic mode with prefetching.
4. **Data Movement:** Results written to L2; L3 used for background streaming.
5. **Output:** Results assembled and returned through the front-facing accelerator.

### 11.5 Properties Achieved
* Deterministic execution behavior.
* Predictable memory access patterns.
* Transparent abstraction of underlying hardware.

---

## Summary
This specification defines a minimal framework for describing heterogeneous compute systems based on behavior and capability. It enables deterministic, multi-layer compute platforms across diverse hardware.
