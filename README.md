# HW2_EECE4811

Q1). The new algorithm would not be correct because the **guard** acts as a safeguard for the lock's **flag**, without it, two threads could both see flag == 0 before either sets it to 1, causing them both to enter the critical section.

Q2). The modified implementation of the algorithm is incorrect because if unlock() is called by mistake when lock is already unlocked, the **flag** changes from 0 to -1 instead of remaining 0.

Q3). When the setpark() is added right before "m->guard = 0;" it solves the "Wakeup/Waiting Race" issue because adding it before releasing **guard** tells the system that the thread is intending to sleep and if unpark() occurs after the setpark() but before park(), that wakeup command is remembered, causing the later park() to return immediately instead of blocking.
