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
            /**
             * 销售
             * @param money
             */
            public void saleProduct(float money);
        
            /**
             * 售后
             * @param money
             */
            public void afterService(float money);
        }

- 生产者

        /**
         * 一个生产者
         */
        public class Producer implements IProducer {
        
            /**
             * 销售
             * @param money
             */
            public void saleProduct(float money) {
                System.out.println("销售产品，并拿到钱：" + money);
            }
        
            /**
             * 售后
             * @param money
             */
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
        
            /**
             * 销售
             * @param money
             */
            public void saleProduct(float money) {
                System.out.println("销售产品，并拿到钱：" + money);
            }
        
            /**
             * 售后
             * @param money
             */
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
(1). 把通知**Bean** 也交给 Spring 来管理

(2). 使用 **aop : config** 标签来表明开始 AOP 的设置

(3). 使用 **aop : aspect** 标签配置切面
&emsp;&emsp;id 属性：是给切面提供一个唯一标识
&emsp;&emsp;ref 属性：是指定通知类 Bean 的 id

(4). 在 **aop : aspect** 标签的内部使用对应标签来配置通知的类型

&emsp;&emsp; **aop :before** 标识前置通知

&emsp;&emsp;**method 属性:** 用于指定类中哪个放啊是前置通知

&emsp;&emsp;**pointcut 属性：** 用于指定切入点表达式，该切入点表达式指的是对业务层中哪些方法增强  

&emsp;&emsp;**切入点表达式的写法：**
&emsp;&emsp;&emsp;&emsp;-关键字：**execution ( 表达式 )**
&emsp;&emsp;&emsp;&emsp;- 表达式：
&emsp;&emsp;&emsp;&emsp;- 标准写法：访问修饰符 + 返回值 + 包名.类名.方法名（参数列表）
&emsp;&emsp;&emsp;&emsp;- 举例：public void com.greyson.service.impl.IAccountServiceImpl.saveAccount ( )
&emsp;&emsp;&emsp;&emsp;- 全通配写法：`pointcut="execution(* * ..*.*(..))"`
&emsp;&emsp;&emsp;&emsp;- 访问修饰符可以省略
&emsp;&emsp;&emsp;&emsp;- 返回值可以使用通配符，表示任意返回值
&emsp;&emsp;&emsp;&emsp;- 包名可以使用通配符，表示任意包，但是有几级包就需要写几个 `*.`
&emsp;&emsp;&emsp;&emsp;- 包名可以使用  `..` 表示当前包和子包
&emsp;&emsp;&emsp;&emsp;- 类名和方法名都可以使用  `*` 来实现通配

&emsp;&emsp;&emsp;&emsp;- 参数列表：
                - 可以直接使写数据类型：
                    - 基本类型直接写名称（如 int ）
                    - 引用类型写包名.类名的方式 （如 java.lang.String ）
                - 可以使用通配符表四任意类型，但是必须有参数
                - 可以使用 `..` 表示有无参数即可，有参数可以是任意类型
        - 实际开发中切入点表达式的通常写法：
            - 切到业务层类实现下的所有方法：`pointcut="execution(* com.greyson.service.impl.*.*(..))"`
        - 配置切入点表达式（aop : pointcut）：
            - id属性用于指定表达式的唯一标识，expression属性用于指定表达式内容
            - 此标签写在 aop : aspect 标签内部只能当前切面使用，在其外部则所有切面可用
              
### 3. Spring常用通知类型
        - 前置通知（aop : before）：在切入点方法执行之前执行
        - 后置通知（aop : after-returning）：在切入点方法正常执行之后执行，它和异常通知永远只能执行一个
        - 异常通知（aop : after-throwing）：在切入点方法执行产生异常之后执行，它和后置通知永远只能执行一个
        - 最终通知（aop : after）：无论切入点方法是否正常执行它都会在其后面执行
### 4.环绕通知
