### Understand the difference between synchronous and asynchronous operations
[very good read](https://medium.com/swlh/asynchronous-programming-296a656fe90d)
**Synchronous Operation:**
> A synchronous operation blocks the execution of the next line of code until the current operation completes. Lines are executed one-by-one, from the start of the program to the end.
> The call stack is used to manage synchronous subroutines. When a synchronous subroutine is called, it is pushed onto the call stack, and the CPU executes its instructions one by one.
> If the synchronous subroutine makes I/O operations, the CPU might be idle during this time, waiting for the I/O to complete, which can lead to inefficiency in resource utilisation.

**Asynchronous Operation:**
> An asynchronous operation allows the execution of other operations before the current operation completes. It doesn't block the progression of the program. Instead, it typically involves a mechanism to notify the program when the operation is complete (like a callback, promise, future, or event). 
> When an asynchronous subroutine performs an operation that would typically block (like I/O), it yields execution back to the caller, often by returning a **placeholder object** (like a future or promise). This object will eventually hold the result of the operation, but immediately after the yield, it is nothing more than a promise of a result to come later.
> The caller can continue executing other tasks while the placeholder is pending.
> Usually the async task will pass onto another thread that will poll its execution until it is complete, at which point it will return the value to the main thread.
> In the bare-metal context there's no OS threads to do multitasking (here: checking the status of the async operation). So how will the main program know when the async operation is done? And if there's more than one (blocking) operation, how does the async operation progress?
> 
> 1. **Interrupt Service Routines (ISRs)**: Hardware events can trigger ISRs, which are special types of subroutines. When an interrupt occurs, the MCU stops its current operation, saves its state, and runs the ISR, which is designed to handle the event <u>quickly</u> and return control back to the main program.
> 2. **State Machines**: The asynchronous subroutine, instead of blocking, quickly changes the state of the program based on the operation it needs to perform and then returns. The main program loop or scheduler can check these states and continue the operation of the subroutine when the condition it's waiting for (like data available from I/O) is met.


### Designing Asynchronous-Friendly Architecture
1. identify tasks that can be performed concurrently. These are typically I/O-bound tasks like network requests, file operations, or waiting for user input.
2. Design your application's architecture to separate synchronous and asynchronous operations. This separation helps in managing different types of tasks effectively.

### Deadlocks and Resource Starvation


### Performance Monitoring

