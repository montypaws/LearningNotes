# 使用C++语言编写固高运动控制卡的运动控制程序

## 研究动机
当前实验室的桥式吊车运动控制平台是基于matlab中的simulink实现的，借助与matlab及其实时内核，可以方便的实现各种控制算法，由于其实现需要依赖于固高公司提供的simulink库，不能充分发挥控制卡的功能，并且不利于远程控制的实现。使用C++语言可以方便的实现控制卡的各种功能，并且为实现远程控制打下基础。

## 实现工具
**软件**
1. 运动控制卡的驱动和对应VC动态链接库。
2. 软件开发平台----VS2010旗舰版

**硬件**
1. PC机和运动控制卡
2. 对应的桥式吊车平台（电机驱动、电机、和响应的编码器）

## 实验平台介绍
**欠驱动吊车实验平台**

所谓“*欠驱动系统*”是指系统的独立控制变量个数小于系统自由度个数的一类非线性系统。在节约能量、降低造价、减轻重量、增强系统灵活度等方面都较完全驱动系统优越。欠驱动系统结构简单，便于进行整体的动力学分析和试验。同时由于系统的高度非线性、参数摄动、多目标控制要求及控制量受限等原因，欠驱动系统又足够复杂，便于研究和验证各种算法的有效性。当驱动器故障时，可能使完全驱动系统成为欠驱动系统，欠驱动控制算法可以起到容错控制的作用。从控制理论的角度看，欠驱动系统控制输入的限制是具有挑战性的控制问题，研究欠驱动机械系统的控制问题有助于非完整约束系统控制理论的发展。桥式吊车、Pendubot (Pendulum Robot)、Acrobot (Acrobat Robot)、倒立摆系统都是典型的欠驱动系统。


实验室中的桥式吊车实验平台的控制量为作用在x、y、z三个方向上的力矩，而待控制的状态量有五个：三个方向上的位置，以及在x、y方向上的摆角

对于实验室中控制卡可以采集五个待控制的状态量，并且在控制卡的开环工作模式下，可以方便的实现各种控制算法使用控制卡的每个轴输出响应的电压（模拟量），进而控制电机驱动器输出响应的力矩。从而控制作用在x、y、z三个方向上的力矩。

通过师兄们使用的积累，实验平台上各变量之间的转换关系已经有了明确的数学表达式，请参考马博军师兄的论文和相关资料：
[三维桥式吊车实验平台说明书](e:\Crane\马博军\2_吊车实验设备\总结文档\三维桥式吊车实验平台说明书_全\)


**PID控制算法**
```
/**********************************************************************************************//**
 * @file	testDAC.cpp
 *
 * @brief	Implements the test DAC class.
 **************************************************************************************************/
#include <windows.h>
#include "conio.h"
#include <time.h>
#include <math.h>
#include "Gt400.h"
#include <iostream>
#include <fstream>

/**********************************************************************************************//**
 * @def	AXISNUM
 *
 * @brief	A macro that defines the number of axises.
 *
 * @author	Jsqu
 * @date	2016/4/29
 **************************************************************************************************/

#define AXISNUM 4

/**********************************************************************************************//**
 * @def	ENC_SENSE
 *
 * @brief	A macro that defines encode encount director .
 *
 * @author	Jsqu
 * @date	2016/4/29
 **************************************************************************************************/

#define ENC_SENSE 0x0000

/**********************************************************************************************//**
 * @def	LIMIT_SENSE
 *
 * @brief	A macro that defines effective level of the limit switch.
 *
 * @author	Jsqu
 * @date	2016/4/29
 **************************************************************************************************/

#define LIMIT_SENSE 0x00

/**********************************************************************************************//**
 * @def	ADC_FILTER_K
 *
 * @brief	A macro that defines ADC filter parameter.
 *
 * @author	Jsqu
 * @date	2016/4/29
 *
 * @param	short	The short.
 **************************************************************************************************/

#define ADC_FILTER_K (short)0.0247

/**********************************************************************************************//**
 * @def	CTRL_MODE
 *
 * @brief	A macro that defines control mode.
 *  * "0":Represent the close loop mode.  
 *  * "1":Represent the open loop mode.
 *
 * @author	Jsqu
 * @date	2016/4/29
 **************************************************************************************************/

#define CTRLMODE_A 0     //设置控制模式为模拟量方式。
#define CTRLMODE_D 1     //设置控制模式为脉冲方式。

/**********************************************************************************************//**
 * @def	MOTION_MODE
 *
 * @brief	A macro that defines motion mode.
 * * "0":Represent the motion mode is the TrapMode.  
 * * "1":Represent the motion mode is the JogMode.
 *
 * @author	Jsqu
 * @date	2016/4/29
 **************************************************************************************************/

#define MOTION_MODE 0
#define pi 3.1415926535
#define AXISY 1			//y方向位置测量	
#define AXISX 2			//\x方向位置测量
#define AXISZ 3			//z方向位置测量
#define AXISThetaX 1	//\x方向摆角测量
#define AXISThetaY 4	//y方向摆角测量
using namespace std;
ofstream fout;
/**********************************************************************************************//**
 * @fn	void commandhandler(char *command, short error)
 *
 * @brief	Return value processing function,Handle the motion controller command.
 *
 * @author	Jsqu
 * @date	2016/4/29
 *
 * @param [in,out]	command	the motion controller command.
 * @param	error		   	represent the return value of the command.
 **************************************************************************************************/

void commandhandler(char *command, short error)
{
	// 如果指令执行返回值为非0，说明指令执行错误，向屏幕输出错误结果
	if(error)
	{
		printf("%s = %d\n", command, error);
	}
}

/**********************************************************************************************//**
 * @fn	void initialGTSV()
 *
 * @brief	Initial the motion controller.
 *
 * @author	Jsqu
 * @date	2016/4/29
 **************************************************************************************************/

struct pid
{
	double Kp;
	double Ki;
	double Kd;
};
void initialGTSV()
{
	short sRtn=0;
	//open the control card
	printf("******************************************************\n");
	printf("*Welcome to using googoltech's motion controller card!\n");
	printf("*The frame of the controller is %s\n","GT-400-SV");
	printf("******************************************************\n");
	//reset the control card
	sRtn=GT_Reset();
	commandhandler("GT_Reset",sRtn);

	sRtn=GT_EncSns(ENC_SENSE);//Set the direction of the encoder.
	commandhandler("GT_EncSns",sRtn);

	sRtn=GT_LmtSns(LIMIT_SENSE);//Set effective electrical level for limit switch.
	commandhandler("GT_LmtSns",sRtn);

	for(int i=1;i<=AXISNUM;i++)
	{
		sRtn=GT_Axis(i);
		commandhandler("GT_Axis",sRtn);
		sRtn=GT_ClrSts();
		commandhandler("GT_ClrSts",sRtn);
		//switch off the alarms
		sRtn=GT_AlarmOff();
		commandhandler("GT_AlarmOff",sRtn);
		// switch off the limits. 
		sRtn=GT_LmtsOff();
		commandhandler("GT_LmtsOff",sRtn);
		//setting the control mode of the axis
		sRtn=GT_CtrlMode(CTRLMODE_A);
		commandhandler("GT_CtrlMode",sRtn);
		sRtn=GT_OpenLp();
		commandhandler("GT_OpenLp",sRtn);
		sRtn=GT_AxisOn();
		commandhandler("GT_AxisOn",sRtn);
	}
	for(int i=1;i<=AXISNUM;i++)
	{
		GT_Axis(i);
		GT_SetMtrCmd(0);
	}
}





void convertToVoltageXY(double f,short axis)
{
	double torque=f*0.03;
	double d02=torque*1000/(0.6*10);
	double drift=0;
	short voltage=0;
	if(f>0)
		drift=1200;
	else 
		drift=-600;
	voltage=short(d02*100/3+drift);
	GT_Axis(axis);
	GT_SetMtrCmd(voltage);
}

 void convertToVoltageZ(double f,short axis)
{
	double torque=f*0.03;
	double d02=torque*100/(0.6*9);
	double drift=0;
	short voltage=0;
	if(f>0)
		drift=1100;
	else 
		drift=-800;
	voltage=short(d02*100+drift);
	GT_Axis(axis);
	GT_SetMtrCmd(voltage);
}

void GetSwingAngel(short axis1,short axis2,double* angel)
{
	long theta1=0;
	long theta2=0;
	GT_EncPos(axis1,&theta1);
	GT_Axis(axis2);
	GT_GetAtlPos(&theta2);
	double tan_theta1=tan(pi/12000*theta1);
	double tan_theta2=tan(pi/12000*theta2);
	angel[0]=atan(sqrt(double(2))/2*(tan_theta1-tan_theta2));
	angel[1]=atan(sqrt(double(2))/2*(tan_theta1+tan_theta2)*cos(angel[0]));
}


double GetXPosition(short axis)
{
	long encoder=0;
	double px=0.0;
	GT_Axis(axis);
	GT_GetAtlPos(&encoder);
	cout<<"The value of the "<<axis<<" encoder is"<<encoder<<endl;
	px=encoder*6*pi*1.0e-7;//encoder*0.25*0.1*0.0004*0.06*pi;
	cout<<"The position of X axis is "<<px<<"(m)."<<endl;
	return px;
}


double GetYPosition(short axis)
{
	long encoder=0;
	double py=0.0;
	GT_Axis(axis);
	GT_GetAtlPos(&encoder);
	cout<<"the value of the "<<axis<<"st encoder is"<<encoder<<endl;
	py=encoder*pi/12000;
	cout<<"The position of Y axis is "<<py<<"(m)."<<endl;
	return py;
}
double GetZPosition(short axis)
{
	long encoder=0;
	double pz=0.0;
	GT_Axis(axis);
	GT_GetAtlPos(&encoder);
	cout<<"the value of the "<<axis<<" encoder is"<<encoder<<endl;
	pz=encoder*2.0e-6*pi/9;//encoder*0.25*0.0004*0.02*pi/9;
	cout<<"The position of Z axis is "<<pz<<"(m)."<<endl;
	return pz;
}

void craneControl_PID(double * position)
{
	pid Kpid_x={63,10,0};
	pid Kpid_y={93,14,0};
	double x=GetXPosition(AXISX);
	double y=GetYPosition(AXISY);
	double angel[2]={0.0,0.0};
	GetSwingAngel(AXISThetaX,AXISThetaY,angel);
	DWORD oldTime=timeGetTime();
	double x_old=0.0f;
	double y_old=0.0f;
	double thetaX_old=0.0f;
	double thetaY_old=0.0f;
	double e_x=0.0f;
	double e_y=0.0f;
	double d_x=0.0f;
	double d_y=0.0f;
	double de_x=0.0f;
	double de_y=0.0f;
	double dtheta_x=0.0f;
	double dtheta_y=0.0f;
	double u_x=0.0f;
	double u_y=0.0f;
	double Ka=1.0f;
	DWORD newTime=0;
	DWORD period=0;
	while(1)
	{
		Sleep(2);
		newTime=timeGetTime();
		period=(newTime-oldTime);
		oldTime=newTime;
		e_x=x-position[0];
		e_y=y-position[1];
		d_x=(x-x_old)*1000/period;
		d_y=(y-y_old)*1000/period;
		de_x=d_x;
		de_y=d_y;
		dtheta_x=(angel[0]-thetaX_old)*1000/period;
		dtheta_y=(angel[1]-thetaY_old)*1000/period;
		u_x=-Kpid_x.Kp*(e_x-Ka*sin(angel[0]))-Kpid_x.Kd*(de_x-Ka*cos(angel[0])*dtheta_x);
		u_y=-Kpid_y.Kp*(e_x-Ka*sin(angel[0]))-Kpid_y.Kd*(de_x-Ka*cos(angel[0])*dtheta_x);
		convertToVoltageXY(u_x,AXISX);
		convertToVoltageXY(u_y,AXISY);
		x_old=x;
		y_old=y;
		thetaX_old=angel[0];
		thetaY_old=angel[1];
		fout<<x<<"\t"<<y<<"\t"<<endl;
		x=GetXPosition(AXISX);
		y=GetXPosition(AXISY);
		GetSwingAngel(AXISThetaX,AXISThetaY,angel);
		if(_kbhit())
		{
			break;
		}
	}
}


int main(int argc, char* argv[])
{
	fout.open("result.txt");
	fout<<"x\t"<<"y\t"<<endl;
	short sRtn=0;
	sRtn=GT_Open();
	if(!sRtn)
	{
		//首先进行初始化	
		initialGTSV();
		double obj[2]={0.5,0.5};
		craneControl_PID(obj);
	}
	for(int i=1;i<=AXISNUM;i++)
	{
		GT_Axis(i);
		GT_SetMtrCmd(0);
		GT_AxisOff();
	}
	GT_Close();
	fout.close();
	return 0;
}
```
