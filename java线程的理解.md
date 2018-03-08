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

<br>
## 书本上面没有使用synchronized

	/**
	 * @author nienie
	 *
	 */
	
	public class Example12_2 {
	
		/**
		 * @param args
		 */
		public static void main(String[] args) {
			// TODO 自动生成的方法存根
			House house = new House();
			house.serWater(10);
			Thread dog,cat;
			dog = new Thread(house);
			cat = new Thread(house);
			dog.setName("dog");
			cat.setName("cat");
			dog.start();
			cat.start();
		}
	
	}  
	public class House implements Runnable{
		int waterAmount;
		public void serWater(int w){
			waterAmount = w;
		}
		public  void run() {
			int m = 1;
			while(true){
				if(waterAmount <=0){
					return;
				}
				waterAmount = waterAmount -m;
				System.out.println("剩 "+waterAmount+" 克");
				try{
					Thread.sleep(2000);
				}
				catch (Exception e) {}
			}
		}
	}  
	运行结果：
	剩 9 克
	剩 8 克
	剩 7 克
	剩 7 克
	剩 6 克
	剩 6 克
	剩 5 克
	剩 5 克
	剩 4 克
	剩 4 克
	剩 3 克
	剩 3 克
	剩 2 克
	剩 2 克
	剩 0 克
	剩 0 克  

明显的出现了很多重复的，所以这里就要用到synchronized

##书本上使用synchronized的另外一个例子  
	/**
	 * @author nienie
	 *
	 */
	
	public class Example12_5 {
	
		/**
		 * @param args
		 */
		public static void main(String[] args) {
			// TODO 自动生成的方法存根
			Bank bank = new Bank();
			bank.setMoney(200);
			Thread accountant,cashier;
			accountant = new Thread(bank);
			cashier = new Thread(bank);
			accountant.setName("会计");
			cashier.setName("出纳");
			accountant.start();
			cashier.start();
		}
	}

	public class Bank implements Runnable{
		int money = 200;
		public void setMoney(int n){
			money = n;
		}
		public void run() {
			// TODO 自动生成的方法存根
			if(Thread.currentThread().getName().equals("会计"))
				saveOtTake(300);
			else if(Thread.currentThread().getName().equals("出纳"))
				saveOtTake(150);
		}
		public synchronized void saveOtTake(int amount) {
			// TODO 自动生成的方法存根
			if(Thread.currentThread().getName().equals("会计")){
				for (int i=1;i<3;i++){
					money = money + amount/3;
					System.out.println(Thread.currentThread().getName()+" 存入 "+amount/3+" ,账上有 "+money+" 万，休息一会再存");
					try{
						Thread.sleep(1000);
					}
					catch (InterruptedException e) {}
				}
			}
			else if(Thread.currentThread().getName().equals("出纳")){
				for (int i=1;i<3;i++){
					money = money - amount/3;
					System.out.println(Thread.currentThread().getName()+" 取出 "+amount/3+" ,账上有 "+money+" 万，休息一会再取");
					try{
						Thread.sleep(1000);
					}
					catch (InterruptedException e) {}
				}
			}
		}
	}
	
	运行结果：
	会计 存入 100 ,账上有 300 万，休息一会再存
	会计 存入 100 ,账上有 400 万，休息一会再存
	会计 存入 100 ,账上有 500 万，休息一会再存
	出纳 取出 50 ,账上有 450 万，休息一会再取
	出纳 取出 50 ,账上有 400 万，休息一会再取
	出纳 取出 50 ,账上有 350 万，休息一会再取

	
这里也是共用一个资源，但是有一个问题就是，这里的是要会计一直工作完，然后出纳才可以做，就是无论里面有多少钱，只要会计还没有将钱存完，出纳就无法拿钱，哪怕里面的钱足够，这样子就起不到两个线程交替运行了，所以这里还要修改一下，使会计和出纳交替运行，只要里面够钱，那出纳就可以用，然后拿了之后，会计继续存钱（就是在会计休息的时候，出纳可以拿钱，出纳休息的时候，会计也可以存钱，然后对那个money也有一个锁，不会出现资源混乱。）
  
对Bank Class进行修改

	public synchronized void saveOtTake(int amount) {
			// TODO 自动生成的方法存根
			if(Thread.currentThread().getName().equals("会计")){
				for (int i=1;i<=3;i++){
					money = money + amount/3;
					System.out.println(Thread.currentThread().getName()+" 存入 "+amount/3+" ,账上有 "+money+" 万，休息一会再存");
					notify();
					try{
						Thread.sleep(1000);
						wait();
					}
					catch (InterruptedException e) {}
				}
			}
			else if(Thread.currentThread().getName().equals("出纳")){
				for (int i=1;i<=3;i++){
					money = money - amount/3;
					System.out.println(Thread.currentThread().getName()+" 取出 "+amount/3+" ,账上有 "+money+" 万，休息一会再取");
					notify();
					try{
						Thread.sleep(1000);
						wait();
					}
					catch (InterruptedException e) {}
				}
			}
			System.exit(0);
		}
	运行结果：
	会计 存入 100 ,账上有 300 万，休息一会再存
	出纳 取出 50 ,账上有 250 万，休息一会再取
	会计 存入 100 ,账上有 350 万，休息一会再存
	出纳 取出 50 ,账上有 300 万，休息一会再取
	会计 存入 100 ,账上有 400 万，休息一会再存
	出纳 取出 50 ,账上有 350 万，休息一会再取  



