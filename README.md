# objc-runtime-compile
objc4-818.2可以编译的源代码

官方Runtime源码编译流程

[Runtime源码地址](https://opensource.apple.com/tarballs/objc4/)

1.打开主工程objc.xcodeproj

target->objc->Build Settings

1.所有target的项目都要更改Base SDK 为masOS

2.更改Order File目录下的libojbc.order路径为项目路径

3.更改Header Search Paths 增加$(SRCROOT)/KCCommon 加入这个内核源码

4.更改Preprocessor Macros 增加LIBC_NO_LIBCRASHREPORTERCLIENT

5.更改Other Linker Flags 参数-lCrashReporterClient -loah(Release保留)

6.修改Build Phases-》Run Script(markgc)中的macosx.internal改成macosx

7.Enable Hardened Runtime改为NO

8.注释掉如下代码


```
#include <os/feature_private.h>

#include <os/reason_private.h>
#include <os/variant_private.h>

 if (!dyld_program_sdk_at_least(dyld_fall_2020_os_versions))
        DisableAutoreleaseCoalescingLRU = true;
        
#include <Cambria/Traps.h>
#include <Cambria/Cambria.h>

#include "objc-bp-assist.h"

include <os/linker_set.h>

LINKER_SET_FOREACH(_dupi, const objc_duplicate_class **, "__objc_dupclass") {
    const objc_duplicate_class *dupi = *_dupi;

    if (strcmp(dupi->name, name) == 0) {
        return;
    }
}

if (!dyld_program_sdk_at_least(dyld_platform_version_macOS_10_13)) {
    DisableInitializeForkSafety = true;
    if (PrintInitializing) {
        _objc_inform("INITIALIZE: disabling +initialize fork "
                     "safety enforcement because the app is "
                     "too old.)");
    }
}

if (!os_feature_enabled_simple(objc4, preoptimizedCaches, true)) {
    DisablePreoptCaches = true;
}

if (!dyld_program_sdk_at_least(dyld_platform_version_macOS_10_11)) {
    DisableNonpointerIsa = true;
    if (PrintRawIsa) {
        _objc_inform("RAW ISA: disabling non-pointer isa because "
                     "the app is too old.");
    }
}

//        if (oah_is_current_process_translated()) {
//            kern_return_t ret = objc_thread_get_rip(threads[count], (uint64_t*)&pc);
//            if (ret != KERN_SUCCESS) {
//                pc = PC_SENTINEL;
//            }
//        } else {
//            pc = _get_pc_for_thread (threads[count]);
//        }

 && dyld_program_sdk_at_least(dyld_fall_2018_os_versions)
 
  || sdkIsAtLeast(10_12, 10_0, 10_0, 3_0, 2_0)
```


target->objc->Build Settings

1.更改Base SDK 为masOS

然后就能build成功了

创建调试工程

File->Target->macOS->Command Line Tool

修改TARGET->调试工程名字->Build Phases->Dependencies,增加objc这个target

可以搞起了