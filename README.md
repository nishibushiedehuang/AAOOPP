# Spring AOP 学习笔记

## 一、Spring AOP的概念

### 1.Spring AOP 是什么？
在单体架构下的软件开发中，一个大型项目通常是依照功能拆分成各个模块。但是如日志、安全和事务管理此类重要且繁琐的开发却没有必要参与到各个模块中，将这些功能与业务逻辑相关的模块分离就是面向切面编程所要解决的问题  
  
**AOP采取的是横向抽取机制，取代了传统纵向继承体系重复性代码。**

### 2.AOP 的作用与优势
**作用：** 在程序运行期间，不修改源码对已有方法进行增强。    
  
**优势：**  
&emsp;&emsp;1.集中处理某一关注点/横切逻辑  
&emsp;&emsp;2.可以很方便地添加/删除关注点  
&emsp;&emsp;3.侵入性少，增强代码可读性及可维护性  

### 3.使用场景
&emsp;&emsp;权限控制、缓存控制、事务控制、审计日志、性能监控、分布式追踪、异常处理  

### 4.AOP底层原理
使用动态代理实现  
  
（1）基于JDK的代理

    适用于有接口情况，使用动态代理创建接口实现类代理对象

（2）基于CGLIB动态代理

    适用于没有接口情况，使用动态代理创建类的子类代理对象

## 二、什么是代理？

简单理解，本来厂商可以自产自销，但是由于各种开销，最后厂商选择只生产产品，销售则交由各级经销商完成。

![](-d7c45db4-1f69-4b5c-b5f5-a9624176e6f8.png)

- **特点:** 字节码随用随创建，随用随加载
- **作用：** 不修改源码的基础上对方法增强
- **分类：**  
&emsp;&emsp;基于**接口**的动态代理  
&emsp;&emsp;基于**子类**的动态代理

### 1、基于接口的动态代理

&emsp;&emsp;涉及的类：`Proxy`

&emsp;&emsp;提供者：`JDK官方`

&emsp;&emsp;**如何创建代理对象：**

&emsp;&emsp;&emsp;&emsp;使用Proxy类中的`newProxyInstance`方法

&emsp;&emsp;**创建代理对象的要求：**

&emsp;&emsp;&emsp;&emsp;被代理类最少实现的一个接口，如果没有则不能使用

&emsp;&emsp;**`newProxyInstance`方法的参数：**

&emsp;&emsp;&emsp;&emsp;`ClassLoader`: 用于加载代理对象字节码，和被代理对象使用相同的类加载器，固定写法

&emsp;&emsp;&emsp;&emsp;`Class [ ]` : 用于让代理对象和被代理对象有相同的方法，固定写法

&emsp;&emsp;&emsp;&emsp;`InvocationHandler` : 用于提供增强的代码

&emsp;&emsp;&emsp;&emsp;它是让我们写如何代理。我们一般是写一个该接口的实现类，通常是匿名内部类，但不是必须的。此接口的实现类都是谁用谁写。

- 生产厂家接口IProducer

        /**
         * 对生产厂家要求的接口
         */
        public interface IProducer {
             // 销售
            public void saleProduct(float money);
       
             //售后
            public void afterService(float money);
        }

- 生产者

        /**
         * 一个生产者
         */
        public class Producer implements IProducer {
        
            // 销售
            public void saleProduct(float money) {
                System.out.println("销售产品，并拿到钱：" + money);
            }
            // 售后
            public void afterService(float money) {
                System.out.println("提供售后服务，并拿到钱：" + money);
            }
        }

- 消费者

        /**
         * 模拟一个消费者
         */
        public class Client {
            public static void main(String[] args) {
                final Producer producer = new Producer();
        
                IProducer proxyProducer = (IProducer) Proxy.newProxyInstance(producer.getClass().getClassLoader(),
                        producer.getClass().getInterfaces(), new InvocationHandler() {
                            /**
                             * 作用：执行被代理对象的任何接口方法都会经过该方法
                             * 方法参数的含义：
                             * @param proxy         代理对象的含义
                             * @param method        当前执行的方法
                             * @param args          当前执行方法的参数
                             * @return              和被代理对象方法有相同的返回值
                             * @throws Throwable
                             */
                            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                                // 提供增强的代码
                                Object returnValue = null;
                                // 1.获取方法执行的参数
                                Float money = (Float)args[0];
                                // 2.判断当前方法是不是销售
                                if ("saleProduct".equals(method.getName())) {
                                    returnValue = method.invoke(producer,money * 0.8f);
                                }
                                return returnValue;
                            }
                        });
            }
        }

### 2、基于子类的动态代理

&emsp;&emsp;涉及的类：Enhancer

&emsp;&emsp;提供者：第三方 cglib 库

&emsp;&emsp;**如何创建代理对象：**

&emsp;&emsp;&emsp;&emsp;使用 Enhancer 类中的 create 方法

&emsp;&emsp;**创建代理对象的要求：**

&emsp;&emsp;&emsp;&emsp;被代理类不能是最终类

&emsp;&emsp;**create 方法的参数：**

&emsp;&emsp;&emsp;&emsp; Class : 它是用于被指定代理对象的字节码

&emsp;&emsp;&emsp;&emsp;callback : 用于提供增强的代码

&emsp;&emsp;&emsp;&emsp;它是让我们写如何代理。我们一般是写一个该接口的实现类，通常是匿名内部类，但不是必须的。此接口的实现类都是谁用谁写。我们一般写的都是该接口的子实现类：MethodInterCeptor

- 生产者

        public class Producer {
          // 销售
            public void saleProduct(float money) {
                System.out.println("销售产品，并拿到钱：" + money);
            }
          // 售后
            public void afterService(float money) {
                System.out.println("提供售后服务，并拿到钱：" + money);
            }
        }
- 消费者

        /**
         * 模拟一个消费者
         */
        public class Client {
            public static void main(String[] args) {
                final Producer producer = new Producer();
                Producer cglibProducer = (Producer) Enhancer.create(producer.getClass(), new MethodInterceptor() {
                    public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
                        // 提供增强的代码
                        Object returnValue = null;
                        // 1.获取方法执行的参数
                        Float money = (Float)args[0];
                        // 2.判断当前方法是不是销售
                        if ("saleProduct".equals(method.getName())) {
                            returnValue = method.invoke(producer,money * 0.8f);
                        }
                        return returnValue;
                    }
                });
                cglibProducer.saleProduct(12000f);
            }
        }
## 三、AOP实例
### 1.AOP术语

- **Advice (通知/增强):**  
&emsp;&emsp;所谓通知是指拦截到 Joinpoint 之后所要做的事情就是通知。通知的类型：前置通知，后置通知，异常通知，最终通知，环绕通知。

- **Joinpoint (连接点):**  
&emsp;&emsp;所谓连接点是指那些被拦截到的点。在 Spring中，这些点指的是方法，因为 Spring 只支持方法类型的连接点。

- **Pointcut (切入点):**  
&emsp;&emsp;所谓切入点是指我们要对哪些 Joinpoint 进行拦截的定义。

- **Introduction (引介):**  
&emsp;&emsp;引介是一种特殊的通知在不修改类代码的前提下, Introduction 可以在运行期为类动态地添加一些方法或 Field。   

- **Target (目标对象):**  
&emsp;&emsp;代理的目标对象。

- **Weaving (织入):**  
&emsp;&emsp;是指把增强应用到目标对象来创建新的代理对象的过程。Spring 采用动态代理织入，而 AspectJ 采用编译期织入和类装载期织入。

- **Proxy (代理):**   
&emsp;&emsp;一个类被 AOP 织入增强后，就产生一个结果代理类。   

- **Aspect (切面):**  
&emsp;&emsp;是切入点和通知（引介）的结合。

### 2.Spring中基于 xml 的 AOP 配置步骤
   <!--配置AOP-->
      <aop:config>
          <!--配置切面 -->
          <aop:aspect id="logAdvice" ref="logger">
              <!-- 配置通知的类型，并且建立通知方法和切入点方法的关联-->
              <aop:before method="printLog" pointcut="execution(* com.itheima.service.impl.*.*(..))"></aop:before>
          </aop:aspect>
      </aop:config>
(1)把通知**Bean** 也交给 Spring 来管理

(2)使用 **aop : config** 标签来表明开始 AOP 的设置

(3)使用 **aop : aspect** 标签配置切面  
&emsp;&emsp;id 属性：是给切面提供一个唯一标识   
&emsp;&emsp;ref 属性：是指定通知类 Bean 的 id  

(4)在**aop : aspect** 标签的内部使用对应标签来配置通知的类型

&emsp;&emsp;**aop :before** 标识前置通知

&emsp;&emsp;**method 属性:** 用于指定类中哪个方法是前置通知

&emsp;&emsp;**pointcut 属性：** 用于指定切入点表达式，该切入点表达式指的是对业务层中哪些方法增强  

&emsp;&emsp;**切入点表达式的写法：**  
- 关键字：**execution ( 表达式 )**  
- 表达式：```访问修饰符 + 返回值 + 包名...类名.方法名（参数列表）```  
- 标准的表达式写法:```public void com.greyson.service.impl.IAccountServiceImpl.saveAccount()```  
- 全通配写法：``` pointcut="execution(* * ..*.*(..))"```  
- 访问修饰符可以省略：``` void com.itheima.service.impl.AccountServiceImpl.saveAccount()```  
- 返回值可以使用通配符，表示任意返回值：``` * com.itheima.service.impl.AccountServiceImpl.saveAccount()```      
- 包名可以使用通配符，表示任意包，但是有几级包就需要写几个 `*.`：  
&emsp;&emsp;``` * *.*.*.*.AccountServiceImpl.saveAccount())```  
- 包名可以使用 `..` 表示当前包和子包：```* *..AccountServiceImpl.saveAccount()```    
- 类名和方法名都可以使用  `*` 来实现通配：```* *..*.*()```  
- 参数列表：  
&emsp;&emsp;- 可以直接使写数据类型：  
&emsp;&emsp;&emsp;&emsp;- 基本类型直接写名称（如 int ）  
&emsp;&emsp;&emsp;&emsp;- 引用类型写包名.类名的方式 （如 java.lang.String ）  
&emsp;&emsp;- 可以使用通配符表四任意类型，但是必须有参数  
&emsp;&emsp;- 可以使用 `..` 表示有无参数即可，有参数可以是任意类型  

&emsp;&emsp;**实际开发中切入点表达式的通常写法：**    
- 切到业务层类实现下的所有方法：`pointcut="execution(* com.greyson.service.impl.*.*(..))"`  

### 3.创建一个AOP实例（Spring_AOP）  
(1)创建一个Maven project  
   
(2)创建账户的业务层接口及实现类  
```
  package com.learn.service;
  /**
   * 账户的业务层接口
   */
  public interface IAccountService {
      /**
       * 模拟保存账户
       */
     void saveAccount();
      /**
       * 模拟更新账户
       * @param i
       */
     void updateAccount(int i);
      /**
       * 删除账户
       * @return
       */
     int  deleteAccount();
  }
  ```    
  
```
  package com.learn.service.impl;
  
  import com.learn.service.IAccountService;
  /**
   * 账户的业务层实现类
   */
  public class AccountServiceImpl implements IAccountService{

      @Override
      public void saveAccount() {
          System.out.println("执行了保存");
      }

      @Override
      public void updateAccount(int i) {
          System.out.println("执行了更新"+i);

      }

      @Override
      public int deleteAccount() {
          System.out.println("执行了删除");
          return 0;
      }
  }
```  
(3)创建一个工具类，让其在切入点方法之前执行  
```
package com.learn.utils;

/**
 * 用于记录日志的工具类，它里面提供了公共的代码
 */
public class Logger {

    /**
     * 用于打印日志：计划让其在切入点方法执行之前执行（切入点方法就是业务层方法）
     */
    public  void printLog(){
        System.out.println("Logger类中的pringLog方法开始记录日志了。。。");
    }
}
```  
(4)配置bean.xml文件  
```
    <!-- 配置srping的Ioc,把service对象配置进来-->
    <bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl"></bean>
    
    <!-- 配置Logger类 -->
    <bean id="logger" class="com.itheima.utils.Logger"></bean>
    
    <!--配置AOP-->
    <aop:config>
        <!--配置切面 -->
        <aop:aspect id="logAdvice" ref="logger">
            <!-- 配置通知的类型，并且建立通知方法和切入点方法的关联-->
            <aop:before method="printLog" pointcut="execution(* com.itheima.service.impl.*.*(..))"></aop:before>
        </aop:aspect>
    </aop:config>
```
(5)编写AOP测试类，运行，前置通知织入成功！
```
package com.learn.test;

import com.learn.service.IAccountService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * 测试AOP的配置
 */
public class AOPTest {

    public static void main(String[] args) {
        //1.获取容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2.获取对象
        IAccountService as = (IAccountService)ac.getBean("accountService");
        //3.执行方法
        as.saveAccount();
        as.updateAccount(1);
        as.deleteAccount();
    }
}
```

## 四、Spring常用通知类型（AdviceType）
### 1.前置通知、后置通知、异常通知与最终通知    
  前置通知（aop : before）：在切入点方法执行之前执行
  后置通知（aop : after-returning）：在切入点方法正常执行之后执行，它和异常通知永远只能执行一个
  异常通知（aop : after-throwing）：在切入点方法执行产生异常之后执行，它和后置通知永远只能执行一个
  最终通知（aop : after）：无论切入点方法是否正常执行它都会在其后面执行  
### 2.实例
(1)增加工具类方法  
```
public class Logger {

    /**
     * 前置通知
     */
    public  void beforePrintLog(){
        System.out.println("前置通知Logger类中的beforePrintLog方法开始记录日志了。。。");
    }

    /**
     * 后置通知
     */
    public  void afterReturningPrintLog(){
        System.out.println("后置通知Logger类中的afterReturningPrintLog方法开始记录日志了。。。");
    }
    /**
     * 异常通知
     */
    public  void afterThrowingPrintLog(){
        System.out.println("异常通知Logger类中的afterThrowingPrintLog方法开始记录日志了。。。");
    }

    /**
     * 最终通知
     */
    public  void afterPrintLog(){
        System.out.println("最终通知Logger类中的afterPrintLog方法开始记录日志了。。。");
    }
```  
(2)配置bean.xml  
```
    <!--配置AOP-->
    <aop:config>
        <!-- 配置切入点表达式 id属性用于指定表达式的唯一标识。expression属性用于指定表达式内容
              此标签写在aop:aspect标签内部只能当前切面使用。
              它还可以写在aop:aspect外面，此时就变成了所有切面可用
          -->
        <aop:pointcut id="pt1" expression="execution(* com.itheima.service.impl.*.*(..))"></aop:pointcut>
        <!--配置切面 -->
        <aop:aspect id="logAdvice" ref="logger">
            <!-- 配置前置通知：在切入点方法执行之前执行
            <aop:before method="beforePrintLog" pointcut-ref="pt1" ></aop:before>-->

            <!-- 配置后置通知：在切入点方法正常执行之后值。它和异常通知永远只能执行一个
            <aop:after-returning method="afterReturningPrintLog" pointcut-ref="pt1"></aop:after-returning>-->

            <!-- 配置异常通知：在切入点方法执行产生异常之后执行。它和后置通知永远只能执行一个
            <aop:after-throwing method="afterThrowingPrintLog" pointcut-ref="pt1"></aop:after-throwing>-->

            <!-- 配置最终通知：无论切入点方法是否正常执行它都会在其后面执行
            <aop:after method="afterPrintLog" pointcut-ref="pt1"></aop:after>-->
        </aop:aspect>
    </aop:config>
```
(3)运行测试类，织入三条通知  
### 2.环绕通知 
(1)工具类明确调用业务层方法
```
    public Object aroundPringLog(ProceedingJoinPoint pjp){
        Object rtValue = null;
        try{
            Object[] args = pjp.getArgs();//

            System.out.println("Logger类中的beforePrintLog方法开始记录日志了。。。前置通知");

            rtValue = pjp.proceed(args);//明确调用业务层方法（切入点方法）

            System.out.println("Logger类中的afterReturningPrintLog方法开始记录日志了。。。后置通知");

            return rtValue;
        }catch (Throwable t){
            System.out.println("Logger类中的afterThrowingPrintLog方法开始记录日志了。。。异常通知");
            throw new RuntimeException(t);
        }finally {
            System.out.println("Logger类中的afterPrintLog方法开始记录日志了。。。最终通知");
        }
    }
```  
(2)配置bean.xml  
```
    <!--配置AOP-->
    <aop:config>
        <aop:pointcut id="pt1" expression="execution(* com.itheima.service.impl.*.*(..))"></aop:pointcut>
        <!--配置切面 -->
        <aop:aspect id="logAdvice" ref="logger">
            <!-- 配置环绕通知-->
            <aop:around method="aroundPringLog" pointcut-ref="pt1"></aop:around>
        </aop:aspect>
    </aop:config>
```
(3)运行测试类，织入三条通知  
