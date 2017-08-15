# Windows OS Functions Used in Malware Techniques

The problem: I can see function imports using PE Studio or some other PE parsing program but I do not know what to look for.

The solution: Below

## Userland
### Locating a Thread
`CreateToolhelp32Snapshot`
`Thread32First`
`Thread32Next`

### Intel on a thread in another process
`OpenThread` `GetThreadContext`
### Controlling a thread in another process
`OpenThread`
`SetThreadContext`
`SuspendThread`
`ResumeThread`
`QueueUserAPC`
### Get another process handle
`CreateToolhelp32Snapshot`
`Process32First`
`Process32Next`
### Get current process handle
`GetCurrentProcess`

### Create a driver
`CreateService` with dwService set to 0x01

Use `CreateFileA` with the name specified above passed into the lpFileName argument to get a handle
### Send a driver data
`DeviceIoControl`


### Write to memory of another process
`VirtualAllocEx`

### 

## Kernel
### Queue APCs
`KeInitializeAPC`
`KeInsertQueueAPC`
### Retrieve the address of a kernel function
`MmGetSystemRoutineAddress`

## Data Structures
`THREADENTRY32`

`CONTEXT`