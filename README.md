# sst_c51

A lightweight, **cooperative event-driven framework** for **8051 / C51** targets.

It provides the core building blocks for event-driven bare-metal applications: tasks, event queues, and time events.

> This repository contains the **SST core/kernel code** and assertion support code only. It does **not** include a concrete chip port, BSP, sample project, or build system. To use it in a real product, you need to provide the platform-specific integration in your own project.

## Features

- Cooperative scheduling with simple and predictable behavior
- Event-queue-driven task processing
- Built-in time event support
- Suitable for bare-metal 8051 projects and custom integration
- `DBC_*` assertion support for debugging and fault detection

## Repository Layout

- `include/sst.h` - Public API
- `include/dbc_assert.h` - Assertion macros
- `src/sst0.c` - Kernel implementation

## Architecture Overview

- **SST_Evt** - Base event type
- **SST_Task** - Task object with an event queue and dispatch entry point
- **SST_TimeEvt** - Time event for delay-based or periodic triggering
- **SST_Task_post()** - Posts an event to a task
- **SST_Task_run()** - Starts the kernel event loop

Because scheduling is cooperative, a task returns control to the kernel only after it finishes processing the current event. In practice, task handlers should stay short and avoid long blocking operations.

## Port Layer and BSP Layer

This repository is the **kernel/core layer**. In a real project, it is typically used together with a **port layer** and a **BSP layer**.

### Port Layer

The port layer adapts the kernel to a specific compiler and MCU runtime environment.

Checklist:

- [ ] Provide `sst_port.h`
- [ ] Define `SST_PORT_MAX_TASK`
- [ ] Define `SST_PORT_INT_DISABLE()` / `SST_PORT_INT_ENABLE()`
- [ ] Define `SST_PORT_CRIT_STAT`
- [ ] Define `SST_PORT_CRIT_ENTRY()` / `SST_PORT_CRIT_EXIT()`
- [ ] Define `SST_REENTRANT` and `DBC_REENTRANT`
- [ ] Define `SST_ReadySet`
- [ ] Provide `SST_LOG2(...)`
- [ ] Define `SST_LockKey` if needed
- [ ] Define `SST_PORT_TASK_ATTR` / `SST_PORT_TASK_OPER` if needed

### BSP Layer

The BSP (Board Support Package) is responsible for board-level and peripheral initialization.

- [ ] Initialize the system clock or base chip runtime environment
- [ ] Initialize the timer or system tick used to drive `SST_TimeEvt_tick()`
- [ ] Configure and enable the required interrupt sources
- [ ] Initialize board-level peripherals such as GPIO and UART
- [ ] Provide low-power / idle support for the idle path

### Application Responsibilities

The application or platform layer typically still needs to provide:

- `sst_port.h` for compiler, interrupt, and critical-section integration
- `SST_onStart()` to start interrupts or the system tick
- `SST_onIdleCond()` for idle handling
- `DBC_fault_handler()` for assertion failures

In short: **SST handles scheduling and event dispatching, the port layer makes the kernel runnable on the target, and the BSP layer initializes board resources.**

## Design by Contract

`dbc_assert.h` reflects a **Design by Contract** philosophy rather than a simple collection of debug macros. The core idea is that software should state its assumptions explicitly and fail fast when those assumptions are violated.

- `DBC_REQUIRE()` expresses preconditions
- `DBC_ENSURE()` expresses postconditions
- `DBC_INVARIANT()` expresses conditions that must always remain true
- `DBC_ASSERT()` and `DBC_ERROR()` guard against invalid states and impossible code paths

In this model, assertion failures are not treated as something to silently ignore. They are treated as evidence that a contract has been broken somewhere in the system. The purpose of `DBC_fault_handler()` is therefore not to hide the failure, but to stop normal execution, preserve fault context, and transfer control to an application-specific recovery or shutdown strategy.

For embedded systems, this is especially important: disabling assertions may reduce visibility into faults, while a carefully designed fault handler becomes the last line of defense when the system's assumptions no longer hold.

## License

This project is licensed under the [MIT License](./LICENSE).
