# Windows OS Functions Used in Malware Techniques

The problem: I can see function imports using PE Studio or some other PE parsing program but I do not know what to look for.

The solution: Below

## Userland
### Threads
#### Locating a Thread handle
`CreateToolhelp32Snapshot`
`Thread32First`
`Thread32Next`

#### Intel on a thread in another process
`OpenThread` `GetThreadContext`
#### Controlling a thread in another process
`OpenThread`
`SetThreadContext`
`SuspendThread`
`ResumeThread`
`QueueUserAPC`
`NtQueueApcThread`
`SetWindowsHookEx`

#### Place a thread in an alertable state to exec APCs
`WaitForSingleObjectEx`
`WaitForMultipleObjectEx`
`Sleep`

#### Start a new thread in a process
`CreateRemoteThread`
`NtCreateThreadEx`
`RtlCreateUserThread`

### Processes
#### Get another process handle
`CreateToolhelp32Snapshot`
`Process32First`
`Process32Next`
#### Get current process handle
`GetCurrentProcess`

#### Create a new process
`CreateProcess`

### Drivers
#### Create a driver
`CreateService` with dwService set to 0x01

Use `CreateFileA` with the name specified above passed into the lpFileName argument to get a handle
#### Send a driver data
`DeviceIoControl`

### Memory
#### Write to memory of another process
`VirtualAllocEx` - Prepare memory

`WriteProcessMemory` - Write bytes to above memory

#### Create a new shared memory "[section](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/section-objects-and-views)"
`NtCreateSection`
#### Load that section into a process memory space

`NtMapViewOfSection`

#### Unmap memory 
`NtUnmapViewOfSection`
#### Change access permissions
`VirtualAllocEx`
### Atom Tables

#### Add data to global atom table
`GlobalAddAtom`
#### Get data from global atom table
`GlobalGetAtomName`

### DLLs
#### Load up a DLL
`LoadLibrary`
`GetProcAddress`

## Kernel
#### Queue APCs
`KeInitializeAPC`
`KeInsertQueueAPC`
#### Retrieve the address of a kernel function
`MmGetSystemRoutineAddress`

## Data Structures
`THREADENTRY32`

`CONTEXT`