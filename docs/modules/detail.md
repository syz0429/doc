# 客户端的详细实现模块

## 模拟器 [Simulator](https://github.com/OpenHUTB/hutb/blob/hutb/LibCarla/source/carla/client/detail/Simulator.cpp)

作为客户端核心调度器：管理 Episode 生命周期、地图加载、帧同步、Actor 与传感器的创建与订阅，并桥接底层 _client 与高层 API。


## 回合代理 [EpisodeProxy](https://github.com/OpenHUTB/hutb/blob/hutb/LibCarla/source/carla/client/detail/EpisodeProxy.cpp)

访问当前仿真回合的上下文，防止在换图/重载后继续操作旧回合的对象导致崩溃或逻辑错误（Episode 变更时优雅地拒绝操作或提示对象已销毁）。

**工作机制**：每次访问通过 TryLock()/Lock()获取模拟器：比较保存的 EpisodeId 与当前的是否一致。不一致或对象已销毁时返回空指针（TryLock）或抛异常（Lock）。


## 参与者工厂 [ActoryFactory](https://github.com/OpenHUTB/hutb/blob/hutb/LibCarla/source/carla/client/detail/ActorFactory.cpp)

**类型分派工厂**: 根据 rpc::Actor 描述符的 id 字段，实例化对应的客户端 Actor 子类（Vehicle、Walker、Sensor、TrafficLight 等）。

**统一构造入口**: 提供 [MakeActor](https://github.com/OpenHUTB/hutb/blob/04b87b07133148b660d0bbb773f58e754bb20cd7/LibCarla/source/carla/client/detail/ActorFactory.cpp#L73) 静态方法，屏蔽内部类型判断细节，上层只需传入描述符即可得到正确的强类型对象。

**垃圾回收策略**：启用（Enabled）时使用 GarbageCollector deleter，在 shared_ptr 析构时自动调用 Actor::Destroy() 销毁服务器端对象，捕获超时与异常。Disabled 时仅用默认 deleter 销毁本地对象，不触碰服务器端；需手动管理。

## 参与者变体 [ActorVariant](https://github.com/OpenHUTB/hutb/blob/hutb/LibCarla/source/carla/client/detail/ActorVariant.cpp)

**统一包装**: 用一个 variant 保存“轻量的 rpc::Actor 描述”或“已构造的客户端 Actor 对象”，便于上层统一管理与延迟处理。


## 缓存的参与者列表 [CachedActorList](https://github.com/OpenHUTB/hutb/blob/hutb/LibCarla/source/carla/client/detail/CachedActorList.h)

**核心目的**：避免重复向服务器请求 Actor 描述信息，通过本地缓存 rpc::Actor 提升性能。

**设计原理**

* **本地描述符缓存**: 维护 [std::unordered_map<ActorId, rpc::Actor>](https://github.com/OpenHUTB/hutb/blob/04b87b07133148b660d0bbb773f58e754bb20cd7/LibCarla/source/carla/client/detail/CachedActorList.h#L59) 存储已知 Actor 的元信息（类型、属性、变换等）
* **线程安全**: 所有操作用互斥锁（ [std::mutex](https://github.com/OpenHUTB/hutb/blob/04b87b07133148b660d0bbb773f58e754bb20cd7/LibCarla/source/carla/client/detail/CachedActorList.h#L81) ）保护（该句代码后面即为被保护的共享数据，析构时自动解锁，即使发生异常也能正确解锁），支持多线程并发访问
* **增量更新**: [只请求缺失的 Actor](https://github.com/OpenHUTB/hutb/blob/04b87b07133148b660d0bbb773f58e754bb20cd7/LibCarla/source/carla/client/detail/CachedActorList.h#L86) ，已缓存的直接返回


## 回调列表 CallbackList

**核心目的**：线程安全的回调函数管理器，支持 [注册回调](https://github.com/OpenHUTB/hutb/blob/04b87b07133148b660d0bbb773f58e754bb20cd7/LibCarla/source/carla/client/detail/CallbackList.h#L32) 、[触发所有回调](https://github.com/OpenHUTB/hutb/blob/04b87b07133148b660d0bbb773f58e754bb20cd7/LibCarla/source/carla/client/detail/CallbackList.h#L25) 、[按ID移除回调](https://github.com/OpenHUTB/hutb/blob/04b87b07133148b660d0bbb773f58e754bb20cd7/LibCarla/source/carla/client/detail/CallbackList.h#L39) ，常用于事件通知、传感器数据分发等场景。



