---
sidebar_position: 100
---

# Access other beans

In some cases, you may need to access other beans from within your current bean to perform actions or retrieve 
information. For example, consider a bean that implements a good-night routine, which closes covers, turns off all 
lights, and performs other tasks. If you want to trigger this routine by pressing a hardware button next to your bed, 
you would typically create a separate bean for the button and invoke the good-night routine from the original bean.

First, take a look at the good-night routine bean. It defines the method `triggerRoutine()`, which we want to invoke 
from the hardware button bean.

````java
public class GoodNightRoutine implements SmartBean {

  public void triggerRoutine() {
    closeAllCovers();
    turnOffAllLights();
  }
}
````

## Bean proxy

You can access other bean instances through the `SmartBeans` API, which can be injected into your bean by declaring a 
field of type `SmartBeans`. This API provides the `getBean()` method in two variants. The first can be used when there
is only one instance of the target bean by simply passing the Java class as a parameter. The second variant accepts the
bean name as an additional parameter, as defined by the `@SmartBeanDef` annotation.

This method does not return the bean instance directly, but a `BeanProxy` object, which can be used to invoke methods on 
the target bean. The proxy is necessary because each bean runs in its own thread. To call a method of another bean, the 
execution must occur in the target bean's thread rather than in the calling bean's thread. 

The `BeanProxy` handles this by executing the requested method in the target bean's thread and, if needed, returning the 
result to the calling bean's thread. Because of this threading model, you can invoke methods either synchronously or 
asynchronously. Synchronous calls block the calling bean's thread until the method completes, while asynchronous calls 
return immediately, with the method execution occurring shortly thereafter in the target bean's thread.

In our example, the good-night routine can be invoked asynchronously, which is preferred whenever possible to avoid 
unnecessary blocking. For synchronous execution, you can use the `invokeSync()` method as an alternative to `invoke()`.

````java
public class HardwareButton implements SmartBean {
  
  private SmartBeans sb;
  
  public void onButtonPressed() {
    sb.getBean(GoodNightRoutine.class)
        .invoke(GoodNightRoutine::triggerRoutine);
  }
}
````

## Returned values

`invoke()` is used to call methods, that don't have any return values. If the method you want to call has a return value
and you want to get this, use `call()` or `callSync()` instead. Where `callSync()` returns the value direcly, `call()` returns
it as a `BeanFuture` object, similar to the known `Future` object in Java core library.

The `invoke()` method is used to call methods that do not return a value. If the method you want to call has a return 
value, use `call()` or `callSync()` instead. While `callSync()` returns the value directly, `call()` returns it as a 
`BeanFuture` object, similar to the standard `Future` in the Java core library.

