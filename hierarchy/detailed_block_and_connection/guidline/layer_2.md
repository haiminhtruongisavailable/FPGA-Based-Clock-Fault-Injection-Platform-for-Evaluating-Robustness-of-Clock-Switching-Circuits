# Layer 2: Clock MUX DUT Subsystem

## Architecture Hierarchy

``` 

 Layer 2: Clock MUX DUT Subsystem
      │
      ├── combinational_mux
      │   ├── Role: Simple combinational clock selection
      │   ├── Inputs: clk0_faulty, clk1_faulty, sel
      │   ├── Output: switched_clk
      │   └── Internal: Pure combinational logic (no state elements)
      │
├── bbm_mux (Break-Before-Make)
│   ├── Role: Safe switching using break-before-make handshake
│   ├── Inputs: clk0_faulty, clk1_faulty, sel
│   ├── Output: switched_clk
│   └── Internal: Cross-coupled enable logic + synchronizers (no FSM)
│
├── timer_based_mux
│   ├── Role: Safe switching using double-synchronizer delay chains
│   ├── Inputs: clk0_faulty, clk1_faulty, sel
│   ├── Output: switched_clk
│   └── Internal: DFF chains acting as timers + handshake logic (no FSM)
│
└── gfcm (Glitch-Free Clock Mux)
    ├── Role: Advanced glitch-free clock switching with failure detection
    ├── Inputs: clk0_faulty, clk1_faulty, sel
    ├── Output: switched_clk
    └── Internal: Clock detection + counters + enable handshake (no FSM)
```

## Block Diagram
```

                     Layer 2: Clock MUX DUT Subsystem (parallel running)
┌──────────────────────────────────────────────────────────────────────────────────────────────┐

│                                                                                               │

│   Faulty Clocks from Layer 1                                                                  │

│              │                                                                              │
│              ▼                                                                              │
│   ┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐            │

│   │   clk0_      │     │   clk0_      │     │   clk0_      │     │   clk0_      │            │

│   │   faulty     │     │   faulty     │     │   faulty     │     │   faulty     │            │

│   │              │     │              │     │              │     │              │            │

│   │   clk1_      │     │   clk1_      │     │   clk1_      │     │   clk1_      │            │

│   │   faulty     │     │   faulty     │     │   faulty     │     │   faulty     │            │

│   └──────┬───────┘     └──────┬───────┘     └──────┬───────┘     └──────┬───────┘            │

│          │                    │                    │                    │                     │

      │          ▼                    ▼                    ▼                    ▼                     │

      │   ┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐            │

      │   │Combinational │     │    BBM       │     │  Timer-based │     │     GFCM     │            │

      │   │     MUX      │     │     MUX      │     │     MUX      │     │     MUX      │            │

      │   │              │     │              │     │              │     │              │            │

      │   │  (No state)  │     │ (Handshake   │     │ (Synchronizer│     │ (Detection + │            │

      │   │              │     │  + CDC)      │     │   chains)    │     │  Counters)   │            │

      │   └──────┬───────┘     └──────┬───────┘     └──────┬───────┘     └──────┬───────┘            │

      │          │                    │                    │                    │                     │

      │          ▼                    ▼                    ▼                    ▼                     │

      │   switched_clk_0      switched_clk_1      switched_clk_2      switched_clk_3                 │

      │          │                    │                    │                    │                     │

      │          └────────────────────┴────────────────────┴────────────────────┘                     │

      │                                              │                                              │
      │                                              ▼                                              │
      │                                   To Layer 3 (Online Monitoring)                            │
      │                                                                                               │

      └──────────────────────────────────────────────────────────────────────────────────────────────┘
```

## Component Descriptions

### combinational_mux



### bbm_mux (Break-Before-Make)



### timer_based_mux



### gfcm (Glitch-Free Clock Mux)

