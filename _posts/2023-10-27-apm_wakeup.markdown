---
layout:     post
title:      卡顿、内存、wakeup监控 - 3
subtitle:   app异常退出，wakeup监控
date:       2023-10-27
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Summary
- APM
- Crash
---

# wakeup监控

这个是运行在iOS系统，短时间大量的线程唤醒，导致系统看门狗机制强杀，app退出。此种情况是app进程，内存等沙盒直接清理，app没有时机处理。


## 分析卡顿：

因为没有明确的文档说明来描述何种情况会发生 weakup 错误。但是可以从开发者论坛大概看到一些信息：[超过唤醒限制](https://developer.apple.com/forums/thread/124180)

一般类型分为：
- wakeups 均值异常 ：
         时机： 300秒内 wakeups 增量均值超过150次（仿苹果系统底层wakeups监控实现，参考资料： https://developer.apple.com/forums/thread/124180 ）
         上报数据：当前监控周期wakeups增长计数总数、每秒增长平均值、监控时长、严重异常分段信息等
- wakeups 突增异常 ：
         时机：当前秒 wakeups 增量超过450次(可配置) && 相较上一秒涨幅较大 (600以下涨幅达到120%，600以上涨幅达到110%)
         上报数据：
           如果为单次突增，则上报当前秒 wakeups 增量、临近行为事件类型、临近行为事件特征；
           如果为连续上涨突增，则上报当前连续突增阶段的wakeups 增量s、临近行为事件类型、临近行为事件特征；
- wakeups 连续高位异常 ：
         时机：wakeups增量在阈值600(可配置) 以上
         上报数据：高位阶段 wakeups 增量每秒平均值

[Wakeups limit exceeded | Apple Developer Forums](https://developer.apple.com/forums/thread/124180)
[iOS/OSX异常 WAKEUPS（闲置唤醒）](https://zhuanlan.zhihu.com/p/536021896)
[快手客户端稳定性体系建设_wakeups over the last-CSDN博客](https://blog.csdn.net/idaretobe/article/details/113064417)
[wakeup in XNU](https://djs66256.github.io/2021/04/03/2021-04-03-wakeup-in-XNU/)


## 数据采集

### 如何采集

几个核心
1、如何获取wakeup次数

```
#include <mach/mach.h>
#include <mach/task.h>

/**
 方法耗时区间 0.000006s - 0.000023s
 */
BOOL GetSystemInterruptWakeups(NSInteger *interrupt_wakeup) {
    struct task_power_info info = {0};
    mach_msg_type_number_t count = TASK_POWER_INFO_COUNT;
    kern_return_t ret = task_info(current_task(), TASK_POWER_INFO, (task_info_t)&info, &count);
    if (ret == KERN_SUCCESS) {
        if (interrupt_wakeup) {
            *interrupt_wakeup = info.task_interrupt_wakeups;
        }
        return true;
    }
    else {
        if (interrupt_wakeup) {
            *interrupt_wakeup = 0;
        }
        return false;
    }
}


+ (NSInteger)getCurrentSystemWakeups {
    NSInteger interruptWakeups;
    GetSystemInterruptWakeups(&interruptWakeups);
    return interruptWakeups;
}

```
2、如何监控 wakeup 超限制。

一个是上面的各种异常判断手段，一个是堆栈获取。

> hook ulock_wake获取backtrace的流程图，通过记录ulock_wake调用时的wakeup数和时间戳，在超出阈值时抓取backtrace并记录到文件中。ulock_wake是内核代码，不能通过常规的Method Swizzling或者fishhook来hook，需要通过Tweak在越狱机上hook。因此，wakeup相关堆栈目前只支持在越狱手机上进行抓取。

堆栈获取通过以上方法无法在线上提供辅助决策。可以接入卡顿监控的堆栈捕获模块，统一进行堆栈捕获，这样就能在判定完成时候，进行一些堆栈补充，同时附加最近的一些信息，计数一个可信度。



## 其他

wakeup 补齐了app退出的一环。前文链接中，快手提供了一个退出率的概念，感觉是一个好思路。将crash，系统强杀，watch dog，oom，用户强杀，app exit，系统升级，app升级，重启等。进行一个统计数据，提供参考意义。通过统计，发现用户在某页面强杀频繁，可以告警以先于用户客诉发现问题，或者发现一些非强阻断的页面问题。