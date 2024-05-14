---
layout:     post
title:      优化app启动
subtitle:   从启动框架到启动时长的优化治理的一些经验思考。
date:       2024-03-30
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Summary
- APM
- Launch
- Code
---

# 优化app启动

从启动框架到启动时长的优化治理的一些经验思考。

## app启动框架
经典的工程，app 启动过程分为几个阶段：

```
1、加载plist
2、创建沙盒路径
3、plist检查权限
4、加载mach O，运行dyld，验证加载依赖库/方法（创建mmap），rebase/bind 内存偏移和符号绑定
5、初始化运行时（class category selector），+load，attribute((constructor))，C++静态变量
6、main 执行，创建Application，Delegate，Runloop，显示RootWindow，RootVC（或者加载 main storyboard）
7、didFinishLaunchingWithOptions，做一些sdk初始化，展示app首页
```
**除此之外，还有**
```
8、隐私页面
9、SDK加载，权限授权
10、首页加载
11、首页已经空闲
```

### 启动框架封装

以上启动过程较多，而且大多数是我们未必关心的，那一部分的时间对于启动耗时的影响也是很有限。进行分层的启动框架，有助于统计启动耗时，优化启动体验。可以有效控制合规要求。同时使用分层而不是依赖，也能够减少app启动的逻辑处理。


#### 定义
```
1、pre_main 先于main执行的，一些方法交换，模块注册，Bridge注册
2、main_basic 底层基础库初始化，crash，日志，storage（不可申请权限）
3、main_biz 其他sdk基础sdk初始化，定位，网络，路由，配置，监控（不可申请权限）
4、privacy 展示隐私协议，同意后进行一些非UI的权限使用，例如 udid，ip等，可以准备首页数据，但不能进行权限操作
5、home 首页加载，初始化首页数据，初始化各个业务sdk
6、home_idle 首页已经渲染完成，进行版本更新检查，首页弹窗，懒加载的部分sdk初始化，debug面板
```
其中每个阶段设置优先级，按照从高到低依次运行。一层运行完成之后可运行另外一层。

#### 实现方式

启动框架需要收归所有启动task，然后统一进行分发执行。

##### pre main
有两种方法可以达到代码启动时机的直接调用的效果，`load`和`__attribute__((constructor))`。

单就效率来说，应该是后者高，因为不需要处理依赖关系。


```c
/// 在.m文件，使用 `__attribute__((constructor))` 进行启动时机的方法分发。
__attribute__((constructor))
static void init(void) {
    
}
```

有了理论干货，如何设计注册分发框架呢：

**有几种方式：**
```
1、使用运行时，拉取 class list，判断实现了启动协议，进行初始化操作。
2、使用注册表对象，各个模块在`load`和`__attribute__((constructor))`的时机注册。使用时检查注册表。
3、使用 `__attribute((used, section("__DATA, XXModules"))) = ""#$module_class""` 在编译时间锁定类目，写入 `SEG_DATA`，加载lib的运行时进行一次性读取注册表。
```

第一种，适用于懒加载的注册解藕框架，但是在启动过程中，未必是时间最优的；另外还需要处理多线程问题。
第二种，最方便，使用load管理，但是load过多可能影响启动性能。
第三种，大概率是时间代价最小，因为不需要注册，直接从mach o读取相关Data完成map创建。

```c

#pragma mark - Module和Service注册处理
NSArray<NSString *>* LoadAnnotationData(char *sectionName,const struct mach_header *mhp);
static void dyld_callback(const struct mach_header *mhp, intptr_t vmaddr_slide) {
    //解析SEG_DATA中标注的module
    NSArray *modules = LoadAnnotationData("XXModules", mhp);
    for (NSString *module in modules) {
        if (!module || module.length <= 0) {
            continue;
        }
        [[XXBModule shared] registerModule:module];
    }
}

__attribute__((constructor))
void init(void) {
    _dyld_register_func_for_add_image(dyld_callback);
}

//解析SEG_DATA中标注的内容
NSArray<NSString *>* LoadAnnotationData(char *sectionName, const struct mach_header *mhp) {
    NSMutableArray *annotationList = [NSMutableArray array];
    unsigned long size = 0;
#ifndef __LP64__
    uintptr_t *memory = (uintptr_t*)getsectiondata(mhp, SEG_DATA, sectionName, &size);
#else
    const struct mach_header_64 *mhp64 = (const struct mach_header_64 *)mhp;
    uintptr_t *memory = (uintptr_t*)getsectiondata(mhp64, SEG_DATA, sectionName, &size);
#endif
    
    unsigned long count = size/sizeof(void*);
    for(int idx = 0; idx < count; ++idx) {
        char *string = (char*)memory[idx];
        NSString *str = [NSString stringWithUTF8String:string];
        if(str == nil)
            continue;
        [annotationList addObject:str];
    }
    return annotationList;
}
```
> 站在巨人的肩膀上。

##### 其他阶段

分发阶段类似以上。

- main阶段，需要接管 ApplicationMainWithHandler，在实现种采用上述方式分发即可。bisic 完成后分发 biz。

- privacy，发出通知，隐私模块开始加载页面。隐私也加载完成，通知启动框架，启动框架分发 idle。用户同意后进行入 home 阶段。

- home 加载完成，主线程空闲一定时间调用 idle。

### 启动耗时

在响应时机进行启动耗时的监听。

#### 温启动

温启动的情况，app会执行一些 pre main，但是不会执行到 main.m。需要进行异常处理。

```
+ (long long)processStartTime
{
    struct kinfo_proc kProcInfo;
    if ([self processInfoForPID:[[NSProcessInfo processInfo] processIdentifier] procInfo:&kProcInfo]) {
        return kProcInfo.kp_proc.p_un.__p_starttime.tv_sec * 1000 + kProcInfo.kp_proc.p_un.__p_starttime.tv_usec / 1000;
    } else {
        NSAssert(NO, @"无法取得进程的信息");
        return 0;
    }
}

+ (BOOL)processInfoForPID:(int)pid procInfo:(struct kinfo_proc*)procInfo
{
    int cmd[4] = {CTL_KERN, KERN_PROC, KERN_PROC_PID, pid};
    size_t size = sizeof(*procInfo);
    return sysctl(cmd, sizeof(cmd)/sizeof(*cmd), procInfo, &size, NULL, 0) == 0;
}

```

## 启动耗时优化

耗时大体分为，基础启动耗时，首页数据加载耗时，这里主要讨论基础启动耗时，即火山、bugly、apple统计到的启动耗时。

> 首页网络加载耗时，可以通过缓存，减少数据体，网络请求前置等方式处理。

### 常见耗时问题

#### task 耗时处理

通过以上分发，可以搜集到所以注册进来的 task 耗时，进儿可以通过统计数据对头部问题进行修复。

经验来说主要耗时还是体现在代码使用不合理上：

- 启动时机过早，例如一个IM模块，完全可以后置到首页完成
- 多线程处理，一些耗时操作，完成可以在更早的时机子线程进行初始化，然后在后续使用时候，无需或者稍微等待即可，例如sqllite。**要控制好时间，否则可能导致优先级反转**，但是代价往往是值得的，因为启动初期，线程数使用数量很少，多线程用到就是赚到。
- 一些系统耗时api在主线程的不合适使用，avplayer，idfa
- 单点耗时函数处理，**可通过 Xcode 的 Launch 的 trace report查看**
- 业务下线，未使用的代码下线

以上 task 耗时处理完成，能将app耗时p50从1200ms下降到600ms。

#### task之外的pre main耗时优化

- load方法耗时监控 

```
load_images -> load_images_nolock -> prepare_load_methods -> schedule_class_load -> add_class_to_loadable_list -> call_load_methods
```
一顿 dyld 的函数调用，ObjC 对于加载的管理主要使用了两个列表，分别是 loadable_classes 和 loadable_categories。然后进行 call_load_methods 调用。对于 C 函数可使用 fishhook 进行 hook，但是通过系统dyld，拿到指针似乎没呢么容易。

而使用常规的方式又
如果实现监控，需要一点点技巧：

使用常规方式的+load，时机很有可能过晚，dyld 已经记录了 list，因为 Load 的调用方式为指针调用，所以hook成功，但是无法收集到系统调用的那次的时长。结合上面的知识，如果能在更早的时机完成 Load 替换，让系统记录到替换后到指针，那么就可以完成统计。

因为 load Image 发生在更早的时机，使用一个动态库，要尽量靠前，即可在启动时机动态库加载时候进行 hook，完成采集。

```

+ (void)load {
        
    _loadInfoArray = [[NSMutableArray alloc] init];
    _total_cost_time = 0.0;
        
    int imageCount = (int)_dyld_image_count();
    for(int iImg = 0; iImg < imageCount; iImg++) {
        
        const char* path = _dyld_get_image_name((unsigned)iImg);
        NSString *imagePath = [NSString stringWithUTF8String:path];
        
        NSBundle* mainBundle = [NSBundle mainBundle];
        NSString* bundlePath = [mainBundle bundlePath];
        
        if (([imagePath containsString:bundlePath] && ![imagePath containsString:@".dylib"]) || [imagePath containsString:@"Build/Products/"]) {
            unsigned int count;
            const char **classes;
            classes = objc_copyClassNamesForImage(path, &count);
            
            for (int i = 0; i < count; i++) {
                NSString *className = [NSString stringWithCString:classes[i] encoding:NSUTF8StringEncoding];
                if (![self replaceLoadMethodOfClass: className]) {
                    continue;
                }
            }
        }
    }
    NSArray<Class> *perservedClasses = [self presevedSystemClasses];
    if (perservedClasses && perservedClasses.count != 0) {
        for(Class cls : perservedClasses) {
            NSString *className = NSStringFromClass(cls);
            if (![self replaceLoadMethodOfClass: className]) {
                continue;
            }
        }
    }
}

+ (BOOL)replaceLoadMethodOfClass:(NSString *)className {
    SEL originalSelector = @selector(load);
    if (![className isEqualToString:@""] && className) {
        Class cls = object_getClass(NSClassFromString(className));
        
        if(cls == [self class]){
            return NO;
        }
        unsigned int methodCount = 0;
        Method * methods = class_copyMethodList(cls, &methodCount);
        NSUInteger currentLoadIndex = 0;
        for(unsigned int methodIndex = 0; methodIndex < methodCount; ++methodIndex){
            Method method = methods[methodIndex];
            std::string methodName(sel_getName(method_getName(method)));
         
            if(methodName == "load"){
                SEL swizzledSelector = NSSelectorFromString([NSString stringWithFormat:@"XXAPM_Load%@",@(currentLoadIndex)]);
                Method originalMethod = method;
                Method swizzledMethod = class_getClassMethod([self class], swizzledSelector);
                BOOL addSuccess = class_addMethod(cls, originalSelector, method_getImplementation(swizzledMethod), method_getTypeEncoding(swizzledMethod));
                // 添加成功，则说明不存在load。但动态添加的load，不会被调用。与load的调用方式有关。
                if(!addSuccess){
                    // 已经存在，则添加新的selector
                    BOOL didAddSuccess = class_addMethod(cls, swizzledSelector, method_getImplementation(swizzledMethod), method_getTypeEncoding(swizzledMethod));
                    if(didAddSuccess){
                    // 然后交换
                        swizzledMethod = class_getClassMethod(cls, swizzledSelector);
                        method_exchangeImplementations(originalMethod, swizzledMethod);
                    }
                }
         
                ++currentLoadIndex;
            }
        }
    }
    return YES;
}

+ (NSArray<Class> *)presevedSystemClasses {
    return @[[UIApplication class], [UIViewController class], [UINavigationController class]];
}

+ (void)XXAPM_Load0 {
    
    CFAbsoluteTime start =CFAbsoluteTimeGetCurrent();
    
    [self XXAPM_Load0];
    
    CFAbsoluteTime end =CFAbsoluteTimeGetCurrent();
    // 时间精度 ms
    double costTime = end-start;
    _total_cost_time += costTime;
    NSString *className = NSStringFromClass([self class]);
    NSDictionary *infoDic = @{@"cost_time":TIMESTAMP_NUMBER(costTime),
                              @"name":className
                              };
    
    [_loadInfoArray addObject:infoDic];
    NSLog(@"apm load monitor : %@", infoDic);
    NSLog(@"apm load monitor total cost time: %@", @(_total_cost_time));
}

+ (void)XXAPM_Load1 {
    
    CFAbsoluteTime start =CFAbsoluteTimeGetCurrent();
    
    [self XXAPM_Load1];
    
    CFAbsoluteTime end =CFAbsoluteTimeGetCurrent();
    // 时间精度 ms
    double costTime = end-start;
    _total_cost_time += costTime;
    NSString *className = NSStringFromClass([self class]);
    NSDictionary *infoDic = @{@"cost_time":TIMESTAMP_NUMBER(costTime),
                              @"name":className
                              };
    
    [_loadInfoArray addObject:infoDic];
    NSLog(@"apm load monitor : %@", infoDic);
    NSLog(@"apm load monitor total cost time: %@", @(_total_cost_time));
}

+ (void)XXAPM_Load2 {
    
    CFAbsoluteTime start =CFAbsoluteTimeGetCurrent();
    
    [self XXAPM_Load2];
    
    CFAbsoluteTime end =CFAbsoluteTimeGetCurrent();
    // 时间精度 ms
    double costTime = end-start;
    _total_cost_time += costTime;
    NSString *className = NSStringFromClass([self class]);
    NSDictionary *infoDic = @{@"cost_time":TIMESTAMP_NUMBER(costTime),
                              @"name":className
                              };
    
    [_loadInfoArray addObject:infoDic];
    NSLog(@"apm load monitor : %@", infoDic);
    NSLog(@"apm load monitor total cost time: %@", @(_total_cost_time));
}

+ (void)XXAPM_Load3 {
    
    CFAbsoluteTime start =CFAbsoluteTimeGetCurrent();
    
    [self XXAPM_Load3];
    
    CFAbsoluteTime end =CFAbsoluteTimeGetCurrent();
    // 时间精度 ms
    double costTime = end-start;
    _total_cost_time += costTime;
    NSString *className = NSStringFromClass([self class]);
    NSDictionary *infoDic = @{@"cost_time":TIMESTAMP_NUMBER(costTime),
                              @"name":className
                              };
    
    [_loadInfoArray addObject:infoDic];
    NSLog(@"apm load monitor : %@", infoDic);
    NSLog(@"apm load monitor total cost time: %@", @(_total_cost_time));
}


```

```
  s.pod_target_xcconfig = {"CLANG_CXX_LANGUAGE_STANDARD" => "gnu++17",
    "CLANG_CXX_LIBRARY" => "libc++", "MACH_O_TYPE" => "mh_dylib"
  # s.static_framework = false

```

> 如果动态库启动出现问题，检查是否copy成功。

- 二进制重排减少启动内存缺页中断

二进制重排，可以对函数的内存地址进行排序，达到减少内存中断的目的，提高启动速度。

有比较多的资料：
[二进制重排 Page Fault](https://www.jianshu.com/p/c06601772238)
[启动优化（二）——二进制重排](https://cloud.tencent.com/developer/article/1814588)

```
// h

#import <Foundation/Foundation.h>

extern void AppOrderFiles(void(^completion)(NSString *orderFilePath));

// m
#import <dlfcn.h>
#import <libkern/OSAtomicQueue.h>
#import <pthread.h>

static OSQueueHead queue = OS_ATOMIC_QUEUE_INIT;

static BOOL collectFinished = NO;
static BOOL guardInited = NO;

typedef struct {
    void *pc;
    void *next;
} PCNode;

// The guards are [start, stop).
// This function will be called at least once per DSO and may be called
// more than once with the same values of start/stop.
void __sanitizer_cov_trace_pc_guard_init(uint32_t *start,
                                         uint32_t *stop) {
    static uint32_t N;  // Counter for the guards.
    if (start == stop || *start) return;  // Initialize only once.
    printf("INIT: %p %p\n", start, stop);
    for (uint32_t *x = start; x < stop; x++)
        *x = ++N;  // Guards should start from 1.
    guardInited = YES;
}

// This callback is inserted by the compiler on every edge in the
// control flow (some optimizations apply).
// Typically, the compiler will emit the code like this:
//    if(*guard)
//      __sanitizer_cov_trace_pc_guard(guard);
// But for large functions it will emit a simple call:
//    __sanitizer_cov_trace_pc_guard(guard);
void __sanitizer_cov_trace_pc_guard(uint32_t *guard) {
    if (collectFinished) {
        return;
    }
    if (guardInited && !*guard) {
        return;
    }
    // If you set *guard to 0 this code will not be called again for this edge.
    // Now you can get the PC and do whatever you want:
    //   store it somewhere or symbolize it and print right away.
    // The values of `*guard` are as you set them in
    // __sanitizer_cov_trace_pc_guard_init and so you can make them consecutive
    // and use them to dereference an array or a bit vector.
    *guard = 0;
    void *PC = __builtin_return_address(0);
    PCNode *node = malloc(sizeof(PCNode));
    *node = (PCNode){PC, NULL};
    OSAtomicEnqueue(&queue, node, offsetof(PCNode, next));
}

extern void AppOrderFiles(void(^completion)(NSString *orderFilePath)) {
    collectFinished = YES;
    NSLog(@"AppOrder: AppOrderFiles start");
    __sync_synchronize();
    NSString *functionExclude = [NSString stringWithFormat:@"_%s", __FUNCTION__];
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.01 * NSEC_PER_SEC)), dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSMutableArray <NSString *> *functions = [NSMutableArray array];
        while (YES) {
            PCNode *node = OSAtomicDequeue(&queue, offsetof(PCNode, next));
            if (node == NULL) {
                break;
            }
            Dl_info info = {0};
            dladdr(node->pc, &info);
            if (info.dli_sname) {
                NSString *name = @(info.dli_sname);
                BOOL isObjc = [name hasPrefix:@"+["] || [name hasPrefix:@"-["];
                NSString *symbolName = isObjc ? name : [@"_" stringByAppendingString:name];
                [functions addObject:symbolName];
            }
        }
        if (functions.count == 0) {
            if (completion) {
                completion(nil);
            }
            return;
        }
        NSMutableArray<NSString *> *calls = [NSMutableArray arrayWithCapacity:functions.count];
        NSEnumerator *enumerator = [functions reverseObjectEnumerator];
        NSString *obj;
        while (obj = [enumerator nextObject]) {
            if (![calls containsObject:obj]) {
                [calls addObject:obj];
            }
        }
        [calls removeObject:functionExclude];
        NSString *result = [calls componentsJoinedByString:@"\n"];
        NSLog(@"AppOrder: AppOrderFiles finish, result:\n%@", result);
        NSString *filePath = [NSHomeDirectory() stringByAppendingPathComponent:@"app.order"];
        NSData *fileContents = [result dataUsingEncoding:NSUTF8StringEncoding];
        BOOL success = [[NSFileManager defaultManager] createFileAtPath:filePath
                                                               contents:fileContents
                                                             attributes:nil];
        if (completion) {
            completion(success ? filePath : nil);
        }
    });
}

```

以上可以在 Document 生成一份order文件，有助于启动耗时优化。

- 注意要在 release 环境跑
- 自动化情况下，要使用 build 产物 .app，不能使用 export ipa。


## 首页优化

一笔带过：
- 缓存
- 前置网络请求
- 核心接口优化，服务端返回少数据，用户无感加载list
