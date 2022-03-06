最近，开始接触Arduino平台，我认为造轮子的是提高代码能力的最有效的方式。

> 造轮子：简明来说就是重新实现已有的功能，以达到更高效、更稳定等的目的

以下程序例程来自[太极创客官网](http://www.taichi-maker.com/)，此平台的Arduino教程深入浅出，对于想学习Arduino的同学，首推太极创客团队。



![](../Picture/QQ图片20220227102323.png)

------

以下便是今天练习的造轮子程序：

```c++
// Joy——MeArm

/*
MeArm机械臂控制程序
机械臂的控制有两种模式，分别为：
指令模式；在串口中输入格式为“ b90 ”的数据，控制“ base ”舵机运转到“ 90° ”位置；
按键模式：在串口中输入wsad/8456,控制机械臂的自由运动。
两种模式可以通过'm'字符切换。
*/

#include <Servo.h>            //导入Servo库
Servo base, rArm, fArm, claw; //创建四个Servo对象

//存储电机极限值——使用const关键字
const int baseMIn = 0;
const int baseMax = 180;
const int rArmMIn = 45;
const int rArmMax = 180;
const int fArmMIn = 3;
const int fArmMax = 120;
const int clawMIn = 25;
const int clawMax = 100;

int DSD = 15;     //DSD——Default Servo Delay
bool mode = true; //mode=1 指令模式  mode=0 按键模式
int moveStep = 3; //每一次按下按键 舵机的移动量

void setup() {       //初始化内容包括：伺服电机连接PWM、伺服电机初始化、串口初始化
    base.attach(11); //base 伺服舵机连接引脚11 舵机代号'b'
    delay(200);      //稳定性等待
    rArm.attach(10); //rArm 伺服舵机连接引脚10 舵机代号'r'
    delay(200);      //稳定性等待
    fArm.attach(9);  //fArm 伺服舵机连接引脚9 舵机代号'f'
    delay(200);      //稳定性等待
    claw.attach(6);  //claw 伺服舵机连接引脚6 舵机代号'c'
    delay(200);      //稳定性等待  
    
    base.write(90);
    delay(10);
    rArm.write(90);
    delay(10);
    fArm.write(90);
    delay(10);
    claw.write(90);
    delay(10);
    
    Serial.begin(9600);
    Serial.println("Ths is MeArm!"); 
}

void loop() {                   //反复读取串口数据
    if (Serial.available()>0) { //如果串口有数据输入
        char serialCmd = Serial.read();
        if (mode) {
            armDataCmd(serialCmd);
        } else {
            armJoyCmd(serialCmd);      
        }
    }   
}

//五个子函数
void armDataCmd(char seerialCmd) { //指令模式
    if(serialCmd == 'w' || serialCmd == 's' || serialCmd == 'a' || 
      serialCmd == 'd' || serialCmd == '8' || serialCmd == '5' || 
      serialCmd == '4' || serialCmd == '6'){
        Serial.println("+Warning: Robot in Instruction Mode...");
        delay(100);
        while(Serial.available()>0) char wrongCommand = Serial.read();
        return;
    }
    
    if(serialCmd == 'b' || serialCmd == 'r' || serialCmd == 'f' || 
      serialCmd == 'c') {
        int servoData = Serial.parseInt();
        servoCmd(servoCmd, servoData, DSD);
    } else {
        switch (serialCmd){
			case 'm' :   //切换至手柄模式 
      			mode = 0; 
        		Serial.println("Command: Switch to Joy Mode.");
       		    break;
            case 'o':
                reportStatus();
                break;
            case 'i:
                armIniPos();
                break;                
            default:
                Serial.println("Unknown Command.");     
        }
    }
 
}

void armJoyCmd(char serialCmd) { // 按键模式
    if(serialCmd == 'b' || serialCmd == 'r' ||
       serialCmd == 'f' || serialCmd == 'c' ){
        Serial.println("+Warning: Robot in Joy Mode...");
        delay(100);
        while(Serial.available()>0) char wrongCommand = Serial.read();
        return;
    }
    
    int baseJoyPos;
    int rArmJoyPos;
    int fArmJoyPos;
    int clawJoyPos;
    switch(serialCmd){
        case 'a':
            Serial.println("Receive Command : Base turn Left");
            baseJoyPos = base.read() - moveStep;
            servoCmd('b', baseJoyPos, DSD);
            break;
        case 'd':
            Serial.println("Receive Command : Base turn Left");
            baseJoyPos = base.read() + moveStep;
            servoCmd('b', baseJoyPos, DSD);
            break;            
        case 'w':
            Serial.println("Receive Command : Base turn Left");
            baseJoyPos = rArm.read() + moveStep;
            servoCmd('r', baseJoyPos, DSD);
            break;
        case 's':
            Serial.println("Receive Command : Base turn Left");
            baseJoyPos = rArm.read() - moveStep;
            servoCmd('r', baseJoyPos, DSD);
            break;            
        case '4':
            Serial.println("Receive Command : Base turn Left");
            baseJoyPos = claw.read() - moveStep;
            servoCmd('c', baseJoyPos, DSD);
            break;
        case '6':
            Serial.println("Receive Command : Base turn Left");
            baseJoyPos = claw.read() + moveStep;
            servoCmd('c', baseJoyPos, DSD);
            break;            
        case '8':
            Serial.println("Receive Command : Base turn Left");
            baseJoyPos = fArm.read() + moveStep;
            servoCmd('f', baseJoyPos, DSD);
            break;
        case '5':
            Serial.println("Receive Command : Base turn Left");
            baseJoyPos = fArm.read() - moveStep;
            servoCmd(fr', baseJoyPos, DSD);
            break;            
		case 'm' :   //切换至指令模式 
      		mode = 0; 
        	Serial.println("Command: Switch to Instruction Mode.");
       		break;
        case 'o':
            reportStatus();
            break;
        case 'i:
            armIniPos();
            break;                
        default:
            Serial.println("Unknown Command.");            
    }
 
}

void servoCmd(char servoName, int toPos, int servoDelay) { //???
    Servo servo2go;
    //串口监视器输出接受指令信息
    Serial.println("");
    Serial.print("+Command: Servo ");    
    Serial.print(servoName);
    Serial.print(" to ");
    Serial.print(toPos);
    Serial.print(" at servoDelay value ");
    Serial.print(servoDelay);
    Serial.println(".");
    
    int fromPos;
    
    switch (servoName) {
        case 'b':
            if(toPos >= baseMin && toPos <= baseMax) {
                servo2fo = base;
                fromPos = base.read(); //获取当前电机角度值
                break;
            } else {
                Serial.println("+Warning: base Servo Value Out Of Limit!");
                return; 
            }
            
        case 'r':
            if(toPos >= rArmMin && toPos <= rArmMax) {
                servo2fo = rArm;
                fromPos = rArm.read(); //获取当前电机角度值
                break;
            } else {
                Serial.println("+Warning: rArm Servo Value Out Of Limit!");
                return;
            }
 
            if(toPos >= fArmMin && toPos <= fArmMax) {
                servo2fo = fArm;
                fromPos = fArm.read(); //获取当前电机角度值
                break;
            } else {
                Serial.println("+Warning: fArm Servo Value Out Of Limit!");
                return;
            }
            
            if(toPos >= clawMin && toPos <= clawMax) {
                servo2fo = claw;
                fromPos = claw.read(); //获取当前电机角度值
                break;
            } else {
                Serial.println("+Warning: claw Servo Value Out Of Limit!");
                return;
            }                                
    }
    
	
	//指挥电机运行
    if(fromPos <= toPos) {
        for(int i=fromPos; i<=toPos; i++) {
       		servo2go.write(i);  
            delay(servoDelay);
        } 
    } else {
        for(int i=fromPos; i>=toPos; i--) {
       		servo2go.write(i);  
            delay(servoDelay);
        }
    }       
}

void reportStatus() { //显示当前状态
    Serial.println("");
    Serial.println("");
    Serial.println("+ MeArm Status Report +");
    Serial.print("Base Position: "); Serial.println(base.read());
    Serial.print("Rear Arm Position: "); Serial.println(rArm.read());
    Serial.print("Front Arm Position: "); Serial.println(fArm.read());
    Serial.print("Claw Position: "); Serial.println(claw.read());
    Serial.println("++++++++++++++++++++++++++");
    Serial.println("");
}

void armIniPos() { //初始化舵机位置
    Serial.println("+Command: Restore Initial Position.");
    int robotIniPosArray[4][3] = {
        {'b', 90, DSD},
        {'r', 90, DSD},
        {'f', 90, DSD},
        {'c', 90, DSD},
    };
     for (int i=0; i<4; i++) {
         servoCmd(robotIniPosArray[i][0],
                  robotIniPosArray[i][1], 
                  robotIniPosArray[i][2],
                  robotIniPosArray[i][3]);
     }
}
```



