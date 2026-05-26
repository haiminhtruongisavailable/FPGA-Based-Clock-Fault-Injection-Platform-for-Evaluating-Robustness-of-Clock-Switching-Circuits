# Layer 1: Fault Injection Subsystem

## Architecture Hierarchy

```
Layer 1: Fault Injection Subsystem
│
├── fault_controller
│   ├── Role: Main FSM + configuration interface
│   ├── Inputs (from Layer 4): fault_type, duration, target_clock, trigger, recovery_en
│   ├── Outputs:
│   │   - fault_type_reg
│   │   - duration_reg
│   │   - start_fault
│   │   - recovery_enable
│   └── Internal: Main Control FSM (5 states)
│
├── clock_fault_generator (parameterized, one per clock)
│   ├── Inputs: clk_clean, fault_type_reg, duration_reg, start_fault, recovery_enable
│   ├── Outputs: clk_faulty
│   └── Internal: Fault Generation FSM (6 states)
│
├── stop_high_logic
├── stop_low_logic
├── missing_pulse_logic
├── duty_distortion
├── jitter_generator
│   └── All connected to clock_fault_generator via internal mux
│
└── fault_sequencer
    ├── Inputs: start_fault, duration_reg
    └── Outputs: fault_active, fault_done
```

## Block Diagram

```
 Layer 1: Fault Injection Subsystem
  ┌──────────────────────────────────────────────────────────────────────────────────────────────┐
  │                                                                                               │
  │   ┌──────────────────────┐                                                                    │
  │   │   Layer 4            │                                                                    │
  │   │ (Control Layer)      │                                                                    │
  │   └──────────┬───────────┘                                                                    │
  │              │ fault_type, duration, target_clock, trigger, recovery_en                       │
  │              ▼                                                                                │
│   ┌──────────────────────┐      start_fault      ┌──────────────────────────────┐            │
│   │  fault_controller    │──────────────────────►│     fault_sequencer          │            │
│   │                      │                       │                              │            │
│   │  - Main Control FSM  │◄──────────────────────│  - Timing control            │            │
│   │  - Register Map      │   fault_active,       │  - Duration counter          │            │
│   └──────────┬───────────┘   fault_done          └──────────────────────────────┘            │
│              │                                                                                 │
│              │ fault_type_reg, duration_reg, recovery_enable                                 │
│              ▼                                                                                 │
│   ┌───────────────────────────────────────────────────────────────────────────────────────┐  │
│   │                              clock_fault_generator                                    │  │
│   │                                                                                       │  │
│   │   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐          │  │
│   │   │ stop_high    │   │ stop_low     │   │ missing_     │   │ duty_        │          │  │
│   │   │ _logic       │   │ _logic       │   │ pulse_logic  │   │ distortion   │          │  │
│   │   └──────────────┘   └──────────────┘   └──────────────┘   └──────────────┘          │  │
│   │                                                                                       │  │
│   │   ┌──────────────┐   ┌──────────────┐                                                 │  │
│   │   │ jitter_      │   │ recovery_    │                                                 │  │
│   │   │ generator    │   │ logic        │                                                 │  │
│   │   └──────────────┘   └──────────────┘                                                 │  │
│   │                                                                                       │  │
│   │                              Internal Fault Selection MUX                             │  │
│   └───────────────────────────────────────────────────────────────────────────────────────┘  │
│                                               │                                               │
│                                               ▼                                               │
│                                    clk_faulty (to Layer 2)                                    │
│                                                                                               │
└──────────────────────────────────────────────────────────────────────────────────────────────┘
```

## Component Descriptions

### fault_controller
- **Role**: Main FSM + configuration interface
- **States**: 5-state FSM
- **Key Responsibilities**: Register management, fault type selection, duration control

### clock_fault_generator
- **Parameterized**: One instance per clock domain
- **States**: 6-state FSM
- **Outputs**: Faulty clock (clk_faulty)

### Fault Injection Modules
All modules are selected via internal multiplexer based on `fault_type_reg`:
- **stop_high_logic**: Hold clock at high level
- **stop_low_logic**: Hold clock at low level
- **missing_pulse_logic**: Remove clock pulses
- **duty_distortion**: Alter clock duty cycle
- **jitter_generator**: Add timing jitter
- **recovery_logic**: Enable graceful fault recovery

### fault_sequencer
- **Role**: Timing and sequence control
- **Inputs**: start_fault signal, duration configuration
- **Outputs**: fault_active, fault_done signals
