# Useful Vocabulary

## Threads

#### Mutex
A ADS which allows only one thread to "grab" it at once. Thus, when another thread attempts to grab a mutex which is already held by another thread, it gets rejected.

#### Spinlock
When a thread is in an infinite loop attempting to grab a mutex. It will continuously "spin" throughout the loop until the mutex is no longer held by another thread, in  which case the spinlock ends for the subject thread.

#### Deadlock
When a spinlock is set up but a thread which holds a mutex never releases the mutex and "retires," the thread in the spinlock is said to be in a "deadlock" because it will spin forever, attempting to grab the mutex but never actually grabbing it.

#### Critical Section
A section of code which can only be executed by one thread at a time. Implemented with a mutex.

#### Readers/Writer Lock (RW Lock)
Allows multiple readers into the critical section, but only one writer.

#### Preventing Race Conditions
In order to be certain that two threads can't grab a mutex at the same instant and create a "race condition", we must use one single atomic instruction which both checks *and grabs* the mutex in one instruction. One instruction is Bit Test and Set (BTS). It moves the bit into the Carry Flag (CF) to test it but it also simultaneously sets the bit to 1 in one cycle.
```assembly
SpinLock:
  lock bts mutex, 0 ; Test and set bit 0 of mutex, load 0 if its not taken, 1 if it is
  jc SpinLock ; If the carry flag is 1, the mutex is already taken, so spin
  
  ; Critical Section Code goes here
  
  mov mutex, 0 ; if this doesn't happen, a deadlock will occur
  
  ret
```
Other options: Bit Test and Reset (BTR), Bit Test and Complement (BTC), Compare and Exchange (CMPXCHG),  Compare and Exchange 8 bytes (CMPXCHG8B), Compare and Exchange 16 Bytes (CMPXCHG16B) 

## Kernel Stuff
#### Interrupts
Intel architecture has an Interrupt Descriptor Table (IDT) of 256 interrupt structures, each of which is a struct that defines an interrupt handler for the given code. The address of the IDT is stored in a register called the IDT Register (IDTR). When the int command is received with a given code, the proc stops what it's doing and looks the code up in the IDT to determine how to act. Some of these codes are predefined by the architecture, while the majority (32-255) can be utilized however the OS wants.

The IDT can be viewed in WinDbg with the command `!idt`


### sysenter stuff
`sysenter`, `syscall`, and (obsolete) `int 0x2e` are used to "bridge the gap" from userland to kernel mode. The way this works is that a userland routine places a code which corresponds to a given kernel function (inside of the System Service Descriptor Table) inside of eax. It then executes the instruction `sysenter` which calls `KiFastSystemCall`. This routine takes the code, called the Dispatch ID, from eax and searches the *System Service **descriptor** table* -> *System Service Table* which finally has a pointer to an array of function pointers, which is called the *System Service **Dispatch** Table* (`KiServiceTable`) which serves as the lookup table for the Dispatch ID placed in eax. This tells the kernel which function to execute and at this point, it also fetches the stack and passes in the arguments as well.

See the SSDT in code:
```C
typedef struct _KSERVICE_DESCRIPTOR_TABLE
{
    PULONG ServiceTableBase; 
    PULONG ServiceCounterTableBase; 
    ULONG NumberOfServices; 
    PUCHAR ParamTableBase; 
}KSERVICE_DESCRIPTOR_TABLE,*PKSERVICE_DESCRIPTOR_TABLE;
```
These tables can be viewed with the following WinDbg commands:

```
dps nt!KeServiceDescriptorTable
dps nt!KiServiceTable L200
```

example output shows that in Win 7 there are 401 (0x191) entries in the Service Dispatch Table:
```
1: kd> dps nt!KeServiceDescriptorTable
fffff800`02b10840  fffff800`028e0300 nt!KiServiceTable
fffff800`02b10848  00000000`00000000
fffff800`02b10850  00000000`00000191
fffff800`02b10858  fffff800`028e0f8c nt!KiArgumentTable
fffff800`02b10860  00000000`00000000
fffff800`02b10868  00000000`00000000
fffff800`02b10870  00000000`00000000
fffff800`02b10878  00000000`00000000
fffff800`02b10880  fffff800`028e0300 nt!KiServiceTable
fffff800`02b10888  00000000`00000000
fffff800`02b10890  00000000`00000191
fffff800`02b10898  fffff800`028e0f8c nt!KiArgumentTable
fffff800`02b108a0  fffff960`00191f00 win32k!W32pServiceTable
fffff800`02b108a8  00000000`00000000
fffff800`02b108b0  00000000`0000033b
fffff800`02b108b8  fffff960`00193c1c win32k!W32pArgumentTable
```

sysenter is Intel's version, syscall is AMD's version of the instruction. It was created because int 0x2e was having to be called so often (many thousand times per second) that it was affecting performance since interrupts come with overhead.
### Implementation Differences between int 0x2e and sysenter
#### int 0x2e
The difference between int 0x2e and the more modern sysenter is that int 0x2e is legacy and also int 0x2e was used by first placing the proper kernel routine code in eax then having a user-mode routine mov an offset to `SharedUserData!SystemCallStub` into a register and then calling it. The call to `SystemCallStub` leads to one final user-mode routine being called: `ntdll!KiIntSystemCall` which places a pointer to the stack in edx and then finally executes the `int 2Eh` before returning.

#### sysenter (always used on x64 systems)
sysenter works a bit different. It uses a dedicated, hardware-provided path to the kernel mode system service handler, called `nt!KiFastCallEntry`. The "path" that the CPU provides is that the OS loads the address to `KiFastCallEntry` into the MSR register during system boot. That way, the CPU has this address inside of it at all times and does not have to deal with an interrupt. On x64, the routines which use sysenter or syscall simply place the stack ptr in r10 func code in eax and then call `syscall` or `sysenter` and that's it. For x86, they place the `SharedUserData!SystemCallStub` into edx then call it, which leads to `ntdll!KiFastSystemCall` which moves the stack ptr into edx and then executes `sysenter` or `syscall`, much like the int 2Eh method.

View the MSR reg (SYSENTER_EIP_MSR) with this command in WinDbg:
`rdmsr 176`
then take that address which is returned and do
`ln [adr]`

Dispatch IDs up to 0xFFF will be retrieved from `nt!KiServiceTable` SSDT and Dispatch IDs between 0x1000 and 0x1FFF will be retrieved from `win32k!W32pServiceTable` SSDT.
