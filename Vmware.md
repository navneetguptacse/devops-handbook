# VMware Fusion VM Management using vmrun (CLI-Based)

![VMware](https://img.shields.io/badge/VMware-Fusion-blue)
![Tool](https://img.shields.io/badge/CLI-vmrun-green)
![Mode](https://img.shields.io/badge/Mode-Headless%20%7C%20GUI-orange)
![Platform](https://img.shields.io/badge/Platform-macOS-lightgrey)
![Automation](https://img.shields.io/badge/Automation-Scripting-yellow)
![Level](https://img.shields.io/badge/Level-Beginner--Friendly-success)
![License](https://img.shields.io/badge/License-MIT-brightgreen)

This guide explains how to manage VMware Fusion virtual machines using the `vmrun` CLI tool without navigating into VM directories.

---

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [VM Path](#vm-path)
- [Start Virtual Machine](#start-virtual-machine)
- [Stop Virtual Machine](#stop-virtual-machine)
- [Restart Virtual Machine](#restart-virtual-machine)
- [Pause and Resume](#pause-and-resume)
- [List Running VMs](#list-running-vms)
- [Stop All Running VMs](#stop-all-running-vms)
- [Best Practices](#best-practices)
- [Conclusion](#conclusion)

---

## Overview

`vmrun` is a command-line utility provided by VMware that allows you to:

- Start and stop virtual machines
- Run VMs in headless mode
- Automate VM lifecycle operations
- Manage multiple VMs via scripts

This approach is useful for:

- Automation workflows
- Dev/test environments
- CI/CD pipelines
- Remote VM management

---

## Prerequisites

Ensure the following:

- VMware Fusion installed on macOS
- `vmrun` available in your system PATH

Verify:

```bash id="u2l9rb"
vmrun -T fusion list
```

---

## VM Path

Always use the **absolute path** to the `.vmx` file:

```bash id="m9k3r1"
/Users/navneetgupta/Virtual Machines.localized/Ubuntu 24.04 LTS M1.vmwarevm/Ubuntu 24.04 LTS M1.vmx
```

This allows execution without changing directories.

---

## Start Virtual Machine

### Start in Headless Mode (No GUI)

```bash id="o8p4zl"
vmrun -T fusion start "<vmx-path>" nogui
```

Example:

```bash id="4r7m1x"
vmrun -T fusion start "/Users/navneetgupta/.../Ubuntu 24.04 LTS M1.vmx" nogui
```

---

### Start with GUI

```bash id="l9z8dp"
vmrun -T fusion start "<vmx-path>"
```

---

## Stop Virtual Machine

### Graceful Shutdown

```bash id="6i2xsl"
vmrun -T fusion stop "<vmx-path>" soft
```

---

### Force Stop

```bash id="q7p2dy"
vmrun -T fusion stop "<vmx-path>" hard
```

---

## Restart Virtual Machine

```bash id="f3t1sd"
vmrun -T fusion reset "<vmx-path>" soft
```

---

## Pause and Resume

### Pause VM

```bash id="2h4kdl"
vmrun -T fusion pause "<vmx-path>"
```

---

### Resume VM

```bash id="n9f8sj"
vmrun -T fusion unpause "<vmx-path>"
```

---

## List Running VMs

```bash id="c7v1rm"
vmrun -T fusion list
```

Example output:

```id="k3t9av"
Total running VMs: 1
/Users/.../Ubuntu 24.04 LTS M1.vmx
```

---

## Stop All Running VMs

```bash id="z8p2wl"
vmrun -T fusion list | tail -n +2 | while read vm; do
  echo "Stopping: $vm"
  vmrun -T fusion stop "$vm" soft
done
```

---

## Best Practices

- Always use **absolute paths** for reliability
- Prefer `soft` shutdown to avoid data corruption
- Use `nogui` mode for automation and servers
- Script repetitive operations for efficiency
- Validate VM status using `vmrun list`

---

## Conclusion

Using `vmrun`, you can fully control VMware Fusion VMs from the command line.

### Key Benefits

- No need to navigate directories
- Easy automation and scripting
- Efficient VM lifecycle management
