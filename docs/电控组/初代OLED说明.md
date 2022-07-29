---
sidebar_position: 3
---
 
作者： @赖日海  

本OLED程序主要分**数据初始化**，**数据实时更新**，**屏幕显示**三大模块，其中以数据更新和屏幕显示为主。
数据更新和屏幕显示放于task函数while循环中做到不断更新。  

本OLED程序需要创建一个.c和.h文件使得OLED显示作为单独的任务。在.c文件中需包含<**bsp_oled.h**><**adc.h**>"**detect_task.h**"，在.h文件中需包含"**sys.h**" "**stm32f4xx_conf.h**" 头文件。  
因为官方提供的字库和函数有些缺陷，我对<**bsp_oled.c**>和<**oledfont.h**>文件作了一些改动：
在<**bsp_oled.c**>中新增汉字显示函数、部分清除函数，并对显示字符函数和显示字符串函数作出较大改动，
使能够增删多种尺寸字体，具体更改代码操作在相应代码的注释中有相关解释，使字符串能中英文混显；
在<**oledfont.h**>中新增16和12字体规格的汉字库和字符库，由于取模方式和原本库不同，并且字符显示
是按照下面的模式显示的，可把原有的字符库删掉。
如需增加汉字或字符，只需在<u>***PCtoLCD2002完美版***</u>选择字符模式，在字符选项中选择**阴码**，**C51格式**，**逆向**，将取得的字模复制到库中并和字库中格式一致即可。

```c
typedef struct
{
	char name[3];
	char dat[32];
}chinese;

chinese Hzk[]={
"菜",{0x04,0x04,0x44,0xC4,0x4F,0x44,0x44,0xC4,0x24,0x24,0x2F,0xB4,0x24,0x04,0x04,0x00,0x40,0x44,0x24,0x24,0x15,0x0C,0x04,0xFE,0x04,0x0C,0x15,0x24,0x24,0x44,0x40,0x00},/*"菜",0*/
...
}


```

本OLED菜单是通过将每个界面层和每个选项都配上一个号码，每一级菜单对应一个结构体，
结构体存储的是菜单号码和菜单选项的号码。


```c
typedef struct
{
  uint16_t third_menu[4];
}Third_menu;

```

如二级菜单是100，选项则从100开始加1。而更新菜单或者上下滑动选项实际上就是选择不同的号码。  

如给一个变量enum,初始为0即一级菜单，按右则进入下一级菜单，enum+=100。
然后根据enum用switch语句选择对应的菜单界面并显示出来

对于Init()函数，除了对一些要用到的硬件设备作初始化，即ADC和按键，还需给每个菜单选项赋予一个标志数



```c
oled_menu.fisrt_menu=FIRST_MENU_NUM;    
    	for(int i=0;i<SECOND_MENU_OPTIONS;i++)
    	{
    		oled_menu.second_menu.second_menu[i]=SECOND_MENU_OPTIONS+i;
    		for(int j=0;j<THIRD_MENU_OPTIONS;j++)
    		{
    			oled_menu.second_menu.second_to_third[i].third_menu[j]=THIRD_MENU_NUM+TIME_TEN*i+j;
    		}
    	}
}


```

对于数据更新函数，其中的大致流程是先每次循环判断一下按键的值有没有发生改变，若没有，则不变；
若改变了，返回的值和原来的值不同，则先判断是按下哪一个键，每个按键按下都有一个ADC数值通过
get_real_adc_button()返回，因为这个数值在一定区间内波动，我们可取该范围的中值并将判定按下改按键
的范围限定在（data-BOTTON_DATA_LIMIT，data+BOTTON_DATA_LIMIT），判断为真则存下新值，
再根据新值作出对应的操作。如按向上按键，此时按键的值不等于原来的值，更新adc_keep_data，此时
按键的值key_num=botton_up_num，进入switch选择语句，当此时选项不是第一个则选项继续上移。
 
```c
if((get_real_adc_button()<(adc_keep_data-BOTTON_DATA_LIMIT))||(get_real_adc_button()>(adc_keep_data+BOTTON_DATA_LIMIT)))  //如果按下按键
	{
		adc_keep_data=get_real_adc_button();      //更新按键数值
		if(adc_keep_data>=(BOTTON_UP-BOTTON_DATA_LIMIT)&&adc_keep_data<=(BOTTON_UP+BOTTON_DATA_LIMIT))
		{
			options_clear=upclear; 
			key_num=botton_up_num;                  //如果按下相应的按键，给key_num一个对应按键的数值
		}


```


```c
switch(key_num)                    //每次按下按键，都有对应的操作，上下是选项翻动，左右是退出或进入下一菜单，中间的回到初界面
{
	case botton_up_num:              //当按下up键，若不是指向第一个选项，则可指向上一选项
		if(oled_menu_num%TIME_TEN!=0)  
		{
			oled_menu_num--;
		}
		break;
...
}


```

屏幕显示函数的大致过程是先判断是否需要全屏清除，当需要进出另一级菜单则全屏清除


```c
if(clear_flag)                                           //在按下左或者右后清除一次屏幕
	{
		clear_flag=STATUS_FALSE;                 //恢复标志位
		oled_clear(Pen_Clear);
	}


```

如果不会改变菜单，而仅仅是移动选项，则无需全屏清除，只部分清除改动的选项，防止屏闪
当选项上移，原来的地方要变回黑色，因为被选中区域是白色背景。


```c
if(options_clear==upclear)           //如果按下up键,即选项上移，则将原来选项恢复原来颜色
 {
	options_clear=noclear;       //恢复标志位
	 oled_partclear(0,16*(oled_menu_num%TIME_TEN+1),127,16*(oled_menu_num%TIME_TEN+2)-1,Pen_Clear);
 }
 if(options_clear==downclear)
 {
	 options_clear=noclear;
	oled_partclear(0,16*(oled_menu_num%TIME_TEN-1),127,16*(oled_menu_num%TIME_TEN)-1,Pen_Clear);
 }


```

具体的显示是通过先将菜单内容事先放于一个数组中存起来，再通过for循环逐个显示出来。  
使用数组一是方便之后的统一修改，二是方便使用for循环，而用for循环又可减少代码量。



```c
for(u8 i=0;i<SECOND_MENU_OPTIONS;i++)

```


```c
if(oled_menu_num%TIME_TEN==i)	                              //指向该选项，该选项反色显示，即白屏黑字
{
    	oled_partclear(0,16*i,127,16*(i+1)-1,Pen_Write);
    	OLED_ShowText(0,16*i,16,text[0][i].str,Pen_Clear);
}
else
{
    	OLED_ShowText(0,16*i,16,text[0][i].str,Pen_Write);    //否则正常显示
}

```

此程序是初代版本，还有很多地方需要去测试和优化。可以再增加一个部分刷新函数减小刷新量；可以
使用指针函数去作为一个菜单页的显示；可以通过以一个字节为单位一次性去控制屏幕的多个点取代原来版本
的显示字符函数一次只控制一个点，进一步提高帧率......这些都有待我们去完善!

