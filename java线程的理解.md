# 对java的Thread的理解  
最好不适用线程的子线程，直接调用线程，然后用rannable接口  
然后如果要公用一个参数，就是公用资源的时候，一定要在run方法的前面加上synchronized  

例子
猫和狗喝同一杯水（共用资源的问题）

	/**
	 * @author nienie
	 *
	 */
	
	public class test {
	
		/**
		 * @param args
		 */
		public static void main(String[] args) {
			// TODO 自动生成的方法存根
			test1 test2 = new test1();
			int n=10;
			test2.setW(n);
			System.out.println("一共" +n +"克水,dog和cat每次喝1克水");
			Thread dog,cat;
			dog = new Thread(test2);
			cat = new Thread(test2);
			dog.setName("dog");
			cat.setName("cat");
			dog.start();
			cat.start();
		}
	
	}
	public class test1 implements Runnable{
		int w = 10;
		public void setW(int n){
			w = n;
		}
		public synchronized void run() {
			String name = Thread.currentThread().getName();
			while(name.equals("dog")){
				w = w-1;
				System.out.println(name+" 喝水，剩下 "+w+ " 克");
				
				notify();
				try {
					wait();
				} catch (InterruptedException e) {
					// TODO 自动生成的 catch 块
					e.printStackTrace();
				}
				if(w==0) System.exit(0);
			}
			while(name.equals("cat")){
				w = w-1;
				System.out.println(name+" 喝水，剩下 "+w + " 克");
				notify();
				try {
					wait();
				} catch (InterruptedException e) {
					// TODO 自动生成的 catch 块
					e.printStackTrace();
				}
				if(w==0) System.exit(0);
			}
		}	
	}
运行结果  

	一共10克水,dog和cat每次喝1克水
	dog 喝水，剩下 9 克
	cat 喝水，剩下 8 克
	dog 喝水，剩下 7 克
	cat 喝水，剩下 6 克
	dog 喝水，剩下 5 克
	cat 喝水，剩下 4 克
	dog 喝水，剩下 3 克
	cat 喝水，剩下 2 克
	dog 喝水，剩下 1 克
	cat 喝水，剩下 0 克

这里要注意的是一定要有那个synchronized，这个的作用相当于一个锁，锁定某一个线程先做好，但是如果直接使用，有一个矛盾，就是无法实现两个线程交替地使用，这个会导致等这个线程结束了之后，然后再执行第二个线程，哪怕有sleep也不行的，所以这里就灵活的使用wait（）方法，这个函数会将这个线程停止，然后让出cpu，让cpu执行其他线程，但是有一点要注意的是，在执行了wait之后，要使用notify方法激活另外的线程，不然的话，就会导致全部的线程都停止，然后程序就停下来了。
	
## 没有synchronized的情况  


	public class test1 implements Runnable{
		int w = 10;
		public void setW(int n){
			w = n;
		}
		public void run() {
			String name = Thread.currentThread().getName();
			while(name.equals("dog")){
				w = w-1;
				System.out.println(name+" 喝水，剩下 "+w+ " 克");
				notify();
				try {
					Thread.sleep(1000);
					wait();
				} catch (InterruptedException e) {
					// TODO 自动生成的 catch 块
					e.printStackTrace();
				}
				if(w==0) System.exit(0);
			}
			while(name.equals("cat")){
				w = w-1;
				System.out.println(name+" 喝水，剩下 "+w + " 克");
				notify();
				try {
					Thread.sleep(1000);
					wait();
				} catch (InterruptedException e) {
					// TODO 自动生成的 catch 块
					e.printStackTrace();
				}
				if(w==0) System.exit(0);
			}
		}
	}

	运行结果：
	一共10克水,dog和cat每次喝1克水
	dog 喝水，剩下 9 克
	Exception in thread "dog" cat 喝水，剩下 8 克
	Exception in thread "cat" java.lang.IllegalMonitorStateException
		at java.lang.Object.notify(Native Method)
		at test1.run(test1.java:29)
		at java.lang.Thread.run(Unknown Source)
	java.lang.IllegalMonitorStateException
		at java.lang.Object.notify(Native Method)
		at test1.run(test1.java:16)
		at java.lang.Thread.run(Unknown Source)