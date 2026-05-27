## Architecture Hierarchy
```
      Layer 4: Control & Observation Layer
      │
      ├── main_central_fsm
      │   ├── Role: Top-level control of the entire platform
      │   ├── Inputs: Results from error_aggregator, user commands
      │   ├── Role: Top-level control of the entire platform
      │   ├── Inputs: Results from error_aggregator, user commands
      │
      ├── experiment_sequencer
      ├── experiment_sequencer
  │   ├── Inputs: Current experiment status, configuration
  │   ├── Inputs: Current experiment status, configuration
  │   ├── Outputs: Next fault scenario, experiment triggers
      │
      ├── register_map
      │   ├── Role: Configuration interface for fault parameters
      │   ├── Inputs: Write data from main_central_fsm or external
      │   ├── Outputs: fault_type, duration, target_clock, recovery_en, etc.
      │   └── Internal: Address-decoded registers
      │
      └── result_collector
          ├── Role: Receives and stores measurement results from Layer 3
          ├── Inputs: Aggregated data from error_aggregator
          ├── Outputs: Formatted results (to UART or memory)
          └── Internal: Result registers and logging logic
```
## Block Diagram

```
      ─ text
                           Layer 4: Control & Observation Layer
      ┌──────────────────────────────────────────────────────────────────────────────────────────────┐
      │   │                              main_central_fsm                                         │  │
      │   │                                                                                       │  │
      │   │   - Overall experiment control                                                        │  │
      │   │   - Coordinates Layer 1 and Layer 3                                                   │  │
      │   └───────────────────────────────────────────────────────────────────────────────────────┘  │
      │                    │                              │                                          │
      │                    │                              │                                          │
      │   ┌──────────────────────────────┐     ┌──────────────────────────────┐                     │
      │   │      experiment_sequencer    │     │         register_map         │                     │
      │   │                              │     │                              │                     │
      │   │   - Fault scenario control   │     │   - Fault configuration      │                     │
      │   │   - Experiment flow          │     │   - Duration, type, target   │                     │
      │   └──────────────────────────────┘     └──────────────────────────────┘                     │
      │                    │                              │                                          │
      │                    │                              ▼                                          │
      │                    │                    ┌──────────────────────────────┐                     │
      │                    │                    │        result_collector      │                     │
      │                    │                    │                              │                     │
      │                    │                    │   - Store Layer 3 results    │                     │
      │                    │                    │   - Prepare data for output  │                     │
      │                    │                    └──────────────────────────────┘                     │
      │                    │                                                                         │
      │                    ▼                                                                         │
      │   ┌───────────────────────────────────────────────────────────────────────────────────────┐  │
      │   │                        Control Signals                                                  │  │
      │   │                                                                                       │  │
      │   │   → Layer 1: fault_type, duration, trigger, recovery_en                               │  │
      │   │   → Layer 3: reset, read_enable, capture                                              │  │
      │   │   ← Layer 3: measurement results                                                      │  │
      │   └───────────────────────────────────────────────────────────────────────────────────────┘  │
      │                                                                                               │
      └──────────────────────────────────────────────────────────────────────────────────────────────┘
```
## Component Descriptions

### main_central_fsm

• Role: The brain of the platform. Controls the overall experiment flow and coordinates all other    layers.
• Responsibilities:
• Trigger fault injection
• Read results from Layer 3
• Decide when to move to the next fault scenario

### experiment_sequencer

• Role: Manages the sequence and timing of multiple fault injection experiments.
• Responsibilities:
• Stores a list of fault scenarios
• Controls the order and repetition of tests

### register_map

• Role: Provides configurable registers for fault parameters.
• Registers include: fault_type, duration, target_clock, recovery_en, trigger

result_collector

• Role: Collects and formats measurement data from Layer 3 for logging or transmission.