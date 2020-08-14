ARM big.LITTLE is a thing where the CPU in something like a phone
has a few out-of-order execution cores optimized to get the best
single threaded performance they can get, which are useful for
running code that the user is waiting for (maybe something like
rendering a website so the user can start reading and scrolling,
much less CPU-intensive activities) and also a few in-order execution
cores which are optimized to run tasks which are not CPU intensive
like reacting to swipes and to the user typing and background tasks
like spying on you in an energy efficient way,
to maximize battery life.

The important part is that the hardware power management and the OS
cooperate to schedule the runnable threads on the hardware cores in
some intelligent manner to achieve user visible latency goals and
battery life goals. Threads are switched from high performance to
low power cores and back, as needed.

Some devices can run either the high perf or low power cores,
some can run both kinds of cores at the same time.

All phones and handheld game consoles based on Tegra have been doing
this for almost a decade now.

Switching threads between cores can only work if the cores support
the same instruction set and the caches are coherent (another word
from physics that maye computer people should not have used).

[This](http://www.mono-project.com/news/2016/09/12/arm64-icache/)
is the story of what happens when one core type has 64 byte
cache lines and the other core type has 128 byte cache lines and
the JIT doesn't know about it and has trouble clearing instruction
cache lines after JITting some code. It's a hilarious bug.
