### IIC简介

- 简介

  > **IIC**（Inter-Integrated Circuit）是 IIC Bus 简称，中文叫集成电路总线，是一种通信协议
  >
  > IIC使用两根信号线进行通信：一根时钟线SCL，一根数据线SDA
  >
  > ![原始接线图](.\img\原始接线图.png)

- 分类

  > 1. **软件iic**
  >   - 灵活性高
  >    
  >   - 硬件占用少
  >    
  >   - 调试方便
  >    
  >2. **硬件iic**
  >   - 性能高
  >   - 稳定性强
  >   -  资源占用少
  
- 主从机总线
  ![线路图](.\img\线路图.png)
  
  >1. 主机
  >
  >>- 主机可以发起和结束一次通讯，在这里stm32作为主机
  >>- 主机可通过地址 读取从机(寄存器)数据和向从机(寄存器)写入数据
  >
  >2. 从机
  >
  >>- 从机在总线上有唯一地址(出场时就决定的)，主机通过地址来查找从机
  >>- 常用的从机
  >>  eg. OLED显示屏 MPU6050 存储器
  
- 信号类型

  ![时序图](.\img\时序图.png)

  > 1. **开始信号**
  >
  >     SCL高电平期间，SDA从高电平切换到低电平
  >
  > 2. **结束信号**
  >
  >    SCL高电平期间，SDA从低电平切换到高电平
  >
  > 3. **应答信号**
  >
  >    主机在接收完一个字节之后，在下一个时钟发送一位数据，数据0表示应答，数据1表示非应答
  >
  > 4. **数据有效性**
  >
  >    只有当SCL为高电频的时候 SDA的数据才是有效数据，并且SDA这时候不能改变数据，只有当SCL为低电平的时候，SDA才允许改变状态
  >
  > 5. **数据传输**(8bits)
  >
  > 6. **空闲状态**
  >
  >    SCL和SDA都为高电平

- 执行逻辑

  > - **写时序**
  >   ![写时序](.\img\写时序.png)
  >
  >   1. 开始信号
  >   2. 发送从机地址位
  >   3. 写控制位
  >   4. 等待从机ACK应答
  >   5. 再次发送开始信号
  >   6. 发送写入寄存器的地址
  >   7. 等待从机ACK应答
  >   8. 发送写入的数据
  >   9. 等待从机ACK应答
  >   10. 停止信号
  >
  > - **读时序**
  >
  >   ![读时序](.\img\读时序.png)
  >
  >   1. 开始信号
  >   2. 发送从机地址位
  >   3. 写控制位
  >   4. 等待从机ACK应答
  >   5. 重新发送开始信号
  >   6. 发送读取寄存器的地址
  >   7. 等待从机ACK应答
  >   8. 重新发送开始信号
  >   9. 发送从机地址位
  >   10. 读控制位
  >   11. 等待从机ACK应答
  >   12. 从机发送寄存器数据
  >   13. 主机发送NACK信号，表示读取完成
  >   14. 主机发送结束信号
  >   
  > - 代码
  >
  > ~~~c
  > /*硬件iic初始化*/
  > GPIO_InitTypeDef GPIO_InitStructure;
  > GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_OD; //复用开漏输出模式 可以拉低作用电平无法真正输出高电平 使得从机和主机都对本条线有控制权
  > GPIO_InitStructure.GPIO_Pin = GPIO_Pin_10 | GPIO_Pin_11;
  > GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
  > GPIO_Init(GPIOB, &GPIO_InitStructure);
  > 
  > I2C_InitTypeDef I2C_InitStructure;
  > I2C_InitStructure.I2C_Mode = I2C_Mode_I2C;
  > I2C_InitStructure.I2C_ClockSpeed = 50000;
  > I2C_InitStructure.I2C_DutyCycle = I2C_DutyCycle_2;
  > I2C_InitStructure.I2C_Ack = I2C_Ack_Enable;
  > I2C_InitStructure.I2C_AcknowledgedAddress = I2C_AcknowledgedAddress_7bit;
  > I2C_InitStructure.I2C_OwnAddress1 = 0x00;
  > I2C_Init(I2C2, &I2C_InitStructure);
  > 
  > I2C_Cmd(I2C2, ENABLE);
  > 
  > /*软件iic初始化 初始化引脚就行*/
  > RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
  > 	
  > GPIO_InitTypeDef GPIO_InitStructure;
  > GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_OD; //开漏输出 可以拉低作用电平无法真正输出高电平 使得从机和主机都对本条线有控制权
  > GPIO_InitStructure.GPIO_Pin = GPIO_Pin_10 | GPIO_Pin_11;
  > GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
  > GPIO_Init(GPIOB, &GPIO_InitStructure);
  > 
  > GPIO_SetBits(GPIOB, GPIO_Pin_10 | GPIO_Pin_11);
  > ~~~
  >
  > 





### MPU6050

- 简介

  >- MPU6050是一个6轴姿态传感器，可以测量芯片自身X、Y、Z轴的加速度、角速度参数(可拓展磁感传感器和气压传感器)
  >- 3轴加速度计:测量X、Y、Z轴的加速度
  >
  >>   ![图片1](.\img\图片1.png)
  >- 3轴陀螺仪传感器 : 测量X、Y、Z轴的角速度
  >>   ![图片2](.\img\图片2.jpg)
  >>   - **部分参数**
  >
  >>      16位ADC采集传感器的模拟信号，量化范围：-32768~32767
  >>
  >>      加速度计满量程选择：±2、±4、±8、±16（g）
  >>      陀螺仪满量程选择： ±250、±500、±1000、±2000（°/sec）
  >
  >>      - 寄存器配置
  >>     ~~~c
  >>     #define	MPU6050_SMPLRT_DIV		0x19
  >>     #define	MPU6050_CONFIG			0x1A
  >>     #define	MPU6050_GYRO_CONFIG		0x1B
  >>     #define	MPU6050_ACCEL_CONFIG	0x1C
  >>     #define	MPU6050_ACCEL_XOUT_H	0x3B
  >>     #define	MPU6050_ACCEL_XOUT_L	0x3C
  >>     #define	MPU6050_ACCEL_YOUT_H	0x3D
  >>     #define	MPU6050_ACCEL_YOUT_L	0x3E
  >>     #define	MPU6050_ACCEL_ZOUT_H	0x3F
  >>     #define	MPU6050_ACCEL_ZOUT_L	0x40
  >>     #define	MPU6050_TEMP_OUT_H		0x41
  >>     #define	MPU6050_TEMP_OUT_L		0x42
  >>     #define	MPU6050_GYRO_XOUT_H		0x43
  >>     #define	MPU6050_GYRO_XOUT_L		0x44
  >>     #define	MPU6050_GYRO_YOUT_H		0x45
  >>     #define	MPU6050_GYRO_YOUT_L		0x46
  >>     #define	MPU6050_GYRO_ZOUT_H		0x47
  >>     #define	MPU6050_GYRO_ZOUT_L		0x48  
  >>     #define	MPU6050_PWR_MGMT_1		0x6B
  >>     #define	MPU6050_PWR_MGMT_2		0x6C
  >>     #define	MPU6050_WHO_AM_I		0x75
  >>     ~~~

  

- 欧拉角和四元数
  
  >- 欧拉角
  >
  >X 轴角度（**滚转角 Roll**）即为绕 X 轴旋转方向的角度，
  >Y 轴角度（**俯仰角 Pitch**）即为绕 Y 轴旋转方向的角度，
  >Z 轴角度（**偏航角 Yaw**）即为绕 Z 轴旋转方向的角度
  >
  >![欧拉角](.\img\欧拉角.png)
  >
  >- DMP
  >
  >DMP就是MPU6050内部的运动引擎，全称Digital Motion Processor，可以直接输出四元数，但是算法是闭源的。
  >
  >四元数可以直接转换成欧拉角
  >
  >- 姿态融合
  >
  >常用的姿态融合算法有四元数法 、一阶互补算法和卡尔曼滤波算法，其中第三者最为常用。
  >
  >卡尔曼滤波算法简述：卡尔曼滤波就是一种数据融合算法，结合加速度计和陀螺仪的信息，共同来解算姿态，集合二者的优点获得在动态环境下可以准确测量姿态的方法。
  >
  >- 四元数转换欧拉角
  >
  >~~~c
  >/* 计算得到俯仰角/横滚角/航向角 */
  >pitch = asin(-2 * q1 * q3 + 2 * q0* q2)* 57.3;
  >roll  = atan2(2 * q2 * q3 + 2 * q0 * q1, -2 * q1 * q1 - 2 * q2* q2 + 1)* 57.3;
  >yaw   = atan2(2*(q1*q2 + q0*q3),q0*q0+q1*q1-q2*q2-q3*q3) * 57.3;
  >~~~
  >
  >
  
  
  
  
  
   
  
  
  
  
  
  
  
  
  
  
  
  

