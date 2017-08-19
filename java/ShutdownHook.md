## ShutdownHook ##

在线上Java程序中经常遇到进程程挂掉，一些状态没有正确的保存下来，这时候就需要在JVM关掉的时候执行一些清理现场的代码。Java中得**ShutdownHook**提供了比较好的方案。
　　

JDK在1.3之后提供了Java **Runtime.addShutdownHook**(Thread hook)方法，可以注册一个JVM关闭的钩子，这个钩子可以在以下几种场景被调用

- 程序正常退出
- 使用System.exit()
- 终端使用Ctrl+C触发的中断
- 系统关闭
- 使用Kill pid命令干掉进程

		注：在使用kill -9 pid是不会JVM注册的钩子不会被调用。


###在JDK中方法的声明：
	public void addShutdownHook(Thread hook)
	参数
	hook -- 一个初始化但尚未启动的线程对象，注册到JVM钩子的运行代码。
	异常
	IllegalArgumentException -- 如果指定的钩已被注册，或如果它可以判定钩已经运行或已被运行
	IllegalStateException -- 如果虚拟机已经是在关闭的过程中
	SecurityException -- 如果存在安全管理器并且它拒绝的RuntimePermission（“shutdownHooks”）
### 代码示例： ###

	package com.netease.test.java.lang;
	
	import java.util.Timer;
	import java.util.TimerTask;
	import java.util.concurrent.atomic.AtomicInteger;
	
	/**
	 * Date: 14-6-18
	 * Time: 11:01
	 * 测试ShutdownHook
	 */
	public class TestShutdownHook {

    //简单模拟干活的
    static Timer timer = new Timer("job-timer");

    //计数干活次数
    static AtomicInteger count = new AtomicInteger(0);

    /**
     * hook线程
     */
    static class CleanWorkThread extends Thread{
        @Override
        public void run() {
            System.out.println("clean some work.");
            timer.cancel();
            try {
                Thread.sleep(2 * 1000);//sleep 2s
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    public static void main(String[] args) throws InterruptedException {
        //将hook线程添加到运行时环境中去
        Runtime.getRuntime().addShutdownHook(new CleanWorkThread());
        System.out.println("main class start ..... ");
        //简单模拟
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                count.getAndIncrement();
                System.out.println("doing job " + count);
                if (count.get() == 10) {  //干了10次退出
                    System.exit(0);
                }
            }
        }, 0, 2 * 1000);

    }
	}