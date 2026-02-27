---
title: "Complete C++ Mastery Part 13: Domain-Specific Topics"
date: 2024-02-21 10:00:00 +0000
categories: [C++, Programming, Domains]
tags: [cpp, systems-programming, embedded, networking, graphics, game-dev]
---

# Complete C++ Mastery Part 13: Domain-Specific Topics

## Introduction

Welcome to Part 13 of the Complete C++ Mastery series. C++ excels in various domains: systems programming, embedded systems, game development, graphics, networking, and high-performance computing. This part explores domain-specific techniques, libraries, and best practices that make C++ the go-to language for performance-critical applications.

Each domain has unique requirements and constraints. Understanding these specializations helps you apply C++ effectively in your target domain.

**Prerequisites:** This part assumes knowledge from Parts 1-12, especially performance optimization and modern C++ features.

---

## 13.1 Systems Programming

### Low-Level Memory Access

```cpp
#include <iostream>
#include <cstdint>
#include <bit>

// Memory-mapped I/O (typical in embedded/driver development)
class MemoryMappedRegister {
    volatile uint32_t* reg;
    
public:
    explicit MemoryMappedRegister(uintptr_t address)
        : reg(reinterpret_cast<volatile uint32_t*>(address)) {}
    
    void write(uint32_t value) {
        *reg = value;  // volatile prevents optimization
    }
    
    uint32_t read() const {
        return *reg;
    }
    
    void set_bit(int bit) {
        *reg |= (1u << bit);
    }
    
    void clear_bit(int bit) {
        *reg &= ~(1u << bit);
    }
};

// Bit manipulation
struct StatusRegister {
    uint32_t value;
    
    bool get_flag(int position) const {
        return (value >> position) & 1;
    }
    
    void set_flag(int position, bool state) {
        if (state) {
            value |= (1u << position);
        } else {
            value &= ~(1u << position);
        }
    }
};

// Packed structures (no padding)
struct __attribute__((packed)) PacketHeader {
    uint16_t type;
    uint16_t length;
    uint32_t timestamp;
};  // Exactly 8 bytes, no padding

static_assert(sizeof(PacketHeader) == 8, "Unexpected padding");
```

### Platform-Specific Code

```cpp
#include <iostream>

// Conditional compilation
#ifdef _WIN32
    #include <windows.h>
    using thread_handle = HANDLE;
#elif defined(__linux__)
    #include <pthread.h>
    using thread_handle = pthread_t;
#else
    #error "Unsupported platform"
#endif

// Platform abstraction layer
class Thread {
    thread_handle handle;
    
public:
    template<typename Func>
    void create(Func&& func) {
#ifdef _WIN32
        handle = CreateThread(nullptr, 0, 
                             [](void* param) -> DWORD {
                                 (*static_cast<Func*>(param))();
                                 return 0;
                             }, &func, 0, nullptr);
#elif defined(__linux__)
        pthread_create(&handle, nullptr,
                      [](void* param) -> void* {
                          (*static_cast<Func*>(param))();
                          return nullptr;
                      }, &func);
#endif
    }
    
    void join() {
#ifdef _WIN32
        WaitForSingleObject(handle, INFINITE);
        CloseHandle(handle);
#elif defined(__linux__)
        pthread_join(handle, nullptr);
#endif
    }
};
```

### System Calls and Error Handling

```cpp
#include <system_error>
#include <cerrno>

// RAII wrapper for file descriptor
class FileDescriptor {
    int fd = -1;
    
public:
    explicit FileDescriptor(const char* path, int flags) {
        fd = open(path, flags);
        if (fd < 0) {
            throw std::system_error(errno, std::system_category(),
                                   "Failed to open file");
        }
    }
    
    ~FileDescriptor() {
        if (fd >= 0) {
            close(fd);
        }
    }
    
    FileDescriptor(const FileDescriptor&) = delete;
    FileDescriptor& operator=(const FileDescriptor&) = delete;
    
    FileDescriptor(FileDescriptor&& other) noexcept : fd(other.fd) {
        other.fd = -1;
    }
    
    int get() const { return fd; }
};
```

---

## 13.2 Embedded Systems

### Memory Constraints

```cpp
#include <array>
#include <cstdint>

// Fixed-size containers for predictable memory
template<typename T, size_t N>
class StaticVector {
    std::array<T, N> data;
    size_t size_ = 0;
    
public:
    void push_back(const T& value) {
        if (size_ < N) {
            data[size_++] = value;
        }
    }
    
    T& operator[](size_t index) { return data[index]; }
    const T& operator[](size_t index) const { return data[index]; }
    
    size_t size() const { return size_; }
    size_t capacity() const { return N; }
    bool full() const { return size_ == N; }
    
    // No dynamic allocation!
};

// Memory pool for deterministic allocation
template<typename T, size_t N>
class MemoryPool {
    alignas(T) uint8_t storage[N * sizeof(T)];
    bool used[N] = {};
    
public:
    T* allocate() {
        for (size_t i = 0; i < N; ++i) {
            if (!used[i]) {
                used[i] = true;
                return reinterpret_cast<T*>(&storage[i * sizeof(T)]);
            }
        }
        return nullptr;  // Pool exhausted
    }
    
    void deallocate(T* ptr) {
        size_t index = (reinterpret_cast<uint8_t*>(ptr) - storage) / sizeof(T);
        if (index < N) {
            used[index] = false;
        }
    }
};
```

### Real-Time Constraints

```cpp
#include <chrono>
#include <cstdint>

// Deterministic timing
class Timer {
    using Clock = std::chrono::steady_clock;
    Clock::time_point start;
    std::chrono::milliseconds period;
    
public:
    Timer(std::chrono::milliseconds ms) : period(ms), start(Clock::now()) {}
    
    bool expired() const {
        return Clock::now() - start >= period;
    }
    
    void reset() {
        start = Clock::now();
    }
};

// Interrupt-safe ring buffer
template<typename T, size_t N>
class RingBuffer {
    std::array<T, N> buffer;
    volatile size_t read_idx = 0;
    volatile size_t write_idx = 0;
    
public:
    // Called from interrupt
    bool push(const T& value) {
        size_t next = (write_idx + 1) % N;
        if (next == read_idx) return false;  // Full
        
        buffer[write_idx] = value;
        write_idx = next;
        return true;
    }
    
    // Called from main loop
    bool pop(T& value) {
        if (read_idx == write_idx) return false;  // Empty
        
        value = buffer[read_idx];
        read_idx = (read_idx + 1) % N;
        return true;
    }
};
```

### Hardware Abstraction Layer

```cpp
#include <cstdint>

// GPIO abstraction
class GPIO {
public:
    enum class Mode { Input, Output, Alternate };
    enum class Pull { None, Up, Down };
    
    virtual ~GPIO() = default;
    virtual void set_mode(Mode mode) = 0;
    virtual void set_pull(Pull pull) = 0;
    virtual void write(bool value) = 0;
    virtual bool read() const = 0;
};

// Platform-specific implementation
class STM32_GPIO : public GPIO {
    volatile uint32_t* base_addr;
    uint8_t pin;
    
public:
    STM32_GPIO(uint32_t port, uint8_t pin_num)
        : base_addr(reinterpret_cast<uint32_t*>(port)),
          pin(pin_num) {}
    
    void set_mode(Mode mode) override {
        // STM32-specific register manipulation
    }
    
    void write(bool value) override {
        if (value) {
            base_addr[4] = (1u << pin);  // BSRR register
        } else {
            base_addr[4] = (1u << (pin + 16));
        }
    }
    
    bool read() const override {
        return (base_addr[2] >> pin) & 1;  // IDR register
    }
    
    void set_pull(Pull pull) override { /* Implementation */ }
};
```

---

## 13.3 Game Development

### Entity Component System (ECS)

```cpp
#include <vector>
#include <memory>
#include <unordered_map>
#include <typeindex>

// Component base
struct Component {
    virtual ~Component() = default;
};

// Concrete components
struct Transform : Component {
    float x = 0, y = 0, z = 0;
    float rotation = 0;
};

struct Velocity : Component {
    float vx = 0, vy = 0, vz = 0;
};

struct Sprite : Component {
    std::string texture_name;
    int width = 0, height = 0;
};

// Entity (just an ID)
using EntityID = uint32_t;

// ECS Manager
class ECS {
    EntityID next_id = 0;
    std::unordered_map<EntityID, 
        std::unordered_map<std::type_index, std::unique_ptr<Component>>> entities;
    
public:
    EntityID create_entity() {
        return next_id++;
    }
    
    template<typename T, typename... Args>
    T* add_component(EntityID entity, Args&&... args) {
        auto component = std::make_unique<T>(std::forward<Args>(args)...);
        T* ptr = component.get();
        entities[entity][std::type_index(typeid(T))] = std::move(component);
        return ptr;
    }
    
    template<typename T>
    T* get_component(EntityID entity) {
        auto& components = entities[entity];
        auto it = components.find(std::type_index(typeid(T)));
        if (it != components.end()) {
            return static_cast<T*>(it->second.get());
        }
        return nullptr;
    }
    
    void destroy_entity(EntityID entity) {
        entities.erase(entity);
    }
};

// System
class MovementSystem {
public:
    void update(ECS& ecs, float dt) {
        // Process all entities with Transform and Velocity
        // (simplified - real ECS would have efficient iteration)
    }
};
```

### Game Loop

```cpp
#include <chrono>
#include <thread>

class GameEngine {
    bool running = true;
    const double fps = 60.0;
    const double frame_time = 1.0 / fps;
    
public:
    void run() {
        using Clock = std::chrono::high_resolution_clock;
        using Duration = std::chrono::duration<double>;
        
        auto last_time = Clock::now();
        double accumulator = 0.0;
        
        while (running) {
            auto current_time = Clock::now();
            double delta = Duration(current_time - last_time).count();
            last_time = current_time;
            
            accumulator += delta;
            
            // Fixed timestep updates
            while (accumulator >= frame_time) {
                update(frame_time);
                accumulator -= frame_time;
            }
            
            // Render with interpolation
            double alpha = accumulator / frame_time;
            render(alpha);
            
            // Frame limiting
            auto frame_duration = Clock::now() - current_time;
            if (frame_duration < Duration(frame_time)) {
                std::this_thread::sleep_for(Duration(frame_time) - frame_duration);
            }
        }
    }
    
private:
    void update(double dt) {
        // Update game logic
    }
    
    void render(double alpha) {
        // Render with interpolation
    }
};
```

### Object Pooling

```cpp
#include <vector>
#include <queue>

template<typename T>
class ObjectPool {
    std::vector<T> objects;
    std::queue<T*> available;
    
public:
    ObjectPool(size_t initial_size) {
        objects.reserve(initial_size);
        for (size_t i = 0; i < initial_size; ++i) {
            objects.emplace_back();
            available.push(&objects.back());
        }
    }
    
    T* acquire() {
        if (available.empty()) {
            objects.emplace_back();
            return &objects.back();
        }
        
        T* obj = available.front();
        available.pop();
        return obj;
    }
    
    void release(T* obj) {
        // Reset object state if needed
        available.push(obj);
    }
};

// Usage
struct Bullet {
    float x, y;
    float velocity_x, velocity_y;
    bool active = false;
};

ObjectPool<Bullet> bullet_pool(100);  // Pre-allocate 100 bullets
```

---

## 13.4 Graphics Programming

### OpenGL Integration

```cpp
#include <vector>
#include <memory>

// Vertex data
struct Vertex {
    float x, y, z;
    float r, g, b;
};

// RAII wrapper for OpenGL buffer
class VertexBuffer {
    unsigned int vbo = 0;
    
public:
    VertexBuffer() {
        glGenBuffers(1, &vbo);
    }
    
    ~VertexBuffer() {
        if (vbo) {
            glDeleteBuffers(1, &vbo);
        }
    }
    
    VertexBuffer(const VertexBuffer&) = delete;
    VertexBuffer& operator=(const VertexBuffer&) = delete;
    
    VertexBuffer(VertexBuffer&& other) noexcept : vbo(other.vbo) {
        other.vbo = 0;
    }
    
    void bind() const {
        glBindBuffer(GL_ARRAY_BUFFER, vbo);
    }
    
    void upload(const std::vector<Vertex>& vertices) {
        bind();
        glBufferData(GL_ARRAY_BUFFER,
                    vertices.size() * sizeof(Vertex),
                    vertices.data(),
                    GL_STATIC_DRAW);
    }
};

// Shader management
class Shader {
    unsigned int program = 0;
    
public:
    Shader(const char* vertex_src, const char* fragment_src) {
        // Compile and link shaders
        program = glCreateProgram();
        // ... compilation code
    }
    
    ~Shader() {
        if (program) {
            glDeleteProgram(program);
        }
    }
    
    void use() const {
        glUseProgram(program);
    }
    
    void set_uniform(const char* name, float value) {
        int location = glGetUniformLocation(program, name);
        glUniform1f(location, value);
    }
};
```

### Vulkan Patterns

```cpp
#include <vulkan/vulkan.h>
#include <vector>

// RAII wrapper for Vulkan resources
class VulkanBuffer {
    VkDevice device;
    VkBuffer buffer = VK_NULL_HANDLE;
    VkDeviceMemory memory = VK_NULL_HANDLE;
    
public:
    VulkanBuffer(VkDevice dev, VkDeviceSize size, VkBufferUsageFlags usage)
        : device(dev) {
        
        VkBufferCreateInfo buffer_info = {};
        buffer_info.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
        buffer_info.size = size;
        buffer_info.usage = usage;
        
        vkCreateBuffer(device, &buffer_info, nullptr, &buffer);
        
        // Allocate memory
        VkMemoryRequirements mem_reqs;
        vkGetBufferMemoryRequirements(device, buffer, &mem_reqs);
        
        VkMemoryAllocateInfo alloc_info = {};
        alloc_info.sType = VK_STRUCTURE_TYPE_MEMORY_ALLOCATE_INFO;
        alloc_info.allocationSize = mem_reqs.size;
        // ... find memory type
        
        vkAllocateMemory(device, &alloc_info, nullptr, &memory);
        vkBindBufferMemory(device, buffer, memory, 0);
    }
    
    ~VulkanBuffer() {
        if (buffer) vkDestroyBuffer(device, buffer, nullptr);
        if (memory) vkFreeMemory(device, memory, nullptr);
    }
    
    VkBuffer get() const { return buffer; }
};
```

---

## 13.5 Networking

### Socket Programming

```cpp
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>
#include <cstring>

// RAII socket wrapper
class Socket {
    int fd = -1;
    
public:
    Socket() {
        fd = socket(AF_INET, SOCK_STREAM, 0);
        if (fd < 0) {
            throw std::runtime_error("Failed to create socket");
        }
    }
    
    ~Socket() {
        if (fd >= 0) {
            close(fd);
        }
    }
    
    Socket(const Socket&) = delete;
    Socket& operator=(const Socket&) = delete;
    
    void connect(const char* host, uint16_t port) {
        sockaddr_in addr = {};
        addr.sin_family = AF_INET;
        addr.sin_port = htons(port);
        inet_pton(AF_INET, host, &addr.sin_addr);
        
        if (::connect(fd, (sockaddr*)&addr, sizeof(addr)) < 0) {
            throw std::runtime_error("Connection failed");
        }
    }
    
    void bind(uint16_t port) {
        sockaddr_in addr = {};
        addr.sin_family = AF_INET;
        addr.sin_addr.s_addr = INADDR_ANY;
        addr.sin_port = htons(port);
        
        if (::bind(fd, (sockaddr*)&addr, sizeof(addr)) < 0) {
            throw std::runtime_error("Bind failed");
        }
    }
    
    void listen(int backlog = 10) {
        if (::listen(fd, backlog) < 0) {
            throw std::runtime_error("Listen failed");
        }
    }
    
    Socket accept() {
        int client_fd = ::accept(fd, nullptr, nullptr);
        if (client_fd < 0) {
            throw std::runtime_error("Accept failed");
        }
        return Socket(client_fd);
    }
    
    ssize_t send(const void* data, size_t size) {
        return ::send(fd, data, size, 0);
    }
    
    ssize_t receive(void* buffer, size_t size) {
        return ::recv(fd, buffer, size, 0);
    }
    
private:
    explicit Socket(int client_fd) : fd(client_fd) {}
};
```

### Asynchronous I/O with Boost.Asio

```cpp
#include <boost/asio.hpp>
#include <iostream>
#include <memory>

using boost::asio::ip::tcp;

class Session : public std::enable_shared_from_this<Session> {
    tcp::socket socket;
    std::array<char, 1024> buffer;
    
public:
    Session(tcp::socket sock) : socket(std::move(sock)) {}
    
    void start() {
        do_read();
    }
    
private:
    void do_read() {
        auto self = shared_from_this();
        socket.async_read_some(
            boost::asio::buffer(buffer),
            [this, self](boost::system::error_code ec, size_t length) {
                if (!ec) {
                    do_write(length);
                }
            }
        );
    }
    
    void do_write(size_t length) {
        auto self = shared_from_this();
        boost::asio::async_write(
            socket,
            boost::asio::buffer(buffer, length),
            [this, self](boost::system::error_code ec, size_t) {
                if (!ec) {
                    do_read();
                }
            }
        );
    }
};

class Server {
    tcp::acceptor acceptor;
    
public:
    Server(boost::asio::io_context& io, uint16_t port)
        : acceptor(io, tcp::endpoint(tcp::v4(), port)) {
        do_accept();
    }
    
private:
    void do_accept() {
        acceptor.async_accept(
            [this](boost::system::error_code ec, tcp::socket socket) {
                if (!ec) {
                    std::make_shared<Session>(std::move(socket))->start();
                }
                do_accept();
            }
        );
    }
};
```

---

## 13.6 High-Performance Computing

### Parallel Algorithms

```cpp
#include <vector>
#include <algorithm>
#include <execution>
#include <numeric>

void parallel_examples() {
    std::vector<int> vec(1000000);
    std::iota(vec.begin(), vec.end(), 0);
    
    // Parallel sort
    std::sort(std::execution::par, vec.begin(), vec.end());
    
    // Parallel transform
    std::vector<int> result(vec.size());
    std::transform(std::execution::par,
                   vec.begin(), vec.end(),
                   result.begin(),
                   [](int x) { return x * x; });
    
    // Parallel reduce
    auto sum = std::reduce(std::execution::par,
                          vec.begin(), vec.end(),
                          0);
}
```

### SIMD Operations

```cpp
#include <immintrin.h>

// Matrix multiplication with AVX
void matrix_multiply_avx(const float* A, const float* B, float* C,
                        int M, int N, int K) {
    for (int i = 0; i < M; ++i) {
        for (int j = 0; j < N; j += 8) {  // Process 8 elements at once
            __m256 sum = _mm256_setzero_ps();
            
            for (int k = 0; k < K; ++k) {
                __m256 a = _mm256_set1_ps(A[i * K + k]);
                __m256 b = _mm256_loadu_ps(&B[k * N + j]);
                sum = _mm256_fmadd_ps(a, b, sum);
            }
            
            _mm256_storeu_ps(&C[i * N + j], sum);
        }
    }
}
```

---

## Conclusion

Part 13 explored domain-specific C++ applications:

1. **Systems Programming** - Low-level memory, platform abstraction
2. **Embedded Systems** - Memory constraints, real-time, HAL
3. **Game Development** - ECS, game loops, object pooling
4. **Graphics** - OpenGL/Vulkan RAII wrappers
5. **Networking** - Sockets, async I/O
6. **HPC** - Parallel algorithms, SIMD

**Key Takeaways:**
- Each domain has unique requirements
- RAII applies universally
- Platform abstraction enables portability
- Performance matters in all domains

**Domain Selection Guide:**
- Systems: Low-level control needed
- Embedded: Resource constraints
- Games: Real-time performance
- Graphics: GPU programming
- Networking: Concurrent I/O
- HPC: Maximum performance

**Next:** Part 14 will cover C++ Standards Evolution.

---

## References

- **Game Programming Patterns** by Robert Nystrom
- **Vulkan Programming Guide**
- **Boost.Asio Documentation**

---

*This blog post is part of the Complete C++ Mastery series.*
