# ZLToolKit 学习指南

本指南旨在帮助初学者系统地学习和掌握 ZLToolKit 这套轻量级C++11网络编程框架。

## 目录

1. [项目概述](#项目概述)
2. [学习前的准备](#学习前的准备)
3. [推荐学习路径](#推荐学习路径)
4. [核心模块详解](#核心模块详解)
5. [实战练习](#实战练习)
6. [常见问题](#常见问题)
7. [进阶学习](#进阶学习)

---

## 项目概述

ZLToolKit 是一个基于C++11开发的轻量级网络编程框架，具有以下特点：

- **现代化设计**：使用C++11标准，避免裸指针，代码稳定可靠
- **高性能**：采用 epoll+线程池+异步网络IO 模式
- **跨平台**：支持 Linux、macOS、iOS、Android、Windows
- **易于使用**：接口简单，文档完善，适合快速开发

### 项目结构

```
ZLToolKit/
├── src/                    # 源代码目录
│   ├── Network/           # 网络模块
│   ├── Poller/            # 事件轮询模块
│   ├── Thread/            # 线程模块
│   ├── Util/              # 工具模块
│   └── win32/             # Windows平台特定代码
├── tests/                 # 测试示例代码
├── cmake/                 # CMake配置文件
└── README.md             # 项目说明文档
```

---

## 学习前的准备

### 必备知识

1. **C++基础**
   - C++11 特性（智能指针、lambda表达式、右值引用等）
   - 面向对象编程
   - 模板编程基础

2. **网络编程基础**
   - TCP/UDP协议
   - Socket编程概念
   - 基本的网络通信流程

3. **多线程编程**
   - 线程创建和管理
   - 互斥锁、信号量等同步机制
   - 线程安全概念

### 开发环境

#### Linux (推荐)
```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install build-essential cmake git

# 可选：安装 OpenSSL 和 MySQL 客户端库
sudo apt-get install libssl-dev libmysqlclient-dev
```

#### macOS
```bash
# 使用 Homebrew
brew install cmake
brew install openssl
brew install mysql-client
```

#### Windows
- Visual Studio 2017 或更高版本
- CMake-GUI
- 可选：OpenSSL、MySQL客户端库

### 编译项目

```bash
# 克隆仓库
git clone https://github.com/ZLMediaKit/ZLToolKit.git
cd ZLToolKit

# Linux 编译
./build_for_linux.sh

# macOS 编译
./build_for_mac.sh

# 手动编译（通用方法）
mkdir build
cd build
cmake ..
make
```

---

## 推荐学习路径

### 第一阶段：工具模块 (1-2周)

从最基础的工具模块开始，这些模块相对独立，容易理解。

#### 1. 日志系统 (`Util/logger.h`)

**学习目标**：
- 了解如何使用日志系统
- 掌握不同日志级别的使用场景
- 理解异步日志的实现原理

**示例代码**：查看 `tests/test_logger.cpp`

```cpp
#include "Util/logger.h"
using namespace toolkit;

int main() {
    // 初始化日志系统
    Logger::Instance().add(std::make_shared<ConsoleChannel>());
    Logger::Instance().add(std::make_shared<FileChannel>());
    Logger::Instance().setWriter(std::make_shared<AsyncLogWriter>());
    
    // 使用不同级别的日志
    TraceL << "跟踪日志";
    DebugL << "调试日志";
    InfoL << "信息日志";
    WarnL << "警告日志";
    ErrorL << "错误日志";
    
    return 0;
}
```

**深入理解**：
- 阅读 `src/Util/logger.h` 源码
- 理解 Channel（日志通道）的概念
- 学习异步日志写入的实现

#### 2. 配置文件读写 (`Util/mini.h`)

**学习目标**：
- 掌握INI配置文件的读写
- 了解配置管理的最佳实践

**示例代码**：查看 `tests/test_ini.cpp`

```cpp
#include "Util/mini.h"
using namespace toolkit;

int main() {
    mINI ini;
    // 读取配置文件
    ini.parse("config.ini");
    
    // 获取配置项
    string value = ini["section"]["key"];
    
    // 设置配置项
    ini["section"]["key"] = "new_value";
    
    // 保存配置文件
    ini.dump("config.ini");
    
    return 0;
}
```

#### 3. 时间工具 (`Util/TimeTicker.h`)

**学习目标**：
- 掌握代码性能测量方法
- 了解时间统计的应用场景

```cpp
#include "Util/TimeTicker.h"
using namespace toolkit;

void some_function() {
    Ticker ticker;
    
    // 执行一些操作
    // ...
    
    InfoL << "函数执行耗时: " << ticker.elapsedTime() << "ms";
}
```

#### 4. 资源池 (`Util/ResourcePool.h`)

**学习目标**：
- 理解对象池的概念和优势
- 掌握智能指针管理资源的方法

**示例代码**：查看 `tests/test_resourcePool.cpp`

```cpp
#include "Util/ResourcePool.h"
using namespace toolkit;

// 定义资源类型
class MyResource {
public:
    MyResource() { InfoL << "创建资源"; }
    ~MyResource() { InfoL << "销毁资源"; }
    void use() { InfoL << "使用资源"; }
};

int main() {
    // 创建资源池
    ResourcePool<MyResource> pool;
    pool.setSize(10); // 设置池大小
    
    // 从池中获取资源
    auto resource = pool.obtain();
    resource->use();
    
    // 资源会自动归还到池中
    return 0;
}
```

### 第二阶段：线程模块 (1-2周)

理解多线程编程是使用该框架的关键。

#### 1. 线程池 (`Thread/ThreadPool.h`)

**学习目标**：
- 理解线程池的工作原理
- 掌握异步任务的提交和执行
- 学习线程同步机制

**示例代码**：查看 `tests/test_threadPool.cpp`

```cpp
#include "Thread/ThreadPool.h"
using namespace toolkit;

int main() {
    // 创建线程池
    ThreadPool pool(4, ThreadPool::PRIORITY_HIGHEST);
    
    // 提交异步任务
    pool.async([]() {
        InfoL << "在后台线程执行任务";
    });
    
    // 提交同步任务（等待任务完成）
    auto result = pool.async_first([]() {
        return 42;
    }).get();
    
    InfoL << "同步任务结果: " << result;
    
    return 0;
}
```

**深入理解**：
- 阅读 `src/Thread/ThreadPool.h` 源码
- 理解任务队列的实现
- 学习线程优先级的设置

#### 2. 信号量 (`Thread/semaphore.h`)

**学习目标**：
- 掌握信号量的使用场景
- 理解线程同步的方法

**示例代码**：查看 `tests/test_semaphore.cpp`

```cpp
#include "Thread/semaphore.h"
using namespace toolkit;

int main() {
    semaphore sem;
    
    // 在一个线程中等待
    std::thread t1([&]() {
        sem.wait(); // 等待信号
        InfoL << "收到信号，继续执行";
    });
    
    // 在另一个线程中发送信号
    std::thread t2([&]() {
        std::this_thread::sleep_for(std::chrono::seconds(1));
        sem.post(); // 发送信号
        InfoL << "信号已发送";
    });
    
    t1.join();
    t2.join();
    
    return 0;
}
```

#### 3. 定时器 (`Poller/Timer.h`)

**学习目标**：
- 掌握定时任务的实现
- 理解事件驱动的定时器机制

**示例代码**：查看 `tests/test_timer.cpp`

```cpp
#include "Poller/Timer.h"
#include "Poller/EventPoller.h"
using namespace toolkit;

int main() {
    // 获取事件轮询器
    auto poller = EventPollerPool::Instance().getPoller();
    
    // 创建延迟任务（1秒后执行）
    auto timer = poller->doDelayTask(1000, []() {
        InfoL << "1秒后执行";
        return 0; // 返回0表示不再重复
    });
    
    // 创建重复任务（每2秒执行一次）
    auto repeat_timer = poller->doDelayTask(2000, []() {
        InfoL << "每2秒执行一次";
        return 2000; // 返回延迟时间，继续定时
    });
    
    // 取消定时器
    // timer->cancel();
    
    sleep(10);
    return 0;
}
```

### 第三阶段：网络模块 (2-3周)

这是框架的核心部分，需要重点学习。

#### 1. TCP 服务器开发

**学习目标**：
- 掌握TCP服务器的基本架构
- 理解Session（会话）的概念
- 学习如何处理客户端连接和数据

**示例代码**：查看 `tests/test_tcpEchoServer.cpp`

```cpp
#include "Network/TcpServer.h"
#include "Network/Session.h"
using namespace toolkit;

// 自定义会话类
class EchoSession : public Session {
public:
    EchoSession(const Socket::Ptr &sock) : Session(sock) {
        DebugL << "新客户端连接";
    }
    
    ~EchoSession() {
        DebugL << "客户端断开";
    }
    
    // 接收到数据
    void onRecv(const Buffer::Ptr &buf) override {
        InfoL << "收到数据: " << buf->size() << " 字节";
        // 回显数据
        send(buf);
    }
    
    // 发生错误或断开连接
    void onError(const SockException &err) override {
        WarnL << err.what();
    }
    
    // 定期管理（如超时检查）
    void onManager() override {
        // 可以实现超时逻辑
    }
};

int main() {
    // 初始化日志
    Logger::Instance().add(std::make_shared<ConsoleChannel>());
    Logger::Instance().setWriter(std::make_shared<AsyncLogWriter>());
    
    // 创建TCP服务器
    TcpServer::Ptr server = std::make_shared<TcpServer>();
    server->start<EchoSession>(9000); // 监听9000端口
    
    InfoL << "TCP Echo服务器启动在端口9000";
    
    // 保持运行
    static semaphore sem;
    signal(SIGINT, [](int) { sem.post(); });
    sem.wait();
    
    return 0;
}
```

**测试服务器**：
```bash
# 编译后运行服务器
./build/bin/test_tcpEchoServer

# 使用telnet测试
telnet localhost 9000
```

#### 2. TCP 客户端开发

**学习目标**：
- 掌握TCP客户端的实现
- 理解异步连接和数据发送
- 学习断线重连机制

**示例代码**：查看 `tests/test_tcpClient.cpp`

```cpp
#include "Network/TcpClient.h"
using namespace toolkit;

class MyTcpClient : public TcpClient {
public:
    MyTcpClient() : TcpClient() {}
    
    // 连接成功
    void onConnect(const SockException &ex) override {
        if (ex) {
            WarnL << "连接失败: " << ex.what();
        } else {
            InfoL << "连接成功";
            // 发送数据
            send("Hello Server!");
        }
    }
    
    // 接收到数据
    void onRecv(const Buffer::Ptr &buf) override {
        InfoL << "收到数据: " << buf->data();
    }
    
    // 连接断开或错误
    void onError(const SockException &ex) override {
        WarnL << "连接错误: " << ex.what();
    }
};

int main() {
    Logger::Instance().add(std::make_shared<ConsoleChannel>());
    
    MyTcpClient client;
    client.startConnect("127.0.0.1", 9000); // 连接到服务器
    
    sleep(10);
    return 0;
}
```

#### 3. UDP 通信

**学习目标**：
- 理解UDP通信的特点
- 掌握UDP服务器和客户端的实现
- 学习无连接通信的处理方式

**示例代码**：查看 `tests/test_udpEchoServer.cpp` 和 `tests/test_udpClient.cpp`

```cpp
// UDP服务器示例
#include "Network/Socket.h"
using namespace toolkit;

int main() {
    Logger::Instance().add(std::make_shared<ConsoleChannel>());
    
    Socket::Ptr sock = Socket::createSocket();
    sock->bindUdpSock(9001); // 绑定UDP端口
    
    sock->setOnRead([](const Buffer::Ptr &buf, struct sockaddr *addr, int addr_len) {
        InfoL << "收到UDP数据: " << buf->size() << " 字节";
        // 回复数据
        // sock->send(buf, addr, addr_len);
    });
    
    static semaphore sem;
    signal(SIGINT, [](int) { sem.post(); });
    sem.wait();
    
    return 0;
}
```

### 第四阶段：高级特性 (1-2周)

#### 1. SSL/TLS 支持

**学习目标**：
- 掌握安全通信的实现
- 理解SSL握手过程
- 学习证书的使用

**示例代码**：查看 `tests/test_ssl.cpp`

需要先安装OpenSSL库，编译时启用`ENABLE_OPENSSL`选项。

#### 2. MySQL 连接池

**学习目标**：
- 掌握数据库连接池的使用
- 学习SQL语句的生成和执行
- 理解异步数据库操作

**示例代码**：查看 `tests/test_sql.cpp`

```cpp
#include "Util/SqlPool.h"
using namespace toolkit;

int main() {
    // 初始化MySQL连接池
    SqlPool::Instance().Init("localhost", 3306, "dbname", 
                             "username", "password");
    
    // 执行SQL查询
    auto writer = SqlPool::Instance().obtain();
    
    // 使用占位符方式生成SQL
    vector<Value> values;
    values.emplace_back("test_value");
    
    *writer << "SELECT * FROM table WHERE field = ?" << values << endl;
    
    return 0;
}
```

#### 3. 消息广播器 (`Util/NoticeCenter.h`)

**学习目标**：
- 理解观察者模式
- 掌握事件驱动编程
- 学习模块间通信机制

**示例代码**：查看 `tests/test_noticeCenter.cpp`

```cpp
#include "Util/NoticeCenter.h"
using namespace toolkit;

int main() {
    // 订阅消息
    NoticeCenter::Instance().addListener(nullptr, "event_name", 
        [](int arg1, const string &arg2) {
            InfoL << "收到事件: " << arg1 << ", " << arg2;
        });
    
    // 发布消息
    NoticeCenter::Instance().emitEvent("event_name", 100, string("hello"));
    
    return 0;
}
```

---

## 核心模块详解

### Network 模块

**主要类**：
- `Socket`: 套接字封装，支持TCP/UDP
- `TcpServer`: TCP服务器模板类
- `TcpClient`: TCP客户端基类
- `Session`: TCP/UDP会话基类
- `Buffer`: 缓冲区管理

**设计模式**：
- 工厂模式（创建Socket）
- 模板模式（TcpServer）
- 观察者模式（事件回调）

### Poller 模块

**主要类**：
- `EventPoller`: 事件轮询器（epoll/select/kqueue）
- `Timer`: 定时器实现
- `Pipe`: 管道封装

**工作原理**：
- 使用IO多路复用技术
- 支持跨平台（epoll、kqueue、select）
- 事件驱动模型

### Thread 模块

**主要类**：
- `ThreadPool`: 线程池
- `TaskQueue`: 任务队列
- `semaphore`: 信号量
- `WorkThreadPool`: 工作线程池管理

**关键概念**：
- 线程安全
- 任务调度
- 异步执行

### Util 模块

**工具集合**：
- 日志系统（logger）
- 配置管理（mini）
- 文件操作（File）
- 时间工具（TimeTicker）
- 加密工具（MD5、SHA1、base64）
- 资源池（ResourcePool）
- 环形缓冲（RingBuffer）

---

## 实战练习

### 练习1：简单的聊天服务器

**目标**：实现一个多客户端聊天室

**要求**：
1. 服务器监听指定端口
2. 多个客户端可以连接
3. 一个客户端发送的消息广播给所有其他客户端
4. 客户端连接和断开时通知其他客户端

**提示**：
- 使用 `TcpServer` 和 `Session`
- 维护一个客户端列表
- 实现消息广播逻辑

### 练习2：HTTP 简易服务器

**目标**：实现一个简单的HTTP服务器

**要求**：
1. 解析HTTP请求
2. 返回静态文件
3. 支持GET和POST方法
4. 实现简单的路由功能

**提示**：
- 继承 `Session` 类
- 解析HTTP协议头
- 处理不同的HTTP方法

### 练习3：文件传输工具

**目标**：实现客户端-服务器文件传输

**要求**：
1. 客户端可以上传文件到服务器
2. 客户端可以从服务器下载文件
3. 支持断点续传
4. 显示传输进度

**提示**：
- 使用 `TcpClient` 和 `TcpServer`
- 实现自定义协议
- 处理大文件分块传输

---

## 常见问题

### Q1: 编译错误：找不到OpenSSL或MySQL

**解决方案**：
```bash
# 方法1：安装依赖库
sudo apt-get install libssl-dev libmysqlclient-dev

# 方法2：禁用相关功能
cmake -DENABLE_OPENSSL=OFF -DENABLE_MYSQL=OFF ..
```

### Q2: 如何调试网络程序？

**建议**：
1. 启用详细日志：设置日志级别为 Trace
2. 使用网络抓包工具（Wireshark）
3. 使用telnet或netcat测试连接
4. 查看系统的网络连接状态（netstat）

```cpp
// 设置日志级别
Logger::Instance().setLevel(LogLevel::LTrace);
```

### Q3: 线程池任务执行顺序

**说明**：
- `async()`: 异步执行，不保证顺序
- `async_first()`: 优先执行
- 同一线程内的任务按提交顺序执行

### Q4: 内存泄漏如何检查？

**工具**：
```bash
# Linux下使用valgrind
valgrind --leak-check=full ./your_program

# 启用地址检测（编译时）
cmake -DCMAKE_CXX_FLAGS="-fsanitize=address" ..
```

### Q5: 如何处理大量并发连接？

**优化建议**：
1. 调整系统文件描述符限制
2. 使用多个EventPoller（线程）
3. 优化Session的内存占用
4. 实现连接池复用

```bash
# 增加文件描述符限制
ulimit -n 65535
```

---

## 进阶学习

### 1. 源码阅读建议

**推荐阅读顺序**：

1. **基础组件**
   - `src/Util/util.h` - 基础工具函数
   - `src/Util/logger.h` - 日志系统
   - `src/Poller/Timer.h` - 定时器

2. **线程管理**
   - `src/Thread/ThreadPool.h` - 线程池
   - `src/Thread/TaskQueue.h` - 任务队列
   - `src/Thread/semaphore.h` - 信号量

3. **网络核心**
   - `src/Poller/EventPoller.h` - 事件轮询
   - `src/Network/Socket.h` - Socket封装
   - `src/Network/TcpServer.h` - TCP服务器
   - `src/Network/Session.h` - 会话管理

### 2. 性能优化技巧

**关键点**：
1. 减少内存拷贝（使用Buffer::Ptr）
2. 合理设置线程池大小
3. 使用对象池减少内存分配
4. 批量处理网络数据

### 3. 最佳实践

**代码规范**：
```cpp
// 1. 使用智能指针
Socket::Ptr sock = Socket::createSocket();

// 2. 使用RAII管理资源
onceToken token([]() { /* 初始化 */ }, []() { /* 清理 */ });

// 3. 异步错误处理
client->setOnError([](const SockException &ex) {
    ErrorL << "错误: " << ex.what();
});

// 4. 使用日志记录关键信息
InfoL << "服务器启动在端口: " << port;
```

### 4. 相关项目学习

**推荐项目**：
- [ZLMediaKit](https://github.com/ZLMediaKit/ZLMediaKit): 基于ZLToolKit的流媒体服务器
  - 学习实际项目中的应用
  - 了解更复杂的协议实现（RTSP、RTMP、HLS等）
  - 参考生产环境的代码组织

### 5. 社区资源

**获取帮助**：
- GitHub Issues: 查看已有问题和解决方案
- QQ群: 542509000
- 邮箱: 1213642868@qq.com（建议先查看Issues）

**贡献代码**：
1. Fork项目
2. 创建特性分支
3. 提交代码（附带测试）
4. 发起Pull Request

---

## 学习检查清单

完成以下任务，确保掌握ZLToolKit的核心功能：

### 基础阶段
- [ ] 成功编译并运行项目
- [ ] 运行所有测试示例（tests目录下）
- [ ] 理解日志系统的使用
- [ ] 掌握配置文件的读写
- [ ] 使用线程池执行异步任务

### 进阶阶段
- [ ] 实现一个简单的Echo服务器
- [ ] 实现一个TCP客户端连接到Echo服务器
- [ ] 实现UDP通信示例
- [ ] 使用定时器实现周期性任务
- [ ] 使用资源池优化对象管理

### 高级阶段
- [ ] 实现自定义协议的服务器
- [ ] 处理粘包、半包问题
- [ ] 实现断线重连机制
- [ ] 使用SSL实现安全通信（可选）
- [ ] 使用MySQL连接池（可选）

### 实战项目
- [ ] 完成聊天服务器项目
- [ ] 实现文件传输工具
- [ ] 开发简易HTTP服务器
- [ ] 优化性能，处理高并发场景

---

## 总结

ZLToolKit是一个设计优雅、功能强大的C++网络库。通过系统学习，你将能够：

1. **快速开发网络应用**：简洁的API大大降低开发难度
2. **编写高性能代码**：充分利用异步IO和线程池
3. **跨平台部署**：一次编写，多平台运行
4. **构建复杂系统**：丰富的工具库支持各种应用场景

**学习建议**：
- 循序渐进，从简单到复杂
- 多动手实践，运行和修改示例代码
- 阅读源码，理解设计思想
- 参与社区，分享经验和问题

**下一步行动**：
1. 按照学习路径开始第一阶段的学习
2. 每完成一个阶段，尝试一个实战练习
3. 遇到问题先查阅文档和Issues，再寻求帮助
4. 学习ZLMediaKit项目，了解实际应用

祝你学习愉快！🚀

---

## 附录

### A. 常用代码片段

#### 快速启动TCP服务器
```cpp
#include "Util/logger.h"
#include "Network/TcpServer.h"
#include "Network/Session.h"

class MySession : public Session {
public:
    MySession(const Socket::Ptr &sock) : Session(sock) {}
    void onRecv(const Buffer::Ptr &buf) override { send(buf); }
    void onError(const SockException &ex) override { WarnL << ex; }
};

int main() {
    Logger::Instance().add(std::make_shared<ConsoleChannel>());
    TcpServer::Ptr server = std::make_shared<TcpServer>();
    server->start<MySession>(9000);
    pause(); // 保持运行
    return 0;
}
```

#### 异步HTTP请求（示例框架）
```cpp
// 这是一个框架示例，需要自己实现HTTP协议解析
class HttpClient : public TcpClient {
public:
    void request(const string &url) {
        // 解析URL，连接服务器，发送HTTP请求
        // 实现留给读者作为练习
    }
    void onRecv(const Buffer::Ptr &buf) override {
        // 解析HTTP响应
    }
};
```

### B. 术语表

- **Session（会话）**：表示一个客户端连接的生命周期
- **Poller（轮询器）**：负责监听和分发IO事件
- **Buffer（缓冲区）**：用于存储网络数据的容器
- **EventPoller（事件轮询器）**：基于epoll/select的事件循环
- **Socket（套接字）**：网络通信的端点
- **ResourcePool（资源池）**：对象复用机制，减少内存分配

### C. 参考链接

- [项目主页](https://github.com/ZLMediaKit/ZLToolKit)
- [ZLMediaKit项目](https://github.com/ZLMediaKit/ZLMediaKit)
- [性能测试报告](https://github.com/ZLMediaKit/ZLMediaKit/blob/master/benchmark.md)
- [C++11参考](https://en.cppreference.com/w/cpp/11)
