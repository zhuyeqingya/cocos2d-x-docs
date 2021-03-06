## 给 OPPO 手机做的优化

 > 该优化代码只在 __Cocos2d-x 3.17.2__ 及以上的版本起效，并且目前只在 __OPPO Reno__ 有效果。

 ### 具体优化方案

引擎内部在两个地方添加了优化代码：

- 场景加载
- 编译引擎内置的 shader 脚本

场景加载指的是从 `Scene` 创建到 `Scene::onEnter()` 被调用这段时间，所以加载资源的代码要放在 `Scene::onEnter()` 前，且需要在 `Scene` 创建后马上进行资源加载。

### 手动调用优化代码

程序比引擎更清楚什么时机需要更多的 CPU、GPU 资源，因此程序代码可以在需要更多资源的地方手动掉优化代码。可以在 C++ 或 Java 调用优化代码：

C++ 的调用方式

```c++

// 通知系统某一事件发生，如场景加载开始，场景加载结束。3.17.2 之后才增加的绑定。
#if CC_TARGET_PLATFORM == CC_PLATFORM_ANDROID
    DataManager::setOptimise(const string&, const string&);
#endif

// 场景加载开始，需要更多的资源
// 该接口适用于突发重负载的场景，如加载地图,GC、突发大量访问磁盘文件与数据库等

#if CC_TARGET_PLATFORM == CC_PLATFORM_ANDROID
    DataManager::onSceneLoaderBegin();
#endif

// 场景加载结束
#if CC_TARGET_PLATFORM == CC_PLATFORM_ANDROID
    DataManager::onSceneLoaderEnd();
#endif

// 开始编译 shader 脚本
#if CC_TARGET_PLATFORM == CC_PLATFORM_ANDROID
    DataManager::onShaderLoaderBegin();
#endif

// 结束编译 shader 脚本
#if CC_TARGET_PLATFORM == CC_PLATFORM_ANDROID
    DataManager::onShaderLoaderEnd();
#endif
```

Java 的调用方式

```Java
Cocos2dxDataManager::setOptimise(String thing, float value);
```

| thing | 1 | 0 |
| ----  | -- | -- |
| load_scene| 开始加载场景 | 结束加载场景 |
| shader_compile | 开始编译 shader 脚本 | 结束编译 shader 脚本 |

V3.17.2 版本后，`value` 的类型由 `float` 变成了 `string`，这样就更灵活了。

`thing` 与 `value`目前默认支持上述两种方式，如果需要更加定制化的控制，需遵循手机厂商的标准才能被正确识别并生效。如在OPPO手机上，需要首先调用 `setOptimise（"permission"， permissionIInfo）`，其中 `permissionIInfo` 需登录OPPO开放平台申请授权码，然后再传输定制的参数才会生效。具体可参考[该文档](https://open.oppomobile.com/wiki/doc#id=10355)。





