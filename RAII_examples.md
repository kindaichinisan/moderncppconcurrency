
# 1 Without RAII vs. with RAII
Without RAII:
```FILE* fp = fopen("image.raw", "wb");

if (fp != nullptr) {
    // Write data
    fwrite(data, 1, size, fp);

    fclose(fp);  // Must remember to close
}
```
If an exception or an early return occurs before fclose(), the file might not be closed.

With RAII:
```
#include <fstream>

void saveImage(const std::string& filename,
               const unsigned char* data,
               size_t size)
{
    std::ofstream file(filename, std::ios::binary);

    if (!file) {
        return;
    }

    file.write(
        reinterpret_cast<const char*>(data),
        size
    );

} // file is automatically closed here
```
std::ofstream manages the file resource. When file goes out of scope, its destructor closes the file automatically.

# 2 Common RAII examples in C++
Resource	RAII class
- File -> std::ifstream, std::ofstream
- Dynamically allocated memory -> std::unique_ptr
- Mutex locking -> std::lock_guard, std::unique_lock
- Thread ownership -> std::jthread (C++20)
- OpenCV image memory -> cv::Mat

# 3 RAII with dynamic memory
Without RAII:
```
int* p = new int[1000];

// Process data...
If an exception occurs before delete[], memory leaks.

delete[] p;
```
If an exception occurs before delete[], memory leaks.

With RAII:
```
#include <memory>

auto p = std::make_unique<int[]>(1000);

// Process data...

// Memory automatically released when p goes out of scope
```
You don't need to call delete[] manually.

# 4 RAII with mutexes and threads
Without RAII:
```
queue_mutex.lock();

frame_queue.push(frame);

queue_mutex.unlock();
```
If an exception occurs between lock() and unlock(), the mutex can remain locked, potentially blocking other threads indefinitely.

With RAII:
```
{
    std::lock_guard<std::mutex> lock(queue_mutex);

    frame_queue.push(frame);
}

// Mutex automatically unlocked here
```
Or, when you need to wait on a condition variable:

```
std::unique_lock<std::mutex> lock(queue_mutex);

queue_cond.wait(lock, [&] {
    return !frame_queue.empty() || !ready_to_grab;
});
```
std::unique_lock manages the mutex and supports temporarily unlocking and relocking it during wait().