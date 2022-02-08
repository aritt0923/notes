# Device Management

## Architecture

### Von Neumann Computer Architecture

- In 1945, von Neumann descrubed a "stored program" digital computer in which memory stored both instructions *and* data.

- This simplified loading of new programs and executing them without having to rewire the entire computer each time a new program needed to be loaded 

  ![“Screenshot”_2022-02-05_at_13.07.17](/Users/andrewrittenhouse/Documents/CU/CSCI_3753/notes/images/“Screenshot”_2022-02-05_at_13.07.17.png)

- Want to support more devices: Card reader, magnetic tape reader, printer, dislpay, disk, storage, etc.

- System bus evolved to handle multiple I/O devices

- Includes control, address and data buses

  ![“Screenshot”_2022-02-05_at_13.10.41](/Users/andrewrittenhouse/Documents/CU/CSCI_3753/notes/images/“Screenshot”_2022-02-05_at_13.10.41.png)

### A Typical PC Bus Structure (Graphic)

![“Screenshot”_2022-02-05_at_13.12.50](/Users/andrewrittenhouse/Documents/CU/CSCI_3753/notes/images/“Screenshot”_2022-02-05_at_13.12.50.png)

### Modern COmputer Architecture: Devices and the I/O Bus (Graphic)

![“Screenshot”_2022-02-05_at_13.13.40](/Users/andrewrittenhouse/Documents/CU/CSCI_3753/notes/images/“Screenshot”_2022-02-05_at_13.13.40.png)

### Examples of x86 Exceptions 

| Class                | Cause                                            | Examples                                                     | Return Behavior                                  |
| -------------------- | ------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------ |
| Trap                 | Intentional exception, i.e. "software interrupt" | System calls                                                 | Always returns to next instructionm synchronous  |
| Fault                | Potentially recoverable error                    | Divide by 0, stack overflow, invalid opcode, page fault, seg fault | Might return to current instruction, synchronous |
| (Hardware) Interrupt | Signal from I/O device                           | Disk read finished, packet arrived on netwrok interface card (NIC) | Always returns to next instruction, asynchronous |
| Abort                | Non-recoverable error                            | Hardware bus failure                                         | Never returns, synchronous                       |

#### Exception Table

| Exception Number | Description               | Exception Class   | Pointer to Handler |
| ---------------- | ------------------------- | ----------------- | ------------------ |
| 0                | Divide error              | Fault             | ---                |
| 13               | General protections fault | Fault             | ---                |
| 14               | Page fault                | Fault             | ---                |
| 18               | Machine check             | Abort             | ---                |
| 32-127           | OS-defined                | Interrupt or trap | ---                |
| 128              | System call               | Trap              | ---                |
| 129-255          | OS-defined                | Interrupt or trap | ---                |

- X86 Pentium: table of 256 different exception types
  - Some assigned by CPU designers (divide by 0, memory access violations, page faults)
  - Some assigned by OS, e.g. interrupts or traps
- Pentium CPU contains exception table base register that points to this table, so it can be located anywhere in memory

![“Screenshot”_2022-02-05_at_13.25.12](/Users/andrewrittenhouse/Documents/CU/CSCI_3753/notes/images/“Screenshot”_2022-02-05_at_13.25.12.png)

![“Screenshot”_2022-02-05_at_13.25.23](/Users/andrewrittenhouse/Documents/CU/CSCI_3753/notes/images/“Screenshot”_2022-02-05_at_13.25.23.png)

## Device Manager

- Controls operation of I/O devices 
  - Issue I/O commands to devices
  - Catch interrupts
  - Handle errors
  - Provide a simple and easy-to-use interface
    - Device independence: same interface for all devices

### Device Management Organization (Graphic)

![“Screenshot”_2022-02-05_at_13.29.40](/Users/andrewrittenhouse/Documents/CU/CSCI_3753/notes/images/“Screenshot”_2022-02-05_at_13.29.40.png)

### Device System Call interface

- Crate simple standard interface to access most devices
  - Every I/O device driver should support the following: open, close, read, write, set (ioctl in UNIX), stop, etc.
  - Block vs character
    - Specify how we talk to the device
  - Sequential vs direct/random acces
    - Old tape vs Disk
  - Blocking I/O vs non-blocking I/O
    - Blocking system call: process put on wait queue until I/O completes
    - Non-blocking system call: returns immediatly with partial number of bytes transferred, e.g. keyboard, mouse, network
  - Synchonous vs asynchronous
    - Asynchronous returns immediatly, but at some later time, the full number of bytes requested is transferred

## `ioctl` and `fcntl`(Input/Output Control)

- Avoids havign to create new system calls for each new device and / or unforseen device function
  - Helps make the OS / kernel extensible
- UNIX, Linus, MacOS all support `ioctl`, and Windows has its own version 