https://www.youtube.com/watch?v=Fn0xBsmact4&list=PLvv0ScY6vfd_ocTP2ZLicgqKnvq50OCXM

# 1 First thread with std::thread
- thread1.cpp
- g++ -std=c++17 -pthread thread1.cpp -o prog
    - g++: compile c++ code. use GNU c++ compiler
    - -std=c++17: use c++17 language standard
    - -pthread: may be needed in Linux
    - can have multiple cpp files
    - can also compile each cpp file separately into .o file (using -c option to compile source code without linking executable) and then g++ to compile all the .o to final executable. This is common in big project.
    - -o <programname>: compile into executable
- myThread(&test, 100): pass in function name, then function argument.
- threadInst.join(): current thread wait for threadInst to complete, then continue.
-- join() needed in order to avoid core dump in program.

# 2 std::thread with a lambda in modern c++
- thread2.cpp
lambda function: functional programming
- lambda: function name
- []: capture list is empty. Let function access variables that already exist outside it.
    - [a]: pass a by value
    - [&a]: pass a by reference
- lambda = anonymous function = RHS of auto lambda=[](int x){...

# 3 Launching multiple std::thread in C++
- thread3.cpp
- create multiple threads
- this_thread::get_id()
- initial code create thread and push into vector, then join() and is destroyed, hence all thread ID is the same (dependent on OS C++ implementation if terminated thread ID is reused). In my Windows laptop, the terminated thread ID is not reused. In the course lecturer laptop, the terminated thread ID is reused.
- final code create 10 threads first and push into vector. Then use another for loop to join() them. They will have different thread ID.

# 4 jthread std::jthread in C++ 20
- thread4.cpp
- Uses jthread so that we dont need to join by end of main program. jthread will join for us in the destructor. Available in C++20. Previously, we use thread which is available in C++11.

# 5 std::mutex and preventing data races in C++
- thread5.cpp
- comment gLock.lock(); and gLock.unlock(); in shared_value_increment() to see data race, unexpected behaviour due to non-sync of thread.