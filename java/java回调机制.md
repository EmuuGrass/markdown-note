##JAVA回调机制

 模块之间总是存在这一定的接口，从调用方式上看，可以分为三类：同步调用、回调和异步调用。同步调用是一种阻塞式调用，也是我们在写程序中经常使用的；回调是一种双向的调用模式，也就是说，被调用的接口被调用时也会调用对方的接口，这句话可能有点绕，等文章后面举例说明；异步调用是一种类似消息或事件的机制，解决了同步阻塞的问题，举例来讲：A通知B后，他们各走各的路，互不影响，不用像同步调用那样，A通知B后，非得等到B走完后，A才继续走。回调是异步的基本，因此下面着重说回调机制。


首先创建一个回调接口，让老板得告知干完活如何找到他的方式：留下老板办公室地址：

	package net.easyway.test;  
	  
	/** 
	 * 此接口为联系的方式，不论是电话号码还是联系地址，作为 
	 * 老板都必须要实现此接口 
	 * @author Administrator 
	 * 
	 */  
	public interface CallBackInterface {  
	  
	    public void execute();  
	}  

创建回调对象，就是老板本人，因为员工干完活后要给他打电话，因此老板必须实现回调接口，不然员工去哪里找老板？


	package net.easyway.test;  
	  
	/** 
	 * 老板是作为上层应用身份出现的，下层应用（员工）是不知道 
	 * 有哪些方法，因此他想被下层应用（员工）调用必须实现此接口 
	 * @author Administrator 
	 * 
	 */  
	public class Boss implements CallBackInterface {  
	      
	    @Override  
	    public void execute() {  
	        System.out.println("收到了！！" + System.currentTimeMillis());  
	          
	    }  
	}  
创建控制类，也就是员工对象，他必须持有老板的地址（回调接口），即使老板换了一茬又一茬，办公室不变，总能找到对应的老板。

	package net.easyway.test;  
	  
	/** 
	 * 员工类，必须要记住，这是一个底层类，底层是不了解上层服务的 
	 * @author Administrator 
	 * 
	 */  
	public class Employee {  
	  
	    private CallBackInterface callBack = null;  
	      
	    //告诉老板的联系方式，也就是注册  
	    public void setCallBack(CallBackInterface callBack){  
	        this.callBack = callBack;  
	    }  
	      
	    //工人干活  
	    public void doSome(){  
	        //1.开始干活了  
	        for(int i=0;i<10;i++){  
	            System.out.println("第【" + i + "】事情干完了！");  
	        }  
	          
	        //2.告诉老板干完了  
	        callBack.execute();  
	    }  
	}

测试类代码：


	package net.easyway.test;  
	  
	public class Client {  
	  
	    public static void main(String[] args) {  
	          
	          
	        Employee emp = new Employee();  
	          
	        //将回调对象（上层对象）传入，注册  
	        emp.setCallBack(new Boss());  
	          
	        //开启控制器对象运行  
	        emp.doSome();  
	    }  
	  
	}  

上面这个例子，大家可以和程序员A和程序员B的那个例子结合对照下。

看了上面的例子，有的人可能认为，这不是面向接口的编程吗？怎么会是回调，你再好好想想，咱们面向接口的编程的调用关系？在三层中，当业务层调用数据层时，是不需要把业务层自身传递到数据层的，并且这是一种上层调用下层的关系，比如我们在用框架的时候，一般直接调用框架提供的API就可以了，但回调不同，当框架不能满足需求，我们想让框架来调用自己的类方法，怎么做呢？总不至于去修改框架吧。许多优秀的框架提几乎都供了相关的接口，我们只需要实现相关接口，即可完成了注册，然后在合适的时候让框架来调用我们自己的类，还记不记得我们在使用Struts时，当我们编写Action时，就需要继承Action类，然后实现execute()方法，在execute()方法中写咱们自己的业务逻辑代码，完成对用户请求的处理。由此可以猜测，框架和容器中会提供大量的回调接口，以满足个性化的定制。

不知道上面这个例子懂了没有？我们现在可以想象Filter和Interceptor的区别了，这两者其中最大的一个区别是Filter是基于回调函数，需要容器的支持，没有容器是无法回调doFilter()方法，而Interceptor是基于Java的反射机制的，和容器无关。那到此是否又将反射和回调搞混了呢？请见我讲Java动态代理的博客《以此之长，补彼之短----AOP(代理模式)》。

总之，要明确的一点是，首先要搞清回调函数出现的原因，也就是适用场景，才能搞清楚回调机制，不然事倍功半。

最后，再举一例，为了使我们写的函数接近完美，就把一部分功能外包给别人，让别人个性化定制，至于别人怎么实现不管，我唯一要做的就是定义好相关接口，这一设计允许了底层代码调用高层定义的子程序，增强程序灵活性，和反射有着异曲同工之妙，我觉得这才是回调的真正原因，以上是我个人一些理解，望讨论！

