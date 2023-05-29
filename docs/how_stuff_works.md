# How Stuff Works
The basic rundown:

1. Firmware is running on a microcontroller, which is a computer.
2. Computers run programs, and the firmware is the program.
3. Your computer has an operating system, like Linux or macOS. We also use an
operating system. It's called FreeRTOS.
4. In addition to FreeRTOS, we use libraries that let us do things like talk to
peripherals (GPIO, CAN, ADCs, etc) on the chip. These are provided by the
manufacturer of the microcontroller, Espressif.
5. We write most of our stuff in C.
6. We made a set of libraries on top of all this called Ember OS. One thing
that gives you is `ember-tasking.`

## `ember-tasking`
All operating systems provide some kind of scheduling of different programs,
since computers can really only do one thing at a time (barring multiple cores,
which we won't discuss).

`ember-tasking` is a light layer above FreeRTOS and a hardware timer, plus a
watchdog, that provides you with *cyclic tasks*. It makes this work:

```c
#include <stdio.h>

#include "ember_common.h"
#include "ember_taskglue.h"

static void example_init();
static void example_1Hz();

ember_rate_funcs_S module_rf = {
    .call_init = example_init,
    .call_1Hz = example_1Hz,
};

/*
 * Runs once on bootup.
 */
static void example_init()
{
    printf("Hello, I started up!");
}

/*
 * Runs once every second.
 */
static void example_1Hz()
{
    printf("Hello!"); // printing "Hello!" to the terminal once a second
}
```

### Available rates
You can specify a function to be called at initialization, and at 1, 10, 100, or 1000 Hz:

```c
#include <stdio.h>

#include "ember_common.h"
#include "ember_taskglue.h"

static void example_init();
static void example_1Hz();
static void example_10Hz();
static void example_100Hz();
static void example_1kHz();

ember_rate_funcs_S module_rf = {
    .call_init = example_init,
    .call_1Hz = example_1Hz,
    .call_10Hz = example_10Hz,
    .call_100Hz = example_100Hz,
    .call_1kHz = example_1kHz,
};

/*
 * Runs once on bootup.
 */
static void example_init()
{
    printf("Hello, I started up!");
}

/*
 * Runs once every second.
 */
static void example_1Hz()
{
    printf("Hello!"); // printing "Hello!" to the terminal once a second
}

/*
 * Runs at 10Hz.
 */
static void example_10Hz()
{
    printf("Hello!"); // printing "Hello!" to the terminal at 10Hz
}

...
```

### Interruptibility
We need to get something out of the way.

Computers can only do one thing at a time. You need to keep this rule in mind:

!!! note
    You can be interrupted at any time. When you are interrupted, the CPU will
    run something else (e.g., a different cyclic function) and then resume
    wherever it left off.

This means that while `example_1Hz` is running, it could be paused
(interrupted), and `example_10Hz` could run. If you share data between cyclic
tasks, you need to stay aware that the data could be modified when you might
not expect it.

Let's make some shared data and you'll see the problem:

```c
static int i = 0;

static void example_10Hz()
{
    i = 123;
    printf("10Hz: %d", i);
}

static void example_100Hz()
{
    i = 456;
    printf("100Hz: %d", i);
}
```

if these tasks were not interruptible, you would expect to see output like
`10Hz: 123` and `100Hz: 456` but say:

1. You're running in `example_10Hz`. You set `i = 123`.
2. You get interrupted before the `printf`. You're now running `example_100Hz`.
3. You set `i = 456`.
4. You print `100Hz: 456`.
5. Execution resumes in `example_10Hz`.
6. You read i, which is now 456, and print `10Hz: 456`.

`123` totally got lost!

The same thing applies to shared hardware. If you're interacting with hardware,
remember that you could be interrupted! It could be super important that you do
two actions one after another without delay or something.

Usually this stuff is not super annoying, even though it seems like it would
be. Most of the time, you're only *writing* to a variable in one place and
*reading* in another, and you can usually tolerate this kind of sloppy
synchronization. The computer science nerds would call this *eventual consistency*.

We battle this with a few tools:

- Shared data must be marked `_Atomic`, which you can also spell as `atomic`
if you `#include "ember_common.h"`. This doesn't solve data races, but it
prevents individual variables from being corrupted by being half-read or
half-written. The previous example really should say:
```c
static atomic int i = 0;
```
Although this doesn't fix the problem, just prevents `i` from being corrupted.

- You can disable interrupts if you need to, which will guarantee you won't be
interrupted. You must keep the time spent with interrupts disabled to a
minimum, as interrupts are critical for keeping the system functioning.
There is a limited number of functions you can call with interrupts disabled
(for example, you can't call `printf`).

If you come up against this kind of synchronization challenge, please reach out
to Dan. We'll have more extensive learning material on this in the future.
