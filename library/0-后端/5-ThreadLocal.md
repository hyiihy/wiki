# <center>ThreadLocal</center>

## ThreadLocal
线程变量，用来从空间上解决多并发问题。  
ThreadLocal为每个使用该变量的线程提供独立的变量副本。  
在ThreadLocal类中有一个Map，用于存储每一个线程的变量副本，Map中元素的键为线程对象，而值对应线程的变量副本。  
Spring对一些有状态的Bean采用ThreadLocal进行处理，使其线程安全（如RequestContextHolder、TransactionSynchronizationManager、LocaleContextHolder等）。  

## 四个方法
- void set(T value)
- T get()
- public void remove()
- T initialValue()
