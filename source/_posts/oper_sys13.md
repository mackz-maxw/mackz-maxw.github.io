---
title: 操作系统基础 | 信号量和条件变量
categories: os basic
lang: zh-CN
---

### 🚦 **信号量 (Semaphore) - 停车场模型**
**口诀**：**“计数资源，进出加减”**  
- 像一个停车场管理员：
  - `wait()` = 抬起杆子放车进（**资源-1**）
  - `post()` = 落下杆子放车出（**资源+1**）
- **核心**：管**数量**（剩余车位）

```cpp
// 超简版信号量实现（C++20标准）
#include <semaphore>
std::counting_semaphore<5> parking(5); // 5个车位

void car(int id) {
    parking.acquire();      // 等车位（杆子抬起）
    std::cout << "Car " << id << " parked\n";
    parking.release();      // 开走（杆子落下）
}
```

---

### 🚥 **条件变量 (Condition Variable) - 奶茶店模型**
**口诀**：**“条件不满足，解锁等通知”**  
- 像奶茶店取餐：
  - `wait()` = 没奶茶时**放下号码牌睡觉**
  - `notify()` = 奶茶做好后**叫号唤醒顾客**
- **核心**：管**条件**（奶茶是否做好）

```cpp
// 条件变量基本结构
std::mutex mtx;
std::condition_variable cv;
bool milktea_ready = false; // 关键条件

// 顾客线程
void customer() {
    std::unique_lock lock(mtx);
    cv.wait(lock, []{ return milktea_ready; }); // 没奶茶就睡
    std::cout << "Got milktea!\n";
}

// 店员线程
void staff() {
    {
        std::lock_guard lock(mtx);
        milktea_ready = true; // 奶茶做好
    }
    cv.notify_all(); // 大喊：奶茶好了！
}
```

---

### 🔑 **对比记忆表**
| **特征**       | 信号量                          | 条件变量                     |
|----------------|--------------------------------|------------------------------|
| **核心思想**   | 资源计数器（还剩几个？）         | 条件检查（事情发生了吗？）    |
| **操作**       | `wait()`减资源，`post()`加资源 | `wait()`休眠，`notify()`唤醒 |
| **锁依赖**     | 自带锁机制                      | **必须搭配mutex使用**         |
| **适用场景**   | 连接池/限流                    | 任务协调/事件等待            |
| **生活比喻**   | 停车场进出系统                 | 奶茶店叫号系统               |

---

### 🧠 **记忆技巧**
1. **信号量记数字**：
   - 看到“允许N个线程访问” → 选信号量
   - 代码看到`.acquire()/.release()` → 信号量

2. **条件变量查状态**：
   - 看到“当XX发生时唤醒” → 选条件变量
   - 代码看到`cv.wait(锁, lambda条件)` → 条件变量

3. **必背两句话**：
   - 信号量：**“资源不够就阻塞，释放资源就加一”**
   - 条件变量：**“条件不满足时解锁等待，满足时加锁执行”**

---

### 🌰 1分钟实战场景
```cpp
// 场景：仅允许3个线程同时下载
std::counting_semaphore down_sem(3); 

void download() {
    down_sem.acquire();    // 占位
    // ...下载操作...
    down_sem.release();    // 释放
}

// 场景：线程等待服务器启动
std::condition_variable cv;
bool server_ready = false;

void client() {
    std::unique_lock lk(mtx);
    cv.wait(lk, []{ return server_ready; }); // 等就绪
    // ...连接服务器...
}

void server() {
    // ...启动服务...
    server_ready = true;
    cv.notify_all(); // 通知所有客户端
}
```

### 用条件变量实现信号量

即用条件变量（condition variable）和互斥锁（mutex）来模拟信号量的 P/V 操作。

如果多个资源场景，**各线程对资源的要求条件复杂，容易发生死锁**的话，建议使用条件变量的实现

例如哲学家吃饭问题，每个哲学家都先拿起自己右手边的叉子，就会产生死锁，没有哲学家再能拿起左手边的叉子并吃饭。

- **P 操作（等待/获取资源）**  
  ```cpp
  void P(sem_t *sem) {
      hold(&sem->mutex) {
          while (!COND)
              cond_wait(&sem->cv, &sem->mutex);
          sem->count--;
      }
  }
  ```
  - 先加锁（hold mutex），保证对信号量的操作是原子的。
  - 如果条件不满足（如 count <= 0），就用条件变量等待（cond_wait），并自动释放 mutex，直到被唤醒。
  - 被唤醒后再次加锁，检查条件，满足则 count--，表示获取一个资源。

- **V 操作（释放/归还资源）**  
  ```cpp
  void V(sem_t *sem) {
      hold(&sem->mutex) {
          sem->count++;
          cond_broadcast(&sem->cv);
      }
  }
  ```
  - 加锁，count++，表示释放一个资源。
  - 用条件变量唤醒所有等待线程（cond_broadcast）。

### 用信号量实现条件变量

用信号量实现条件变量的 wait/broadcast 操作：

**1. `wait` 函数**

```c
void wait(struct condvar *cv, mutex_t *mutex) {
    mutex_lock(&cv->lock);
    cv->nwait++;
    mutex_unlock(&cv->lock);

    mutex_unlock(mutex);
    P(&cv->sleep);
    mutex_lock(mutex);
}
```

- `cv->nwait++`：记录当前有多少线程在等待条件变量（进入等待队列）。
- `mutex_unlock(mutex)`：释放主互斥锁，允许其他线程进入临界区修改条件。
- `P(&cv->sleep)`：在信号量 sleep 上等待（阻塞），直到被唤醒。
- `mutex_lock(mutex)`：被唤醒后，重新获得主互斥锁，继续执行。

**2. `broadcast` 函数**

```c
void broadcast(struct condvar *cv) {
    mutex_lock(&cv->lock);
    for (int i = 0; i < cv->nwait; i++) {
        V(&cv->sleep);
    }
    cv->nwait = 0;
    mutex_unlock(&cv->lock);
}
```

- `mutex_lock(&cv->lock)`：保护 nwait 变量，防止并发修改。
- `for (int i = 0; i < cv->nwait; i++) V(&cv->sleep);`：唤醒所有等待的线程（每个线程对应一次 V 操作）。
- `cv->nwait = 0;`：重置等待计数。
- `mutex_unlock(&cv->lock)`：释放锁。

#### 信号量实现条件变量出现的问题

- 线程T2调用wait()，此时已经把自己标记为等待（nwait++），但还没真正阻塞（还没执行P）。
- 这时，因为其它线程的操作，另一个线程T1调用broadcast()，唤醒了所有等待线程（V操作），并继续执行。
- 此时nwait已经归零，T2就会睡死

从理论上来说，我们希望`nwait++`, `mutex_unlock`和P操作是原子性的，并且`mutex_unlock`和P操作不可以互换位置（会在睡时一直持有锁，死锁了），但是实际上我们需要操作系统用类似条件变量的机制帮我们实现wait和release的原子性，这样就套娃了