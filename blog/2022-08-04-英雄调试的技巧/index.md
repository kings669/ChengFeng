---
slug: 英雄调试的技巧
title: 英雄调试的技巧
author: Chen Jie
authorTitle: Head of Hero Robot Control class of '22
authorURL: https://jxlgdx.coding.net/u/FHwtODUjRJ
authorImageURL: https://qlogo4.store.qq.com/qzone/2985794207/2985794207/100?1655782637
tags: [英雄, 麦轮, 电控]
---

## 前言

这里的内容主要是描写我从大二开学开始调试英雄机器人的内容，其实学弟们不用认为这是一个很难的东西，你们每天按部就班的一步步来就行，你们首先要做的前提是将英雄机器人的每一个电机的基础功能先给它完成 就像用遥控器控制3508,6020电机，一开始只要能用遥控器完成基础的控制，先不要着急的将英雄机器人的每个部分给联系起来，你们先完成每个部分单独的任务就行，当时一开始我们是因为英雄机器人还没做出来，所以是跟着机械那边的进度来写的，就是机械做完了那个部分，他们想要调，你就完成哪个部分就行，不过今天前期你们是有整车可以用的，所以进度你可以快点。最好在暑假能完成英雄机器人每个部分单独的代码，然后回校之后可以直接进行整体的统合，将每个部分给联系起来。

## 框架
这一部分其实很简单，就是将你们自己写的代码按照我们的格式进行改编，改写，将他们进行一个规范化，这是你们完成了电机控制之后需要做的，例如射击部分

```c
void shoot_task(void *pvParameters )
{

	   vTaskDelay(SHOOT_TASK_INIT_TIME);
	   Shoot_Init(&Shoot_Struct); //这是代码的初始化部分，
       就是将一些初始要进行赋值的部分放在这里，例如PID的三个初始值。

	   while(1)
	   {				
		Shoot_Set_Mode(&Shoot_Struct);//这里是模式的获取部分，一般是获取遥控器的值，并且根据获得的值来进行模式的赋予

		Shoot_Mode_Change_Control_Transit(&Shoot_Struct);//这里是模式的切换部分

        Shoot_Feedback_Update(&Shoot_Struct);//这里主要是进行数值的更新，就将一些你需要一直重复循环给值的代码放在这，例如遥控器的获取

		Shoot_Set_Contorl(&Shoot_Struct);//这里面主要的是进行模式的具体内容的编写，一般是在这里面根据你的模式来对设定值进行一个设定

		shoot_motor_control(&Shoot_Struct);//这里则是一个PID的运算，
            根据你的设定值来获取给电机的值

        if(Shoot_Struct.plate_motor.tem <70)//这是一个电机的温度保护
		{					
           set_shoot_motor_current(plate_can_set_current,0,0);//最后就是将值传给电机	
		}
		else
        {
		   set_shoot_motor_current(0,0,0);
		}		 
            vTaskDelay(SHOOT_CONTROL_TIME_MS);//头尾这两个延时函数很重要，是用来进行任务切换的，没有可能导致任务运行不下去
	   }
}   
```
## 注意事项
上面的步骤完成之后，你们应该完成一个基础的框架，只不过里面可能只完成了单独电机的控制，而没有将他们组合成一个部分，后面我将描述英雄机器人的各个部分的代码编写写和注意事项（基础部分）

### Chassis
一开始的底盘代码你们不用写的很复杂，那是你们后面进行优化的，首先你们要做的就是让他们的四个轮子动起来，然后用遥控器控制他们，这是很简单的部分，你们不用想的太复杂。主要就是获取电机需要运动的设定值而已，你们有兴趣的可以去了解了解他们的原理。不用的话那就直接用就行。
```c
void chassis_rc_to_control_vector(fp32 *vx_set, fp32 *vy_set,fp32 *wz_set, 
chassis_move_t *chassis_move_rc_to_vector)
{
    
    if (chassis_move_rc_to_vector == NULL || vx_set == NULL || vy_set == NULL)
    {
        return;
    }
    int16_t vx_channel=0, vy_channel=0,wz_channel=0;
    fp32 vx_set_channel=0.0f, vy_set_channel=0.0f,wz_set_channel=0.0f;	
	set_chassis_speed_max(super_cap_chassis_speed_set());
    rc_deadband_limit(chassis_move_rc_to_vector->remote_data->rc.ch[2], vx_channel, CHASSIS_RC_DEADLINE);
    rc_deadband_limit(chassis_move_rc_to_vector->remote_data->rc.ch[3], vy_channel, CHASSIS_RC_DEADLINE);
	rc_deadband_limit(chassis_move_rc_to_vector->remote_data->rc.ch[4], wz_channel, CHASSIS_RC_DEADLINE);
		
    vx_set_channel = vx_channel * CHASSIS_VX_RC_SEN;
    vy_set_channel = vy_channel * CHASSIS_VY_RC_SEN;
	wz_set_channel = wz_channel * -CHASSIS_WZ_RC_SEN;

    *vx_set = vx_set_channel;
    *vy_set = vy_set_channel;
	*wz_set = wz_set_channel;
}
```   
```c

static void Chassis_Wheeel_Speed(const fp32 vx_set,const fp32 vy_set,const fp32 wz_set,fp32 wheel_speed[4])
{

	wheel_speed[CHASSIC_LEFTUP_ID] = vx_set + vy_set - (0 - 1.0f) * MOTOR_DISTANCE_TO_CENTER * wz_set;
    wheel_speed[CHASSIC_RIGHTUP_ID] = +vx_set - vy_set - (0 - 1.0f) * MOTOR_DISTANCE_TO_CENTER * wz_set;
    wheel_speed[CHASSIC_LEFTDOWN_ID] = -vx_set + vy_set - (-0 - 1.0f) * MOTOR_DISTANCE_TO_CENTER * wz_set;
    wheel_speed[CHASSIC_RIGHTDOWN_ID] = -vx_set - vy_set - (-0 - 1.0f) * MOTOR_DISTANCE_TO_CENTER * wz_set;
}
```

这两个基本函数就能完成他们的基本运动 wheel_speed就是你们需要获取的设定值，将他们放入里面就行。后面的优化 键盘控制你们可以等完成各个部分的代码之后，能进行完整运动和操作之后再看。

## Gimbal

云台这部分主要是Pitch轴和Yaw轴，这里我更多的就是和你们说一说一些注意事项，遇到的一些问题和解决方案，这一部分要完成初始的运动还是不难的，只要能用角度环和速度环对电机进行控制就行。云台的主要难点在于它的稳定性，就是PID的调节问题，怎么让他们不抖·不晃，归位很快速。这里主要有两个技术点，一个卡尔曼滤波和前馈控制，你们先单独用PID进行调节，调到你可以调到的最佳状况，看看他们的效果，然后在对Yaw轴 Pitch轴进行卡尔曼滤波，对Pitch轴进行前馈控制，基本就能让他们很稳定。我这不知道你们有没有完成他们角度控制。就是能用遥控器来控制他们360度慢慢转，这里的主要点是对角度的控制，这里的代码我们统一放在了CAN通信那里。

```c
static fp32 motor_ecd_to_angle_change(uint16_t ecd, uint16_t offset_ecd)
{

    int32_t relative_ecd = ecd - offset_ecd;
    if (relative_ecd > HALF_ECD_RANGE)
    {
        relative_ecd -= ECD_RANGE;
    }
    else if (relative_ecd < -HALF_ECD_RANGE)
    {
        relative_ecd += ECD_RANGE;
    }

    return relative_ecd * MOTOR_ECD_TO_RAD ;
}
```
这个主要解决的是他们的疯转，能让他们跟随遥控器，并且设定他们的中值。你们一开始不用想着小陀螺和底盘跟随云盘，你们最开始只要完成它的编码值控制就行。那些是你们进行优化的时候要搞的。还有就是他们中值的设定，这个设定的值不是固定的，而是要根据6020的位置来进行设定的，每次重装，它的值一般也要跟着变，它的调节也很简单，你就随意先设定一个值，比如就下面这个，然后在加个200左右来看他的转向，如果偏的更严重就-200就行，直到把他设定到中间就好。
```c
gimbal_control_init->gimbal_yaw_motor.offset_ecd=4060;//  6400
```
云台前期你就完成这几点就好，能进行简单的控制。

## Shoot

射击部分主要分为摩擦轮和拨盘俩大部分，这两部分是整辆车里面最重要的地方，相对而言也是更容易出现问题的地方，摩擦轮主要会导致弹速不稳定，而拨盘出现的更多的是卡单，不过相对于目前的代码来说，你们现在不用想这么多，你们初期只要用PID让他们动起来就行，相对而言现在还是比较好写的，两个摩擦轮电机用的是3508，只用单环PID就行，而拨盘PID则需要用双环PID，否则会很不稳定。而双环PID这里主要注意的是一个参数比，当时就这一个参数让双环一直调不稳定。
```c
 shoot_feedback_update->plate_motor.motor_spee=shoot_feedback_update->plate_motorshoot_plate_motor_measure->speed_rpm*Motor_RMP_TO_SPEED_3508
```

除此之外，摩擦轮电机这里还需要一个斜波函数，这个函数主要是让摩擦轮电机缓慢开始转，来减少电机的损耗。这个相对还是比较简单的，仔细看一下基本就知道怎么用了。射击部分初期不是太难写，用遥控器控制就行了，后面的一些重点我在后面注意事项那边再说明。另外还有一个就是遥控器控制电机的思路，我们现在用的是S1上拨然后拨回中间算一次开启，然后重复算关闭，这样能更加的充分利用S1的键位。

```c
static int8_t last_s = RC_SW_UP;

if((set_shoot_mode->remote_data->rc.s[1] ==RC_SW_UP)&&(RC_SW_UP!=last_s)&&Shoot_Struct.friction_control.shoot_mode==SHOOT_STOP)
{
	set_shoot_mode->friction_control.shoot_mode = SHOOT_READY_FRIC;
}
else if((set_shoot_mode->remote_data->rc.s[1]==RC_SW_UP)&&(RC_SW_UP!=last_s)&&Shoot_Struct.friction_control.shoot_mode!=SHOOT_STOP)
{
	set_shoot_mode->friction_control.shoot_mode = SHOOT_STOP;
}
	
last_s = set_shoot_mode->remote_data->rc.s[1];
```

### 组合

将前面一些基本的功能实现之后，就需要将他们组合并且联系起来，这里我主要分为两个时间段，一个是初期的遥控器阶段，这个阶段就是将英雄机器人各部分基本调到稳定了，没有很大的技术变换，能用遥控器实现它的所有功能。然后就是键盘阶段，这里主要就要开始考虑一些比赛的注意事项，包括一些超级电容的控制，UI的绘制，裁判系统数据的读取之类的，完成之后才能说有上场的能力，最后就是提升和优化整辆车的稳定性。

### 注意
目前我们英雄机器人采用的是双板控制，就是上面是C板，下面是A板。不过A板更多的是起一个辅助的作用，主要的控制来源还是C板，就是说，现在A板主要就是控制英雄机器人的下部分，就是四个轮子和一个拨盘电机，但是A板也就起一个输出电流的作用，就是在A板着做一个PID然后输出给电机，他们的设定值来源还是C板给的。这里主要的一个技术重点就是一个A C两板之间的通行。这个通信主要用的是CAN之间的通信。首先一个注意重点事项就是一定要将CAN通信的通道给分配好，因为这个通道的数量是有限的，而且到后面你会发现CAN通道可能会有堵塞的现象，一般体现在，当你写好代码之后，电机没有数据，或者数据出现错误导致电机没转，这时的解决办法你可以拔掉其他电机的CAN线或者代码给注释掉看看。

```c
void  canTx(uint8_t data[8], uint32_t id)
{

	CanTxMsg TxMessage;

	TxMessage.CAN_TxHeader.IDE = CAN_ID_STD; 
	TxMessage.CAN_TxHeader.RTR = CAN_RTR_DATA;	 
	TxMessage.CAN_TxHeader.DLC = 0x08;				

	TxMessage.CAN_TxHeader.StdId = id; 

	memcpy(TxMessage.Data, data, 8);

  xQueueSend(CAN1_Queue,&TxMessage,1);
  //vTaskDelay(110);
}
```

相比于其他的发送函数，这个也就改了一点东西，而它的好处就是你可以控制它发送任意你想要发送的数据，例如下面的使用案例
```c
void chassis_data_send(chassis_move_t *chassis_data_send)
{

		VX = (chassis_move.vx_set)*2000;
		VY = (chassis_move.vy_set)*2000;
		VZ = (chassis_move.wz_set)*1000;
		
		TxData[0] = (uint8_t)((VX)>>8);
	    TxData[1] = (uint8_t)(VX);
		TxData[2] = (uint8_t)((VY)>>8);
	    TxData[3] = (uint8_t)(VY);
		TxData[4] = (uint8_t)((VZ)>>8);
	    TxData[5] = (uint8_t)(VZ);
		TxData[6] = (uint8_t)((chassis_data_send->key_send->ctrl)>>8);
	    TxData[7] = (uint8_t)(chassis_data_send->key_send->ctrl);	


		canTx((uint8_t *)&TxData,FEED_BACK_DATA_ID);
}
```

我建议你们可以去仔细的研究一下CAN通信的具体使用，这对你们后面去分配CAN通道和使用CAN有很大的帮助，目前我的建议是不要使用双板控制，统一使用单C板控制，否者会出现上面C板上电慢和控制信号不稳定的问题，而且让后面一些联动代码的编写出现一些麻烦。

# chassis
### chassis（遥控器）
在这里需要注意的是底盘和云台的联动，这里主要有两个模式，一个是正常的遥控器控制，还有一个是小陀螺模式，关于小陀螺的一些原理和底盘四个电机的原理你们可以去看看赖日海学长的全向轮控制那里，在这里我就不过多的讲述了。这边的模式控制代码我放在了chassis_mode.c当中，主要是下面的两个函数
```c
void chassis_follow_chassis_control(fp32 *vx_set, fp32 *vy_set, fp32 *wz_set, chassis_move_t *chassis_move_rc_to_vector)
{

    if (vx_set == NULL || vy_set == NULL || wz_set == NULL || chassis_move_rc_to_vector == NULL)
    {
        return;
    }
      if(!chassis_move_rc_to_vector->remote_data->IF_KeyBoard)
		{
			chassis_rc_to_control_vector(vx_set, vy_set,wz_set ,chassis_move_rc_to_vector);
		}else{
			chassis_keyboard_to_control_vector(vx_set, vy_set,wz_set ,chassis_move_rc_to_vector);
		}  
}
```
```c
static fp32 chassis_calc_wz_speed(fp32 min,fp32 max,uint32_t now)
{

	fp32 wz_speed;
	wz_speed = fabsf((0.2f * sinf(ROTOR_OMEGA * (float)now)) + min);
	return fp32_constrain(wz_speed,min,max);
}
```

### chassis（键盘）

键盘这里的联动也是在云台那，不过多了一个超级电容和键盘控制。和云台的话主要是一个底盘跟随云台的模式，这个模式是很关键的，因为我们的操作视角的第一人称的，看不见底盘的运动情况，为了让操作方便就写这个模式让我们头指向的方向为底盘的前进方向。这里关于底盘的代码也是放在chassis_莫得这里面。
```c
void chassis_follow_gimbal(fp32 *vx_set, fp32 *vy_set, fp32 *wz_set, chassis_move_t *chassis_move_rc_to_vector)
{

	if (vx_set == NULL || vy_set == NULL || wz_set == NULL || chassis_move_rc_to_vector == NULL)
    {
        return;
    }
		fp32 vx=0.0f ,vy=0.0f ,wz=0.0f;
		fp32 sin_yaw = 0.0f, cos_yaw = 0.0f;
		
		if(!chassis_move_rc_to_vector->remote_data->IF_KeyBoard)
		{
		   chassis_rc_to_control_vector(&vx, &vy, &wz ,chassis_move_rc_to_vector);
		}
            else{
		   chassis_keyboard_to_control_vector(&vx, &vy, &wz ,chassis_move_rc_to_vector);
		} 
            sin_yaw = arm_sin_f32(-chassis_move_rc_to_vector->relative_angle);
            cos_yaw = arm_cos_f32(-chassis_move_rc_to_vector->relative_angle);
		if ((chassis_move_rc_to_vector->chassis_mode == CHASSIS_FOLLOW_GIMBAL) &&(chassis_move_rc_to_vector->flag == 1))
		{
	        wz = pid_calculation(&chassis_move_rc_to_vector->chassis_angle_pid,chassis_move_rc_to_vector->relative_angle,0.0f)*0.8f;chassis_move_rc_to_vector->flag = 0;			
		}
		if ((chassis_move_rc_to_vector->chassis_mode == CHASSIS_FOLLOW_GIMBAL) &&(chassis_move_rc_to_vector->flag == 0))
		{
		    wz = pid_calculation(&chassis_move_rc_to_vector->chassis_angle_pid,chassis_move_rc_to_vector->relative_angle,0.0f)*0.3f;
            if(((abs((int)((chassis_move_rc_to_vector->relative_angle-0)*10000)))/10000)<0.0001f)
		{
			 chassis_move_rc_to_vector->flag = 1;
		}					
		}
		
    *vx_set = cos_yaw * vx - sin_yaw * vy;
    *vy_set = sin_yaw * vx + cos_yaw * vy;
    *wz_set=-wz;
}
```

这里出现的主要问题就是头疯转的问题，也就是wz这里的PID的问题，这里比较容易出现一个超调的现象，是需要你们额外注意的，也是你们后面需要改善的地方。除了PID导致出现的一个超调疯转，还有一个就是因为回转速度太快，因为惯性而出现的超调问题，所以我这里采用了一个回调的距离小到一定距离之后减慢它的一个回调速度来进行解决。除此之外，这里可能还会出现一个BUG，不过时间有点久我忘了啥问题，但是我是通过更改模式切换的逻辑来解决的，就是用flag，以后你们出现未知的问题时你们可以试试。

```c
void set_chassis_followgimbal(void)
{	

	chassis_move.chassis_groy = 0;
	chassis_move.chassis_mode=CHASSIS_FOLLOW_GIMBAL;
	chassis_move.chassis_gimbal = 1;
}
void set_chassis_followgryo(void)
{

	chassis_move.chassis_gimbal = 0; 
	chassis_move.chassis_mode=CHASSIS_FOLLOW_GRYO;
	chassis_move.chassis_groy = 1;	
	chassis_move.flag = 0;

}
```

然后就是和键盘控制的联动，这部分主要是后期你们需要更改的，前期并不需要关心太多，原理我就不讲了，你们可以去看看了解了解，我把相关的函数给你们列出来。
```c
void chassis_keyboard_to_control_vector(fp32* vx_set,fp32* vy_set,fp32* wz_set,chassis_move_t* chassis_keyboard_to_vector)
{

	if (chassis_keyboard_to_vector == NULL || vx_set == NULL || vy_set == NULL)
    {
        return;
    }
	set_chassis_speed_max(chassis_keyboard_to_vector->speed_set);
	*vx_set =KEY_LR_Ctrl()*CHASSIS_VX_RC_SEN;
	*vy_set =KEY_FB_Ctrl()*CHASSIS_VY_RC_SEN;
	*wz_set = 0.0f ;
}
```
这边的运用还是比较简单的，他这里的重点是在键盘的控制那边，那里有个关于加减速的逻辑。

底盘和超级电容的联动的话，是更后面的事情了，这边主要是获取超级电容给与的速度值来对电机进行控制。
```c
void chassis_follow_gyro(fp32 *vx_set, fp32 *vy_set, fp32 *wz_set, chassis_move_t *chassis_move_rc_to_vector)
{

		if (vx_set == NULL || vy_set == NULL || wz_set == NULL || chassis_move_rc_to_vector == NULL)
    {
        return;
    }
		fp32 vx=0.0f ,vy=0.0f ,wz=0.0f;
		fp32 sin_yaw = 0.0f, cos_yaw = 0.0f;
		
		if(!chassis_move_rc_to_vector->remote_data->IF_KeyBoard)
		{
			chassis_rc_to_control_vector(&vx, &vy, &wz ,chassis_move_rc_to_vector);
		}else{
			chassis_keyboard_to_control_vector(&vx, &vy, &wz ,chassis_move_rc_to_vector);
		} 
            sin_yaw = arm_sin_f32(-chassis_move_rc_to_vector->relative_angle);
            cos_yaw = arm_cos_f32(-chassis_move_rc_to_vector->relative_angle);
		if(chassis_move_rc_to_vector->chassis_mode == CHASSIS_FOLLOW_GRYO)
		{	
			if(Data_gyo_A.data.bit.One)
	   {
			 if(Data_gyo_A.data.bit.NOMAL_FAST_MODE)
			 {
			    wz = chassis_calc_wz_speed(ONE_ROTOR_WZ_MIN-0.2f,ONE_ROTOR_WZ_MAX-0.2f,HAL_GetTick())*8.0f;
			 }
			 else if(Data_gyo_A.data.bit.NOMAL_SLOW_MODE)
			 {
			 		wz = chassis_calc_wz_speed(ONE_ROTOR_WZ_MIN,ONE_ROTOR_WZ_MAX,HAL_GetTick())*8.0f;
			 }
			 else if(Data_gyo_A.data.bit.SPEED_UP_MODE)
			 {
				 	wz = chassis_calc_wz_speed(ONE_ROTOR_WZ_MIN+0.35f,ONE_ROTOR_WZ_MAX+0.35f,HAL_GetTick())*8.0f;		 
			 }

		 }else if(Data_gyo_A.data.bit.Two)
		 {
			 if(Data_gyo_A.data.bit.NOMAL_FAST_MODE)
			 {
			    wz = chassis_calc_wz_speed(TWO_ROTOR_WZ_MIN-0.2f,TWO_ROTOR_WZ_MAX-0.2f,HAL_GetTick())*8.0f;
			 }
			 else if(Data_gyo_A.data.bit.NOMAL_SLOW_MODE)
			 {
			 		wz = chassis_calc_wz_speed(TWO_ROTOR_WZ_MIN,TWO_ROTOR_WZ_MAX,HAL_GetTick())*8.0f;
			 }
			 else if(Data_gyo_A.data.bit.SPEED_UP_MODE)
			 {
				 	wz = chassis_calc_wz_speed(TWO_ROTOR_WZ_MIN+0.25f,TWO_ROTOR_WZ_MAX+0.25f,HAL_GetTick())*8.0f;		 
			 }

		 }else if(Data_gyo_A.data.bit.Three)
		 {
			 if(Data_gyo_A.data.bit.NOMAL_FAST_MODE)
			 {
			    wz = chassis_calc_wz_speed(THREE_ROTOR_WZ_MIN-0.2f,THREE_ROTOR_WZ_MAX-0.2f,HAL_GetTick())*8.0f;
			 }
			 else if(Data_gyo_A.data.bit.NOMAL_SLOW_MODE)
			 {
			 		wz = chassis_calc_wz_speed(THREE_ROTOR_WZ_MIN,THREE_ROTOR_WZ_MAX,HAL_GetTick())*8.0f;
			 }
			 else if(Data_gyo_A.data.bit.SPEED_UP_MODE)
			 {
				 	wz = chassis_calc_wz_speed(THREE_ROTOR_WZ_MIN+0.15f,THREE_ROTOR_WZ_MAX+0.15f,HAL_GetTick())*8.0f;		 
			 }

		 }
    }
    *vx_set = cos_yaw * vx - sin_yaw * vy;
    *vy_set = sin_yaw * vx + cos_yaw * vy;
	*wz_set=wz;
}
```

## Gimbal
云台这部分的主要也是分为遥控器控制和键盘控制两个时期，不过这个时候你就要把小陀螺模式和Yaw、Pitch两轴的一些稳定性相关的代码写完和测试玩，就是卡尔曼滤波和前馈控制，这里的注意事项倒不是特别的多，主要出现的问题也就是两轴的稳定性而已，所以只要你们把他们两个调好之后问题就不是很大，除此之外也就是和其他部分的联动了，其实一般这都是按照你们的需求来写的，当他们的稳定性弄好了之后，你们后期需要什么加什么就行。相比于其他部分，这里有个自瞄的控制，是需要和视觉那边对接的，不过在云台这里也就是写一些逻辑相关的代码。而后期主要加的也就是鼠标对其进行控制，他们的控制方面都写在gimbal_mode那边。
```c
void gimbal_relative_angle_control(fp32 *yaw, fp32 *pitch, gimbal_control_t *gimbal_control_set)
{

    if (yaw == NULL || pitch == NULL || gimbal_control_set == NULL)
    {
        return;
    }
    static int16_t yaw_channel = 0, pitch_channel = 0;

		
    rc_deadband_limit(first_order_filter_ch0.out, yaw_channel, RC_DEADBAND);
    rc_deadband_limit(first_order_filter_ch1.out, pitch_channel, RC_DEADBAND);

	gimbal_control_set->speed_x = mouse_x_speed();
	gimbal_control_set->speed_y = mouse_y_speed();
		
    *yaw = yaw_channel * YAW_RC_SEN - gimbal_control_set->speed_x * YAW_MOUSE_SEN;
    *pitch = pitch_channel * PITCH_RC_SEN + gimbal_control_set->speed_y * PITCH_MOUSE_SEN;
}
```
```c
void gimbal_autoaim_angle_control(fp32 *yaw, fp32 *pitch, AutoAim_Data_Rx_t *AutoAim_Data_Rx)
{

		if (yaw == NULL || pitch == NULL || AutoAim_Data_Rx == NULL)
		{
            return;
            }
		if(YawAutoAim_Data_Update==true)
		{
			*yaw = AutoAim_Data_Rx->add_yaw;
			*pitch= AutoAim_Data_Rx->add_pitch;
			//YawAutoAim_Data_Update=false;
		}else{
			*yaw =  -mouse_x_speed() * YAW_MOUSE_SEN;
			*pitch = -mouse_y_speed() * PITCH_MOUSE_SEN;
		}
}
 ```
## Shoot
这里我重点讲述的是关于卡弹的问题，也是在后期出现问题最多的地方。这里跟机械那边的设计有很大的关系，有的地方不是我们能解决的，我们要做的就是尽可能的将他变得稳定，想要彻底解决就要看机械那边了。为了防止卡单，我这边的发射逻辑是电机一下鼠标，电机转两下设置，每转一下是30°，因为我们现在的拨齿是6齿，所以子弹完整的发出去需要的是转60°。点击鼠标转两下是逻辑是，当你鼠标按下的时候它会转一下，当你检测到松开的时候，它又会转一下。而这样做的目的是为了防止电机一次性转的角度太大，产生的力很大，导致弹丸卡弹的问题。
```c
if (set_shoot_contorl->shoot_mode == SHOOT_PREE)
{


     if(Shoot_Struct.key_time>10)
     {
		hoot_Struct.plate_motor.motor_set_angle = Shoot_Struct.plate_motor.motor_set_angle - SINGLE_BIG_TRIGGER_ADD_ANGLE;
        set_shoot_contorl->PREE_ACOM = 1;		 
        set_shoot_contorl->flag = 1;
		set_shoot_contorl->flag_l =1; 
		Shoot_Struct.key_time = 0;
	 }
	 else if(Shoot_Struct.key_time<15)
	 {
		Shoot_Struct.key_time++;
	 }
}

else if (set_shoot_contorl->shoot_mode == SHOOT_LOOSE)
{	

	 if(Shoot_Struct.key_time1>10)
	{
        Shoot_Struct.plate_motor.motor_set_angle = Shoot_Struct.plate_motor.motor_set_angle - SINGLE_BIG_TRIGGER_ADD_ANGLE;
		set_shoot_contorl->LOOSE_ACOM = 1;	
        set_shoot_contorl->flag_l = 0;
		set_shoot_contorl->flag = 0;
	    Shoot_Struct.key_time1 = 0;		 
	 }
	else if(Shoot_Struct.key_time1<15)
	{
		Shoot_Struct.key_time1++;
	}
}
```

除此之外我这里还有一个关于卡弹的检测和反转的逻辑，但只能解决一部分的卡卡弹问题，因为现在的拨盘反转会很卡，很容易卡死，如果你们解决了这个问题之后可以再去尝试尝试。

```c
static void trigger_motor_turn_back(void)
{

    if( Shoot_Struct.block_time < BLOCK_TIME)
    {
       Shoot_Struct.plate_motor.motor_set_angle = Shoot_Struct.plate_motor.motor_set_angle;
    }
    else
    {
	if(Shoot_Struct.key_time2>10)
	 {
        Shoot_Struct.plate_motor.motor_set_angle = Shoot_Struct.plate_motor.motor_set_angle + SINGLE_BIG_TRIGGER_ADD_ANGLE_BACK;
		Shoot_Struct.key_time2 = 0;
		Shoot_Struct.block_time = 0;
	}
	else if(Shoot_Struct.key_time2<15)
	{
		 Shoot_Struct.key_time2++;
	} 
    }
    if(fabs(Shoot_Struct.plate_motor.motor_speed) < BLOCK_TRIGGER_SPEED)
    {
        Shoot_Struct.block_time++;
    }

    else 
    {
        Shoot_Struct.block_time = 0;
    }		
}
```

## 建议

其他的建议我到没有太多，主要一个就是上面已经说了把双板换为一个单C板进行调试，然后就是把CAN那部分好好的想一想，CAN的通道要好好理一理思路，然后关于理线的一些思路，这东西单讲没用，我觉得如果那辆英雄要拆的话，我建议你可以去看看他的线路，然后自己去拆一拆，这对你以后布线会有很大的帮助，其他的倒没有很多了，现就这些吧。