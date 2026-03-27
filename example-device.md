# OOI Extension: Heterogeneous System Behavior

OOI enables heterogeneous compute systems to behave like a single deterministic device with multi-tier memory and capability-based scheduling and allows heterogeneous hardware to behave as a single deterministic system, eliminating the need for explicit hardware management in applications.

---

## 12. Example Use Case – Heterogeneous Device (Front TPU + Nodes + GPU)

### 12.1 System Entry Point
The system exposes a single logical device to the operating system. All interaction is performed through a front-facing accelerator (TPU-class device) implementing the OOI interface.

--Example:

A rendering workload is submitted by the OS.

The front TPU receives the job and distributes:
- geometry processing to throughput nodes
- dynamic shading to flexible units (GPU)
- post-processing to deterministic units

All results are returned through shared memory without exposing internal routing.
-- 
**Communication Path:**
`OS` → `Driver` → `Front TPU` → `OOI Fabric` → `Internal Devices`

*Note: The operating system is not aware of the internal topology.*

### 12.2 Discovery Phase
At initialization, the front TPU performs OOI discovery across the internal fabric.

**Flow:**
`Front TPU` → `DISCOVER` → `Fabric` → `Devices` → `RESPONSES` → `Aggregation`

**Discovered devices may include:**
* Throughput nodes (ALU clusters)
* External accelerators (e.g., GPUs)
* Additional specialized units

Each internal device provides a standard OOI capability block.

### 12.3 Capability Aggregation
The front TPU aggregates all responses into a unified capability map that defines the global behavior of the system.

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
The driver exposes a single, unified device profile to the kernel. No internal device identifiers or bus addresses are exposed.

```text
DEVICE {
  compute_profile: mixed
  memory_profile: multi-tier
  supported_modes: deterministic, streaming
}
```

### 12.5 Execution Flow

**Step 1 – Work Submission**
The application submits work via a standard API:
```c
dispatch(workload)
```

**Step 2 – Scheduling**
The front TPU schedules tasks based on capability classes:
* `deterministic_units` → fixed pipeline execution
* `flexible_units` → dynamic or branching workloads (e.g., GPU)
* `throughput_units` → bulk processing (nodes)

**Step 3 – Data Placement**
Data is distributed across memory tiers automatically:
* **L1:** Active working sets (local ram)
* **L2:** Shared data (system fabric ram)
* **L3:** Streaming resources (storage/nvme)
* *Prefetching ensures availability before execution.*

**Step 4 – Offload**
External accelerators are used as specialized execution targets:
```text
if workload.type == flexible:
    assign_to_gpu()
else:
    assign_to_nodes()
```
Offload is handled transparently through the OOI protocol.

**Step 5 – Execution**
Execution follows a controlled, non-blocking model:
* **Deterministic mode:** Fixed dataflow patterns.
* **Streaming mode:** Continuous processing.
* **Sync mode:** Coordinated updates when required.
* *No direct inter-device synchronization is exposed to the application.*

**Step 6 – Result Integration**
Results are written back to shared memory (L2) and assembled. The front TPU returns the final output to the OS as a single completed task.

### 12.6 Properties
* **Single logical device abstraction:** Simplifies software stack.
* **Hidden internal topology:** Enhances security and hardware independence.
* **Capability-based scheduling:** Optimizes workload-to-hardware matching.
* **Multi-tier memory utilization:** Maximizes bandwidth efficiency.
* **Deterministic execution:** Provides predictable performance for real-time tasks.

### 12.7 Summary
This example demonstrates a system where the operating system interacts only with a front TPU device,
while the underlying fabric dynamically integrates nodes and external accelerators.
All internal complexity is abstracted through OOI,
allowing heterogeneous hardware to behave as a unified,
deterministic compute system.
