# 一 概述

AMS是系统的引导服务，应用进程的启动、切换和调度、四大组件的启动和管理都需要AMS的支持。

# 二 AMS的启动流程

1.AMS的启动是在SyetemServer进程中启动的，在“run”中调用“startBootstrapServices()”；

2.在startBootstrapServices()再调用mSystemServiceManager.startService，并传入ActivityTaskManagerService.Lifecycle.class；

```java
Installer installer = mSystemServiceManager.startService(Installer.class);

ActivityTaskManagerService atm = mSystemServiceManager.startService(ActivityTaskManagerService.Lifecycle.class).getService();
mActivityManagerService = ActivityManagerService.Lifecycle.startService(mSystemServiceManager, atm);
mActivityManagerService.setSystemServiceManager(mSystemServiceManager);
mActivityManagerService.setInstaller(installer);
mWindowManagerGlobalLock = atm.getGlobalLock();
```

3.startService方法传入的参数是Lifecycle.class，Lifecycle继承自SystemService。首先，通过反射来创建Lifecycle实例，注释1处得到传进来的Lifecycle的构造器constructor，在注释2处调用constructor的newInstance方法来创建Lifecycle类型的service对象。接着在注释3处将刚创建的service添加到ArrayList类型的mServices对象中来完成注册。最后在注释4处调用service的onStart方法来启动service，并返回该service。

```java
@SuppressWarnings("unchecked")
  public <T extends SystemService> T startService(Class<T> serviceClass) {
      try {
         ...
          final T service;
          try {
              Constructor<T> constructor = serviceClass.getConstructor(Context.class);//1
              service = constructor.newInstance(mContext);//2
          } catch (InstantiationException ex) {
            ...
          }
          // Register it.
          mServices.add(service);//3
          // Start it.
          try {
              service.onStart();//4
          } catch (RuntimeException ex) {
              throw new RuntimeException("Failed to start service " + name
                      + ": onStart threw an exception", ex);
          }
          return service;
      } finally {
          Trace.traceEnd(Trace.TRACE_TAG_SYSTEM_SERVER);
      }
  }
```

4.Lifecycle如下：

```java
public static final class Lifecycle extends SystemService {
     private final ActivityManagerService mService;
     public Lifecycle(Context context) {
         super(context);
         mService = new ActivityManagerService(context);//1
     }
     @Override
     public void onStart() {
         mService.start();//2
     }
     public ActivityManagerService getService() {
         return mService;//3
     }
 }
```

