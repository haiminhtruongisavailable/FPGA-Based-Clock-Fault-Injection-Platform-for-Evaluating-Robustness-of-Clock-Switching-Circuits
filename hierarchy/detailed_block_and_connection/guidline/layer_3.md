# Layer 3: Online Monitoring & Measurement Subsystem

## Architecture Hierarchy
```
Layer 3: Online Monitoring & Measurement Subsystem
│
├── glitch_detector
│   ├── Role: Detect short pulses (glitches) on the switched clock
│   ├── Inputs: switched_clk (from each MUX), threshold
│   ├── Outputs: glitch_detected, glitch_count
│   └── Internal: Edge detection + pulse width measurement logic
│
├── stuck_clock_detector
│   ├── Role: Detect when a clock stops toggling (stuck high or low)
│   ├── Inputs: switched_clk, timeout_value
│   ├── Outputs: stuck_detected
│   └── Internal: Inactivity counter with programmable timeout
│
├── enable_overlap_detector
      │   ├── Role: Detect unsafe simultaneous assertion of two enable signals
      │   ├── Inputs: enable1, enable2 (from MUX architectures)
      │   ├── Outputs: overlap_detected
      │   └── Internal: Simple combinational detection + synchronizer
      │
      ├── failover_latency_counter
      │   ├── Role: Measure time from fault injection to successful clock switch
      │   ├── Inputs: fault_trigger, switched_clk_valid
      │   ├── Outputs: failover_latency
      │   └── Internal: High-resolution cycle counter
      │
      ├── recovery_latency_counter
      │   ├── Role: Measure time from fault end to stable clock recovery
      │   ├── Inputs: fault_end, switched_clk_stable
      │   ├── Outputs: recovery_latency
      │   └── Internal: High-resolution cycle counter
      │
      ├── Role: Collect and store measurement results from all monitors
      ├── Inputs: All detector outputs and latency values
      ├── Outputs: Aggregated results to Layer 4
      └── Internal: Register bank + control logic for data capture
```
## Block Diagram (Initial Version)
```
                  Layer 3: Online Monitoring & Measurement Subsystem
┌──────────────────────────────────────────────────────────────────────────────────────────────┐
│                                                                                               │
│   Switched Clocks from Layer 2 (4 architectures)                                              │
│              │                                                                              │
│              ▼                                                                              │
│   ┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐            │
│   │Combinational │     │    BBM       │     │  Timer-based │     │     GFCM     │            │
│   │  switched_clk│     │  switched_clk│     │  switched_clk│     │  switched_clk│            │
│   └──────┬───────┘     └──────┬───────┘     └──────┬───────┘     └──────┬───────┘            │
│          │                    │                    │                    │                     │
│          ▼                    ▼                    ▼                    ▼                     │
│   ┌───────────────────────────────────────────────────────────────────────────────────────┐  │
│   │                              Online Monitors                                          │  │
│   │                                                                                       │  │
│   │   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐          │  │
│   │   │ glitch_      │   │ stuck_       │   │ enable_      │   │ latency_     │          │  │
      │   │   │ detector     │   │ detector     │   │ overlap_     │   │ counters     │          │  │
  │   │   └──────────────┘   └──────────────┘   └──────────────┘   └──────────────┘          │  │
      │   │                                                                                       │  │
      │   │   ┌──────────────┐   ┌──────────────┐                                                 │  │
      │   │   │ jitter_      │   │ duty_cycle_  │                                                 │  │
      │   │   │ detector     │   │ monitor      │                                                 │  │
      │   │   └──────────────┘   └──────────────┘                                                 │  │
      │   │                                                                                       │  │
      │   │                              error_aggregator                                         │  │
      │   └───────────────────────────────────────────────────────────────────────────────────────┘  │
      │                                               │                                               │
      │                                               ▼                                               │
      │                                    Measurement Results → Layer 4                              │
      │                                                                                               │
      └──────────────────────────────────────────────────────────────────────────────────────────────┘
```