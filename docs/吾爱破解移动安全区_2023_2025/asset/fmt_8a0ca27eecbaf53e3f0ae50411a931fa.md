
```
extern "C" const void* artFindNativeMethodRunnable(Thread* self)
    REQUIRES_SHARED(Locks::mutator_lock_) {
  uint32_t dex_pc;
  ArtMethod* method = self->GetCurrentMethod(&dex_pc);
  ClassLinker* class_linker = Runtime::Current()->GetClassLinker();
  if (!method->IsNative()) {
    .....
  }
  //判断是不是 IsJniDlsymLookupStub
  // 不是的话代表寻找到 jni 的 native 方法指针
  const void* native_code = class_linker->GetRegisteredNative(self, method);
  if (native_code != nullptr) {
    return native_code;
  }
  ...
  //没有的话就根据查找 通过 class——linker 注册 native
  native_code = vm->FindCodeForNativeMethod(method, &error_msg, /*can_suspend=*/true);
  return class_linker->RegisterNative(self, method, native_code);
}
```

## 4> `init_array`

当 `so` 通过 `linker` 加载的时候，会调用初始化段的方法。

如 `.init` 和 `.init_array`

在 bionic 的 `linker` 中可以看到这个地方

```
//linker.cpp

void* do_dlopen(const char* name, int flags,
                const android_dlextinfo* extinfo,
                const void* caller_addr) {
  ....
  //查找 library
  soinfo* si = find_library(ns, translated_name, flags, extinfo, caller);
  loading_trace.End();
  if (si != nullptr) {
    void* handle = si->to_handle();
    //调用 .init_array
    si->call_constructors();
    failure_guard.Disable();
    LD_LOG(kLogDlopen,
          "... dlopen successful: realpath=\"%s\", soname=\"%s\", handle=%p",
          si->get_realpath(), si->get_soname(), handle);
    return handle;
  }
  return nullptr;
}

void soinfo::call_constructors() {
  ....
  get_children().for_each([] (soinfo* si) {
    si->call_constructors();
  });
  if (!is_linker()) {
    bionic_trace_begin((std::string("calling constructors: ") + get_realpath()).c_str());
  }
  //调用 INIT 和 INIT_ARRAY
  // DT_INIT should be called before DT_INIT_ARRAY if both are present.
  call_function("DT_INIT", init_func_, get_realpath());
  call_array("DT_INIT_ARRAY", init_array_, init_array_count_, false, get_realpath());
  if (!is_linker()) {
    bionic_trace_end();
  }
}
```

所以我们监控 `soinfo::call_constructors` 跟踪其堆栈即可。

## 4> `pthread_create`

在 bionic 的 `libc` 中

`pthread_create` 用与创建线程

```
#include "pthread.h"
int pthread_create(pthread_t* __pthread_ptr, pthread_attr_t const* __attr, void* (*__start_routine)(void*), void*);
```

其中 `void* (*__start_routine)(void*)` 是线程调用的方法。

也就是说我们可以通过拦截 `pthread_create` 获取到调用方法的指针。完成方法的跟踪

不过也不能全部都拦截。对于 android 线程来说，跟踪的代码太多。所以我们得屏蔽 `art` 中的方法的

下面是 android 创建线程的过程

```
new Thread(run).start()

class Thread {
  public synchronized void start() {
    ...
    nativeCreate(this, stackSize, daemon);
    ...
  }
}

//runtime/native/java_lang_Thread.cc
static void Thread_nativeCreate(JNIEnv* env, jclass, jobject java_thread, jlong stack_size,
                                jboolean daemon) {
  // There are sections in the zygote that forbid thread creation.
  Runtime* runtime = Runtime::Current();
  ....
  Thread::CreateNativeThread(env, java_thread, stack_size, daemon == JNI_TRUE);
}

void Thread::CreateNativeThread(JNIEnv* env, jobject java_peer, size_t stack_size, bool is_daemon) {
  ....
  int pthread_create_result = 0;
  if (child_jni_env_ext.get() != nullptr) {
    pthread_t new_pthread;
    pthread_attr_t attr;
    pthread_create_result = pthread_create(&new_pthread,
                                           &attr,
                                           gUseUserfaultfd ? Thread::CreateCallbackWithUffdGc
                                                           : Thread::CreateCallback,
                                           child_thread);
  }
  ...
}

void* Thread::CreateCallback(void* arg) {
  ...
  //调用 run 方法
  // Invoke the 'run' method of our java.lang.Thread.
  ObjPtr receiver = self->tlsPtr_.opeer;
  WellKnownClasses::java_lang_Thread_run->InvokeVirtual(self, receiver);
  ...
}
```

## 3> 参考来源

- [github.com/zhanghecn/sktrace](https://github.com/zhanghecn/sktrace)
- [github.com/zhanghecn/frida-trace](https://github.com/zhanghecn/frida-trace)
- [github.com/frida/frida-java-bridge](https://github.com/frida/frida-java-bridge)
