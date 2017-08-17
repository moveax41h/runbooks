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
`sysenter`, `syscall`, and (obsolete) `int 0x2e` are used to "bridge the gap" from userland to kernel mode. The way this works is that a userland routine places a code which corresponds to a given kernel function (inside of the System Service Descriptor Table) inside of eax. It then executes the instruction `sysenter` which calls `KiFastSystemCall`. This routine takes the code from eax and searches the SSDT and eventually the `KiServiceTable` table which is an array of function pointers which serves as the lookup table for the code placed in eax. This tells the kernel which function to execute and at this point, it also fetches the stack and passes in the arguments as well.

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
dps nt!KiServiceTable L200
```
