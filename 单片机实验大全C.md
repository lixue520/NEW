# 单片机基础实验大全



##### 1.单片机流水灯设计

![img](file:///C:\Users\qin\AppData\Local\Temp\ksohtml\wps87A0.tmp.jpg)

- 逻辑电平：图中单片机P1口作为输出口，接8个LED，8个LED按共阳接法，即端口逻辑0时可以点亮LED；而P3口我们接入两个SW-SPST开关，因为P1-P3口均含上拉电阻，因此默认情况下输出端口为逻辑1，由于P3口的两个开关接地因此当关闭SW-SPST开关时，逻辑0。

- 使用开关不同的组合方式实现灯的闪烁流向；即非中断型操作，仅仅检测开关组合方式从而改变灯的流向。

- 实验流程图：

- 实验程序：

  ```c++
  #include "reg52.h"
  #include<intrins.h>
  
  typedef unsigned char u8;	//重定义关键字
  typedef unsigned int u16;
  
  #define A P1		//用宏定义A为P1口
  sbit s1 = P3^3;
  sbit s2 = P3^2;
  u8 k=0;
  u8 p=0;
  
  void delay(u16 i){
      while(i--);
  }
  u8 i=0;
  
  void key1(){
  	//针对按键1
  	for(i=0;i<8;i++){
  		A= _crol_(A,1);			//左移函数
  		delay(50000);
  	}
  	k=A;
  }
  
  void key2(){
  	//针对按键2
  	 for(i=0;i<8;i++){
  		A= _cror_(A,1);			//右移函数
  		delay(50000);
  	}
  }
  
  
  void main(){
  
  	A=0xFE;	   //11111110
  	
      while(1){
  		if(s1==0&&s2==1){
  			//s1开关是打开的
  			 A=0xFE;
  			 key1();
  		}else if(s2==0&&s1==1){
  			A=k;
  			key2();
  		}else if(s2==0&&s1==0){
  			  A=A&0;
  		} else{
  			 A=A|0x11;			//You know? Life is fucking movie.My name is ChenHaoNan.
  		}      	 
      }
  }
  ```

##### 2.秒计时器设计

![image-20220325142322975](../AppData/Roaming/Typora/typora-user-images/image-20220325142322975.png)

- ![image-20220325142400324](../AppData/Roaming/Typora/typora-user-images/image-20220325142400324.png)

- 具体要求：

  - 要求一：编写T0做定时器产生周期为1 秒的方波，从P3.6，P3.7口进行输出，将P3.7接到示波器显示该方波；用T1作计数器对从P3.6输出的方波进行计数，计数结果通过发光二极管显示。

    ```c++
    //周期为1s的方波，则需要计时500ms再取反
    //使用定时器TO，方式1：65536-(50ms/1us)=15536(3CB0H)
    //定时器初值: TH0=0X3C,TL0=0XB0
    //关于计数器T1:将从T0输出的方波信号尽行计数，当一个方波出现时计数1，灯亮一下
    //方式1:65535(FFFFH);一个脉冲经过计数器中断
    
    void TimerOInit(){
      //T1作计数器，T0作定时器
      TMOD|=0X51;
      TH0 = 0X3C;
      TL0 = 0XB0;
      TH1 = 0XFF;
      TL1 = 0XFF;
      ET0 = 1;
      ET1 = 1;
      EA = 1;
      TR0 = 1;
      TR1 = 1;
    }
    ```

  - 要求二：设计一个60秒计数器；

    ```c++
    #include "reg52.h"
    typedef unsigned char u8;
    typedef unsigned int u16;
    sbit led = P3^6;	 //定时器T0输出口
    #define LEDs P1
    #define PL P0
    #define PH P2
    sbit show = P3^7;
    
    void Timer0Init(){
      //T1作计数器，T0作定时器
      TMOD|=0X51;
      TH0 = 0X3C;
      TL0 = 0XB0;
      TH1 = 0XFF;
      TL1 = 0XFF;
      ET0 = 1;
      ET1 = 1;
      EA = 1;
      TR0 = 1;
      TR1 = 1;
    }
    
    void main(){
      Timer0Init();//初始化
      while(1);
    }
    
    void Time0() interrupt 1{
       static u16 i;
       TH0 = 0X3C;
       TL0 = 0XB0;	  //50ms计数器
       i++;
      if(i==10){   //计数500ms后执行灯操作;
        i=0;
        led=~led;
    	show = led;
      }
    }
    
    static u8 k=0;
    static u16 pk=0;
    static u16 pm=0;
    static u8 set=0x00;
    u8 code smgduan[17]={0x3f,0x06,0x5b,0x4f,0x66,0x6d,0x7d,0x07,
    					0x7f,0x6f,0x77,0x7c,0x39,0x5e,0x79,0x71};//显示0~F的值
    						  //9
    void Time1() interrupt 3{
    	
    		TH1 = 0XFF;
        TL1 = 0XFF;
    	P1=set;
    	if(k<=9){
    		PH=smgduan[0];	  //高位位0
    		PL=smgduan[k];
    		k++;	
    	}else if(k%10==0){
    		PL=smgduan[0];	 //进位低位0
    		pk=k/10;		//高位取整
    		PH=smgduan[pk];
    		k++;
    	}else if(k>10&&k<=60&&(k%10!=0)){
    		pk=k/10;
    		pm=k%10;
    		PH=smgduan[pk];
    		PL=smgduan[pm];
    		k++;
    	} else{
    		k=0;
    	}
    	set++;
    }
    ```

    

  - 要求三：设置按键控制计时器的启动、停止及清零功能。

    ```c++
    //待定中。。。。
    //接入三个按键
    //key1:定时器启动与停止  key2:定时器清零
    //具体实现方法：接外部中断1和2，按下降沿触发，触发后启动中断程序1,再按下启动定时器中断程序；按键二是清零工作，即将当前计数器清理0
    sbit key1 P3^2;
    sbit key2 P3^3;
    
    EX0=1;
    EX1=1;
    IT0=1;//下降沿有效--->中断号：0
    IT1=1;//下降沿有效--->中断号：2
    
    void int0() interrupt 0{
      //将当前的程序锁死
      while(1){
        delay(60000);
        break;
      };
      
    }
    
    void int1() interrupt 2{
      //将定时器的计数器清零
      TL1 = 0XFF;
      delay(10000);
    }
    
    
    ```

- 综合设计：

  ```c++
  #include "reg52.h"
  typedef unsigned char u8;
  typedef unsigned int u16;
  sbit led = P3^6;	 //定时器T0输出口
  #define LEDs P1
  #define PL P0
  #define PH P2
  sbit show = P3^7;
  sbit key1=P3^3;
  sbit key2=P3^2;
  
  void Timer0Init(){
    //T1作计数器，T0作定时器
    TMOD|=0X51;
    TH0 = 0X3C;
    TL0 = 0XB0;
    TH1 = 0XFF;
    TL1 = 0XFF;
    ET0 = 1;
    ET1 = 1;
    EA = 1;
    TR0 = 1;
    TR1 = 1;
  
    //有关外中断初始化
  	EX0=1;
  	EX1=1;
  	IT0=1;//下降沿有效--->中断号：0
  	IT1=1;//下降沿有效--->中断号：2
  }
  
  //延时模块
  void delay(u16 i){
    while(i--);
  }
  
  void main(){
    Timer0Init();//初始化
    while(1);
  }
  
  void Time0() interrupt 1{
     static u16 i;
     TH0 = 0X3C;
     TL0 = 0XB0;	  //50ms计数器
     i++;
    if(i==10){   //计数500ms后执行灯操作;
      i=0;
      led=~led;
  	show = led;
    }
  }
  
  static u8 k=0;
  static u16 pk=0;
  static u16 pm=0;
  static u8 set=0x00;
  u8 code smgduan[17]={0x3f,0x06,0x5b,0x4f,0x66,0x6d,0x7d,0x07,
  					0x7f,0x6f,0x77,0x7c,0x39,0x5e,0x79,0x71};//显示0~F的值
  						  //9
  void Time1() interrupt 3{
  	
  	TH1 = 0XFF;
      TL1 = 0XFF;
  	P1=set;
  	if(k<=9){
  		PH=smgduan[0];	  //高位位0
  		PL=smgduan[k];
  		k++;	
  	}else if(k%10==0){
  		PL=smgduan[0];	 //进位低位0
  		pk=k/10;		//高位取整
  		PH=smgduan[pk];
  		k++;
  	}else if(k>10&&k<=60&&(k%10!=0)){
  		pk=k/10;
  		pm=k%10;
  		PH=smgduan[pk];
  		PL=smgduan[pm];
  		k++;
  	} else{
  		k=0;
  	}
  	set++;
  }
  
  //外部中断1
  void StartStop0() interrupt 2{
  	   //将当前的程序锁死
    while(1){
    	delay(40000);
  	break;
    }
  } 
  
  //外部中断0
  void TimerClear() interrupt 0{
  	 //将定时器的计数器清零
    k=0;       //将中断程序2的计数器清零
    delay(10000);
  }
  
  
  
  
  
  ```

  

##### 3.交通灯设计

1. 实验内容：

   ​    模拟交通信号灯控制：一般情况下正常显示，东西-南北交替放行，各方向通行时间为30秒。有救护车或警车到达时，两个方向交通信号灯全为红色，以便让急救车或警车通过，设通行时间为10秒，之后交通恢复正常。用单次脉冲模拟急救车或警车申请外部中断。 

   ![](D:/1AStudy/单片机基础/单片机/例程2-LED闪烁/20220331121925.png)

2. ![image-20220329000108849](../../AppData/Roaming/Typora/typora-user-images/image-20220329000108849.png)

3. 程序思路：

   - 定时器设计
   
     - 定时器0：计时作用，30秒计时当前状态为南北//或者仅为它计时器

       每次中断将中断特殊标记k位0，并由检查k状态的函数将交通灯点亮。
   
     - 定时器1：计时作用，30秒计时当前状态为东西。
   
       - 南北：
   
         ```c++
         P1:都行了，怎么亮自己编
         ```
   
         
   
       - 东西：
   
         ```c++
         P1：都行了，怎么亮自己编
         ```
   
         
   
   - 外中断设计
   
     - 外部中断启动时，东西、南北亮红灯，并延时10s，返回0
   
   - ```c++
     #include "reg52.h"
     typedef unsigned char u8;
     typedef unsigned int u16;
     u16 time = 0;
     u16 time1 = 0;
     u8 SX = 0;//交通方向辨别器
     //u8 ZY = 1;
     #define PL P0
     #define PH P2
     #define JD P1
     
     sbit INT = P3^2;  //INT1
     
     void initstatus(){
     	//定义定时器0的状态
     	TMOD|=0X01;
     	JD=0;
     	EA=1;
     
     	TH0=0XFC;
         TL0=0X18;
     	ET0 = 1;
     	TR0 = 1;
     	PX0 = 1;
     
     	//电平中断
     	INT = 0;
     	EX0=1;//打开外部中断0
         IT0=0;//设置外部中断的触发方式
     	PT0=0;
     
     
     }
     
     //显示交通灯
     void showJT(u8 SX){
     	//前30s标记为南北
     	//后30秒标记为东西
     	//判断标记为每30秒变化的布尔式标记 flag;
     	if(SX==0){
     		//南北绿色、东西红色
     		JD=0;
     		JD|=0xD2;	
     	}
     	else if(SX==1){
     	   //南北黄色、东西红色
     	   JD=0;
     	   JD|=0xE4;
     	}
     	else if(SX==2){
     		//东西绿色、南北红色
     		JD=0;
     		JD|=0xE1;	
     	}
     	else if(SX ==3)
     	{	 //东西黄色、南北红色
     		JD=0;
     		JD=0xC9;
     		
     	}else
     	{
     		SX=0;
     		JD=0;		
     	}	
     }
     
     //显示秒数
     u8 code smgduan[17]={0x3f,0x06,0x5b,0x4f,0x66,0x6d,0x7d,0x07,
     					0x7f,0x6f,0x77,0x7c,0x39,0x5e,0x79,0x71};//显示0~F的值
     u16 pk=0;//取高位
     u16 pm=0;//取低位
     void show(u16 k){
     	//将30秒计数的标记穿入
     	//换算标记输出到LED
     	k = time;
     	if(k<30){
     		//显示交通灯
     		showJT(SX);
     		if(k<=9){
     			PH=smgduan[0];
     			PL=smgduan[k];
     		}else if(k%10==0){
     			PL=smgduan[0];
     		    pk=k/10;
     			PH=smgduan[pk];
     			k++;
     		}else if(k>10&&k<=30&&(k%10!=0)){
     			pk = k/10;
     			pm = k%10;
     			PH = smgduan[pk];
     			PL = smgduan[pm];
     			k++;
     		}			
     	}else{
     		time=0; //将计数清零
     		SX++;	//30秒改变状态1-4
     	}
     
     
     }
     
     
     
      
      void main(){
      	//定义中断初始化
     	initstatus();
     	while(1);
      }
     
     
      //中断程序
     void TimeA1() interrupt 1{
        static u16 i;
        TH0=0XFC;
        TL0=0X18; //1ms计数
        i++;
       if(i==1000){   //计数1s后执行灯操作;
         i=0;
          show(time);
     	 time++;
       }
     }
     
     void delay(u16 i){
     	while(i--);
     
     }
     void TM() interrupt 0{
     	//启动外部中断0
     	JD=0X11;
     	//delay(80000);
     }
     
     ```



- 算法优化：双定时器设计：由于定时器的同步使用，因此将本次中断系统设计为三个中断操作：

  - 定时器0：使用它进行30s精确定时，并设计四个方向的交通灯
  - 定时器1：使用它进行10s精确定时，当没触发外援中断时，不工作，触发时10s后开启TR0,关闭TR1，EX0位1
  - 外部中断0：仅仅作为定时器优先级控制、初始化定时器1控制、每次触发完，将外部0标志位EX0位0
  - PS独立显示函数：Show、ShowA;由于我是初学者设计的函数的整洁度不高---后期会逐渐优化。
  
  ```C++
  #include "reg52.h"
  typedef unsigned char u8;
  typedef unsigned int u16;
  u16 time = 0;
  u16 time1 = 0;
  u16 n=0;
  u8 SX = 0;//交通方向辨别器
  //u8 ZY = 1;
  #define PL P0
  #define PH P2
  #define JD P1
  
  sbit INT = P3^2;  //INT1
  
  void initstatus(){
  	//定义定时器0的状态
  	TMOD|=0X11;
  	JD=0;
  	EA=1;
  
  	TH0=0XFC;
      TL0=0X18;
  	ET0 = 1;
  	TR0 = 1;
  	PX0 = 0;
  
  	//电平中断
  	EX0=1;//打开外部中断0
      IT0=0;//设置外部中断的触发方式
  	PT0=0;
  
  	TH1=0XFC;
     TL1=0X18; //1ms计数
     ET1=1;
     TR1=0;  //启动定时器1
     PT1=1;
  
  
  }
  
  //显示交通灯
  void showJT(u8 SX){
  	//前30s标记为南北
  	//后30秒标记为东西
  	//判断标记为每30秒变化的布尔式标记 flag;
  	if(SX==0){
  		//南北绿色、东西红色
  		JD=0;
  		JD|=0xD2;	
  	}
  	else if(SX==1){
  	   //南北黄色、东西红色
  	   JD=0;
  	   JD|=0xE4;
  	}
  	else if(SX==2){
  		//东西绿色、南北红色
  		JD=0;
  		JD|=0xE1;	
  	}
  	else if(SX ==3)
  	{	 //东西黄色、南北红色
  		JD=0;
  		JD=0xC9;
  		
  	}else
  	{
  		SX=0;
  		JD=0;		
  	}	
  }
  
  //显示秒数
  u8 code smgduan[17]={0x3f,0x06,0x5b,0x4f,0x66,0x6d,0x7d,0x07,
  					0x7f,0x6f,0x77,0x7c,0x39,0x5e,0x79,0x71};//显示0~F的值
  u16 pk=0;//取高位
  u16 pm=0;//取低位
  void show(u16 k){
  	//将30秒计数的标记穿入
  	//换算标记输出到LED
  	k = time;
  	if(k<30){
  		//显示交通灯
  		showJT(SX);
  		if(k<=9){
  			PH=smgduan[0];
  			PL=smgduan[k];
  		}else if(k%10==0){
  			PL=smgduan[0];
  		    pk=k/10;
  			PH=smgduan[pk];
  			k++;
  		}else if(k>10&&k<=30&&(k%10!=0)){
  			pk = k/10;
  			pm = k%10;
  			PH = smgduan[pk];
  			PL = smgduan[pm];
  			k++;
  		}			
  	}else{
  		time=0; //将计数清零
  		SX++;	//30秒改变状态1-4
  	}
  
  
  }
  
  
  void showA(u16 k){
  	JD=0X11;
  	k=n;
  	if(k<10){
  		PH=smgduan[0];
  		PL=smgduan[k];	
  	}else{
  		n=0;
  		TR1=0;
  		TR0=1;
  	}
  }
  
  
   
   void main(){
   	//定义中断初始化
  	initstatus();
  	while(1);
   }
  
  
   //中断程序
  void TimeA1() interrupt 1{
     static u16 i;
     TH0=0XFC;
     TL0=0X18; //1ms计数
     i++;
    if(i==1000){   //计数1s后执行灯操作;
      i=0;
       show(time);
  	 time++;
    }
  }
  
  void delay(u16 i){
  	while(i--);
  
  }
  
  u16 m=0,PK1,PM1;
  void TM() interrupt 0{
  	PT0=0;
  	PT1=1;
  	TR1=1;
  	TR0=0;
  }
  
  //定义定时中断1
  void MM() interrupt 3{
     
     TH1=0XFC;
     TL1=0X18; //1ms计数
     m++;
    if(m==1000){   //计数1s后执行灯操作;
      m=0;
       showA(n);
  	 n++;
    }
  }
  
  
  
  ```
  
  

##### 4.简易电压表

##### 5.串行通信设计

##### 6.综合设计：万年历