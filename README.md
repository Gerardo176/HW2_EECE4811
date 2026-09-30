# HW2_EECE4811

Q1). The new algorithm would not be correct because the **guard** acts as a safeguard for the lock's **flag**, without it, two threads could both see flag == 0 before either sets it to 1, causing them both to enter the critical section.

Q2). The modified implementation of the algorithm is incorrect because if unlock() is called by mistake when lock is already unlocked, the **flag** changes from 0 to -1 instead of remaining 0.

Q3). When the setpark() is added right before "m->guard = 0;" it solves the "Wakeup/Waiting Race" issue because adding it before releasing **guard** tells the system that the thread is intending to sleep and if unpark() occurs after the setpark() but before park(), that wakeup command is remembered, causing the later park() to return immediately instead of blocking.


**Code instructions**
Run on Linux with GCC and POSIX threads:
gcc -O2 -std=c11 -Wall -Wextra -Wpedantic -pthread locks.c -o locks
taskset -c 0-7 ./locks 5000 5 1000 8 > results.csv

These specify measured acquisitions per thread, trials, protected-work iterations, and maximum thread count. Program tests power of 2 from one-maximum. If IDs 0-7 don't work, choose 8 CPU Ids allowed on local machine.

**Program design**
Waiting time was measured using clock_gettime(CLOCK_MONOTONIC) immediately before and after each lock acquisition, excluding the protected work and unlocking.

Both locks used the same workload: 1,000 integer multiply/add operations and a shared-counter increment inside the critical section. Contention was varied using 1, 2, 4, and 8 threads, with execution restricted to eight CPUs.

Each configuration ran five trials. Every thread performed 1,000 warmup acquisitions followed by 5,000 measured acquisitions, producing 750,000 measurements overall. Threads started through a shared atomic gate, and the lock tested first alternated between trials.

The benchmark recorded mean, median, and percentile waiting times. Each trial also checked the shared counter, final workload value, and whether critical sections overlapped; all checks passed.

**Results**

Threads / Lock / Mean wait / SD of means / Median wait / p95 wait / p99 wait

1	/ Ticket / 0.034 / 0.009 / 0.029 / 0.030 / 0.031

1 / Queue / 0.038 / 0.021 / 0.029 / 0.030 / 0.033

2 / Ticket / 2.736 / 1.718 / 1.218 / 1.239 / 6.779

2 / Queue / 22.416 / 4.932 / 16.531 / 38.285 / 145.998

4 / Ticket / 6.607 / 2.121 / 3.797 / 4.043 / 45.603

4 / Queue / 67.919 / 6.788 / 40.566 / 169.056 / 509.767

8 / Ticket / 19.065 / 6.322 / 8.950 / 27.643 / 196.894

8 / Queue / 185.181 / 14.326 / 119.530 / 459.392 / 1239.089

**Analysis**
As waiting time increased, so did the contention as well. The ticket lock was faster in this experiment because spinning avoided the queue lock’s sleep and wakeup overhead for a short critical section. However, spinning consumes CPU time, so the queue lock may perform better with longer critical sections or more threads than available CPUs.

