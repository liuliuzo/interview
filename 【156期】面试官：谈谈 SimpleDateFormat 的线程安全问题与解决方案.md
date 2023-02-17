假如说日期函数处理类使用的是new SimpleDateFormat的方法,那么这里就会有1000次sdf的创建和销毁
Java中提供了一种ThreadLocal的解决方案,它的工作方式是,每个线程只会有一个实例,也就是说我们执行完这1000个任务,总共只会实例化4个sdf.

```
package com.liuliu.learning;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

public class DateUtil {

	/** 锁对象 */
	private static final Object lockObj = new Object();

	/** 存放不同的日期模板格式的sdf的Map */
	private static Map<String, ThreadLocal<SimpleDateFormat>> sdfMap = new HashMap<String, ThreadLocal<SimpleDateFormat>>();

	/**
	 * 返回一个ThreadLocal的sdf,每个线程只会new一次sdf
	 * 
	 * @param pattern
	 * @return
	 */
	private static SimpleDateFormat getSdf(final String pattern) {
		ThreadLocal<SimpleDateFormat> tl = sdfMap.get(pattern);

		// 此处的双重判断和同步是为了防止sdfMap这个单例被多次put重复的sdf
		if (tl == null) {
			synchronized (lockObj) {
				tl = sdfMap.get(pattern);
				if (tl == null) {
					// 只有Map中还没有这个pattern的sdf才会生成新的sdf并放入map
					System.out.println("put new sdf of pattern " + pattern + " to map");

					// 这里是关键,使用ThreadLocal<SimpleDateFormat>替代原来直接new SimpleDateFormat
					tl = new ThreadLocal<SimpleDateFormat>() {

						@Override
						protected SimpleDateFormat initialValue() {
							System.out.println("thread: " + Thread.currentThread() + " init pattern: " + pattern);
							return new SimpleDateFormat(pattern);
						}
					};
					sdfMap.put(pattern, tl);
				}
			}
		}

		return tl.get();
	}

	/**
	 * 是用ThreadLocal<SimpleDateFormat>来获取SimpleDateFormat,这样每个线程只会有一个SimpleDateFormat
	 * 
	 * @param date
	 * @param pattern
	 * @return
	 */
	public static String format(Date date, String pattern) {
		return getSdf(pattern).format(date);
	}

	public static Date parse(String dateStr, String pattern) throws ParseException {
		return getSdf(pattern).parse(dateStr);
	}

}
```
测试类
```
package com.liuliu.learning;

import java.text.ParseException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class Test {

	public static void main(String[] args) {
		final String patten1 = "yyyy-MM-dd";
		final String patten2 = "yyyy-MM";

		Thread t1 = new Thread() {

			@Override
			public void run() {
				try {
					DateUtil.parse("1992-09-13", patten1);
				} catch (ParseException e) {
					e.printStackTrace();
				}
			}
		};

		Thread t2 = new Thread() {

			@Override
			public void run() {
				try {
					DateUtil.parse("2000-09", patten2);
				} catch (ParseException e) {
					e.printStackTrace();
				}
			}
		};

		Thread t3 = new Thread() {

			@Override
			public void run() {
				try {
					DateUtil.parse("1992-09-13", patten1);
				} catch (ParseException e) {
					e.printStackTrace();
				}
			}
		};

		Thread t4 = new Thread() {

			@Override
			public void run() {
				try {
					DateUtil.parse("2000-09", patten2);
				} catch (ParseException e) {
					e.printStackTrace();
				}
			}
		};

		Thread t5 = new Thread() {

			@Override
			public void run() {
				try {
					DateUtil.parse("2000-09-13", patten1);
				} catch (ParseException e) {
					e.printStackTrace();
				}
			}
		};

		Thread t6 = new Thread() {

			@Override
			public void run() {
				try {
					DateUtil.parse("2000-09", patten2);
				} catch (ParseException e) {
					e.printStackTrace();
				}
			}
		};

		System.out.println("单线程执行: ");
		ExecutorService exec = Executors.newFixedThreadPool(1);
		exec.execute(t1);
		exec.execute(t2);
		exec.execute(t3);
		exec.execute(t4);
		exec.execute(t5);
		exec.execute(t6);
		exec.shutdown();

		sleep(1000);

		System.out.println("双线程执行: ");
		ExecutorService exec2 = Executors.newFixedThreadPool(2);
		exec2.execute(t1);
		exec2.execute(t2);
		exec2.execute(t3);
		exec2.execute(t4);
		exec2.execute(t5);
		exec2.execute(t6);
		exec2.shutdown();
	}

	private static void sleep(long millSec) {
		try {
			TimeUnit.MILLISECONDS.sleep(millSec);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}
```
输出结果
```
单线程执行: 
put new sdf of pattern yyyy-MM-dd to map
thread: Thread[pool-1-thread-1,5,main] init pattern: yyyy-MM-dd
put new sdf of pattern yyyy-MM to map
thread: Thread[pool-1-thread-1,5,main] init pattern: yyyy-MM
双线程执行: 
thread: Thread[pool-2-thread-1,5,main] init pattern: yyyy-MM-dd
thread: Thread[pool-2-thread-2,5,main] init pattern: yyyy-MM
```

1. 个线程执行这6个任务的时候,这个线程首次使用过的时候会new一个新的sdf,并且以后都一直用这个sdf,而不是每次处理任务都新建一个新的sdf

2. 个线程执行6个任务的时候也是同理,但是2个线程的sdf是分开的,每个线程都有自己的"yyyy-MM-dd", "yyyy-MM"的sdf,所以他们不会有线程安全安全问题

文章参考自：
[SimpleDateFormat的线程安全问题与解决方案](https://www.cnblogs.com/zemliu/p/3290585.html)