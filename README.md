# rustcounter — Atomic Counter Linux Kernel Module

rustcounter is a Rust Linux kernel module that exposes an atomic counter through `/dev/rustcounter`. Every write to the device increments the counter, and every read returns the current value.

This project demonstrates safe concurrent kernel programming in Rust using `AtomicU64`, preventing data races and lost updates without requiring mutex locks.

## Features

- Linux character device implemented in Rust
- Atomic counter using `AtomicU64`
- Safe concurrent access
- Out-of-tree Linux kernel module
- Built using the Rust-for-Linux kernel infrastructure

## Project Structure

```text
rustcounter/
├── Makefile
├── rustcounter.rs
├── README.md
├── LICENSE
└── .gitignore
```

## Build Instructions

This project must be built inside a Linux virtual machine with Rust-enabled kernel support.

Install dependencies:

```bash
sudo apt update
sudo apt install -y build-essential linux-headers-$(uname -r) kmod tree
sudo apt install -y rustc-1.93 rust-1.93-src bindgen
```

Build the module:

```bash
make
```

Load the module:

```bash
sudo insmod rustcounter.ko
```

Verify the module loaded:

```bash
lsmod | grep rustcounter
sudo dmesg | tail
```

Unload the module:

```bash
sudo rmmod rustcounter
```

## Example Usage

Increment the counter:

```bash
echo bump | sudo tee /dev/rustcounter > /dev/null
```

Read the current value:

```bash
sudo cat /dev/rustcounter
```

Example output:

```text
0
1
3
2000
```

## Concurrency Demonstration

Two separate terminals can increment the counter simultaneously:

Terminal 1:

```bash
for i in {1..1000}; do
  echo bump | sudo tee /dev/rustcounter > /dev/null
done
```

Terminal 2:

```bash
for i in {1..1000}; do
  echo bump | sudo tee /dev/rustcounter > /dev/null
done
```

Final result:

```bash
sudo cat /dev/rustcounter
```

Output:

```text
2000
```

The exact total is preserved because the module uses atomic operations instead of unsafe shared mutable state.

## Design Notes

This module uses `AtomicU64` instead of `Mutex<u64>` because the only required operation is an atomic increment. Atomic operations provide lower overhead and avoid lock contention while remaining thread-safe.

The module is implemented as a Linux miscellaneous character device using the Rust-for-Linux kernel APIs.

## Future Work

- Add a RESET command to clear the counter
- Add `/proc/rustcounter` support
- Track per-process statistics
- Add timestamp logging for writes
- Add configurable module parameters

## Demo

Successful module load and device interaction inside Ubuntu VM:
<img width="648" height="108" alt="Captura de pantalla 2026-05-15 a la(s) 11 42 56 a m" src="https://github.com/user-attachments/assets/066e6f98-a051-471d-ad0d-eabe7bd46b37" />

## License

Licensed under GPL-2.0 to match the Linux kernel.
