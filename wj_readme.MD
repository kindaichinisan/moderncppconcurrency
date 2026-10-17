Introduction to Concurrency in C++
https://www.youtube.com/watch?v=Fn0xBsmact4&list=PLvv0ScY6vfd_ocTP2ZLicgqKnvq50OCXM

# 1 The what and why of concurrency
# 2 First thread with std::thread
- thread1.cpp
- #include <thread>
- g++ -std=c++17 -pthread thread1.cpp -o prog
    - g++: compile c++ code. use GNU c++ compiler
    - -std=c++17: use c++17 language standard
    - -pthread: may be needed in Linux
    - can have multiple cpp files
    - can also compile each cpp file separately into .o file (using -c option to compile source code without linking executable) and then g++ to compile all the .o to final executable. This is common in big project.
    - -o <programname>: compile into executable
- myThread(&test, 100): pass in function name, then function argument. Can be multiple arguments.
- threadInst.join(): current thread wait for threadInst to complete, then continue.
-- join() needed in order to avoid core dump in program.

# 3 std::thread with a lambda in modern c++
- thread2.cpp
lambda function: functional programming
- lambda: function name
- []: capture list is empty. Let function access variables that already exist outside it.
    - [a]: pass a by value
    - [&a]: pass a by reference
- lambda = anonymous function = RHS of auto lambda=[](int x){...

# 4 Launching multiple std::thread in C++
- thread3.cpp
- create multiple threads
- this_thread::get_id()
- initial code create thread and push into vector, then join() and is destroyed, hence all thread ID is the same (dependent on OS C++ implementation if terminated thread ID is reused). In my Windows laptop, the terminated thread ID is not reused. In the course lecturer laptop, the terminated thread ID is reused.
- final code create 10 threads first and push into vector. Then use another for loop to join() them. They will have different thread ID.

# 5 jthread std::jthread in C++ 20
- thread4.cpp
- Uses jthread so that we dont need to join by end of main program. jthread will join for us in the destructor. Available in C++20. Previously, we use thread which is available in C++11.

# 6 std::mutex and preventing data races in C++
- thread5.cpp
- #include <mutex>
- mutex: mutual exclusion. Only 1 thread can enter critical section (atomic: smallest unit that can be accessed. Indivisible. Cannot be accessed by other thread until current thread finishes.) at any one time between lock() and unlock(). 
- comment gLock.lock(); and gLock.unlock(); in shared_value_increment() to see data race, unexpected behaviour due to non-sync of thread.
- indentation of code inside lock() and unlock()

# 7 Preventing deadlock with std::lock_guard in modern C++
- thread6.cpp
- if using lock() and forget to unlock(), will enter into deadlock as other threads cannot progress.
- Sometime, code is complicated and it is not just a matter of forget to unlock(). Eg is try catch block.
- soln: use better primitives like lock_guard (mutex wrapper that provides RAII-style). Auto unlock() when destructor of lock_guard is called
- scoped_lock is preferred one in C++17 with multiple mutex protection.
- RAII: Resource Acquisition Is Initialization. C++ programming technique where a resource is acquired when an object is constructed and automatically released when the object is destroyed. Key idea is to use object lifetime to manage resources automatically, instead of manually releasing them.

# 8 Using std::atomic in modern C++ to update a shared value
- atomics1.cpp
- #include <atomic>
- simpler soln if considering a single shared resource like variable a.
- atomic: atom or indivisible.
- use a++; or a+=1; instead of a=a+1;
- use hardware to implement.
- applicable for primitive data type (bool, char, short, int, etc.)

# 9 Example Data Parallel C++ Program using multiple threads in SFML
- sfml_grid.cpp
- #include <SFML/Graphics.hpp>
- SFML: Simple and Fast Multimedia Library
- update_grid is function executed by each thread. No need lock as each thread is working independently on each grid cell.
- Polymorphism: sf with different shapes (CircleShape, etc.)

# 10 Condition Variable in Modern cpp and unique lock
- cv.cpp
- #include <condition_variable>
- using normal mutex, other threads will be spinlocking while waiting, wasting resources.
- common in producer, consumer paradigm.
- required variables
    - boolean notified = false (for worker thread to inform reporter thread after completing its work)
    - lock - unique_lock
    - condition_variable
    - 2 threads (worker/ reporter: worker works first, reporter works after worker finishes.)
- use lambda function to make thread function more localized and easier to read. [&]{ ...}
- use unique_lock (flexible RAII lock that u can manually unlock and relock, vs lock_guard is a simple RAII lock)
- work thread to set notified to true and notify_one() when complete its work
- reporter thread to if(!notified) gConditionVariable.wait(lock);

| Feature | std::lock_guard | std::unique_lock |
|---|---|---|
| Auto lock | Yes | Yes |
| Auto unlock | Yes | Yes |
| Manual unlock | No | Yes |
| Condition variable | No | Yes |

- Predicate version vs if version
    '''
    gConditionVariable.wait(lock, [&] {
        return notified;
    });
    '''
    is equivalent to 
    '''
    while (!notified) {
        gConditionVariable.wait(lock);
    }
    '''

    There could be spurios wakeups (waiting thread waits up w/o notification)
- condition variable does not remb notifications but notified variable remember.
    - if reporter thread wait() after worker thread notify_one(), then it will wait forever because condition variable does not remember notifications.
    - this is prevented by std::unique_lock<std::mutex> lock(gLock); in both worker and reporter threads.
    - However, spurious wakeups is not prevented in existing code. Use predicate-based wait.

# 11 std::future and std::async in Modern cpp
- async.cpp
- #include <future>: future provides mechanism to access result of async operations.
- std::async: function without synchronization
- future<datatype> asyncFunctionName = async(functiontocall, functionparameters);: create the async function and calls it. Non-blocking and continues to next line after calling this.
- datatype result = asyncFunctionName.get();: blocking and wait for function to complete

# 12 std::async in cpp with background thread loading data example
- async_buffer.cpp
- youtube example of stopping video but streaming bar keeps progressing.
- argument in async function: launch_policy:
    - async
    - deferred
- future_status:
    - ready
    - timeout
    - deferred
- std::future_status status = backgroundThread.wait_for(std::chrono::milliseconds(1));
- if(status == std::future_status::ready){ ...
- This prevents u from using get() which is a blocking call. U can use wait_for() which is non-blocking call.

# Summary
- U can have thread but dont have mutex. Mutex is used to protect a critical section if multiple thread could access shared variables.
- U can use lambda function to define function that thread is doing. Function is more localized and easy to understand.
- Using normal thread, u need to join them eventually in main before returning for no segmentation fault to occur. Another solution is to use jthread (Available in C++20).
- Use mutex to lock() and unlock() critical section (atomic and indivisible).
- Use lock_guard as it is RAII. No need to manually unlock. It will automatically unlock when lock_guard destructor is called.
- Use atomic for simple shared resource operations like ++ or --.
- No need mutex, etc. if many threads are working on different variables of different elements of vector, etc. Data parallel operation.
- condition_variable reduces resource wastage as thread does not spinlock while waiting for resource. They sleep until they are woken up by notified_one() or notified_all().
- Beware of spurious wakeup by using predicate-based wait which is equivalent to while(!notified) wait [easier to understand].
- async function will execute in its thread (non-blocking). To get the result requires to wait until result is available.
- To not block when getting result of async function, u can use future_status and wait_for.

