# pg-tuning-guide
A practical guide and example repository for tuning PostgreSQL parameters based on workload, hardware, and use case.

pg-tuning-guide/
├── README.md                # Project overview, setup instructions, and usage
├── examples/                # Example configurations and scripts
│   ├── memory/             # Memory-related parameter examples
│   │   ├── work_mem.sql
│   │   ├── shared_buffers.sql
│   │   └── maintenance_work_mem.sql
│   ├── parallelism/        # Parallelism-related parameter examples
│   │   ├── max_parallel_maintenance_workers.sql
│   │   ├── max_parallel_workers.sql
│   │   ├── max_parallel_workers_per_gather.sql
│   │   └── max_worker_processes.sql
│   ├── jit/               # JIT-related parameter examples
│   │   ├── jit.sql
│   │   ├── jit_above_cost.sql
│   │   ├── jit_inline_above_cost.sql
│   │   └── jit_optimize_above_cost.sql
│   ├── connections/       # Connection-related parameter examples
│   │   ├── max_connections.sql
│   │   ├── idle_in_transaction_session_timeout.sql
│   │   └── idle_session_timeout.sql
├── scripts/                # Helper scripts for applying settings and monitoring
│   ├── apply_config.sh    # Bash script to apply config changes
│   ├── monitor_performance.sql  # SQL script to monitor performance
└── docs/                   # Additional documentation
    ├── memory.md
    ├── parallelism.md
    ├── jit.md
    └── connections.md
