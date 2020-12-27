# Java 线程的supend()、resume()、stop() 

```java
	final static DateTimeFormatter DATE_TIME_FORMATTER = DateTimeFormatter.ofPattern("HH:mm:ss");

    public static void main(String[] args) throws Exception {

        Thread thread = new Thread(new Runner(), "PrintThread");
        thread.setDaemon(true);
        thread.start();
        TimeUnit.SECONDS.sleep(3);
        // thread 进行暂停 ， 输出内容工作停止
        thread.suspend();
        System.out.println("main suspend PrintThread at " + LocalTime.now().format(DATE_TIME_FORMATTER));
        TimeUnit.SECONDS.sleep(3);
        // thread 进行恢复，输出内容继续
        thread.resume();
        System.out.println("main resume PrintThread at " + LocalTime.now().format(DATE_TIME_FORMATTER));
        TimeUnit.SECONDS.sleep(3);
        // thread 终止，输出内容停止
        thread.stop();
        System.out.println("main stop PrintThread at " + LocalTime.now().format(DATE_TIME_FORMATTER));
        TimeUnit.SECONDS.sleep(3);
        
    }

    static class Runner implements Runnable {

        @Override
        public void run() {
            while (true) {
                System.out.println(Thread.currentThread().getName() + " Run at " + LocalTime.now().format(DATE_TIME_FORMATTER));
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }

    }
```

Java线程supend()、resume()、stop()这些方法完成了线程的暂停、恢复、终止工作，非常人性化。但是这些API都是过期的，官方不建议使用。

不建议的原因：
	suspend() ： 在调用后，线程不会释放已经占有的资源（比如锁），而是占有着资源进入睡眠状态，容易引发死锁的问题。
	比如线程A拿到锁A被suspend()进入了blocked状态，等待线程B调用resume将线程A重新runnable。但是线程B一直在lock pool重等待锁A，线程B要拿到锁A才能running去执行resumeA。容易造成死锁问题。
	stop() ：在终结一个线程时不会保证线程的资源正常释放，通常是没有给与线程完成资源释放工作的机会，会导致程序可能工作在不确定的状态下导致数据得不到同步的处理，出现数据不一致的问题。
	比如线程A转账（获取锁，川川账号减少一百块，张三账号增加一百块，释放锁），线程A刚执行川川账号减少一百块就被调用了stop()方法，释放了锁的资源，CPU资源。川川平白无故少了一百块。这时候川川很伤心...

supend()、resume()、stop()方法带来的副作用，这些方法才标注为不建议使用过期的方法，而执行和恢复操作可以用等待/通知机制来替代。

