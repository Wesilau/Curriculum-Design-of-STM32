mc_api介绍及使用示例
----------------
https://blog.csdn.net/Soonjn/article/details/103006565
* * *

  使用ST FOC控制的电机开发环境MC SDK 即使用Workbench调用cubeMx生成源码工程后。进行基础的电机调试后，可以根据需要调用库函数中给出的[API](https://so.csdn.net/so/search?q=API&spm=1001.2101.3001.7020)接口，实现用户自己的电机控制方式。  
  FOC库给用户调用的电机控制接口，全部在mc_api.c文件块当中。这里面的函数就是用户与电机库控制之间的桥梁。ST 的MC SDK是可以用一片MCU同时独立驱动两台电机的，ST为每一台电机提供一组单独的函数接口供调用。并且这些函数的名字和参数都非常简化，很容易看出其功能和需要传入的参数。

* * *

**先列出在mc_spi.h文件中申明的函数。然后在逐一解释每个函数的用法和注意点。**  
仅列出Motor1的相关函数。如果使用的是双电机，则M2的使用与M1的完全一致。

    /* Starts Motor 1 */
    bool MC_StartMotor1(void);
    
    /* Stops Motor 1 */
    bool MC_StopMotor1(void);
    
    /* Programs a Speed ramp for Motor 1 */
    void MC_ProgramSpeedRampMotor1( int16_t hFinalSpeed, uint16_t hDurationms );
    
    /* Programs a Torque ramp for Motor 1 */
    void MC_ProgramTorqueRampMotor1( int16_t hFinalTorque, uint16_t hDurationms );
    
    /* Programs a current reference for Motor 1 */
    void MC_SetCurrentReferenceMotor1( qd_t Iqdref );
    
    /* Returns the state of the last submited command for Motor 1 */
    MCI_CommandState_t  MC_GetCommandStateMotor1( void);
    
    /* Stops the execution of the current speed ramp for Motor 1 if any */
    bool MC_StopSpeedRampMotor1(void);
    
    /* Returns true if the last submited ramp for Motor 1 has completed, false otherwise */
    bool MC_HasRampCompletedMotor1(void);
    
    /* Returns the current mechanical rotor speed reference set for Motor 1, expressed in the unit defined by #SPEED_UNIT */
    int16_t MC_GetMecSpeedReferenceMotor1(void);
    
    /* Returns the last computed average mechanical rotor speed for Motor 1, expressed in the unit defined by #SPEED_UNIT */
    int16_t MC_GetMecSpeedAverageMotor1(void);
    
    /* Returns the final speed of the last ramp programmed for Motor 1, if this ramp was a speed ramp */
    int16_t MC_GetLastRampFinalSpeedMotor1(void);
    
    /* Returns the current Control Mode for Motor 1 (either Speed or Torque) */
    STC_Modality_t MC_GetControlModeMotor1(void);
    
    /* Returns the direction imposed by the last command on Motor 1 */
    int16_t MC_GetImposedDirectionMotor1(void);
    
    /* Returns the current reliability of the speed sensor used for Motor 1 */
    bool MC_GetSpeedSensorReliabilityMotor1(void);
    
    /* returns the amplitude of the phase current injected in Motor 1 */
    int16_t MC_GetPhaseCurrentAmplitudeMotor1(void);
    
    /* returns the amplitude of the phase voltage applied to Motor 1 */
    int16_t MC_GetPhaseVoltageAmplitudeMotor1(void);
    
    /* returns current Ia and Ib values for Motor 1 */
    ab_t MC_GetIabMotor1(void);
    
    /* returns current Ialpha and Ibeta values for Motor 1 */
    alphabeta_t MC_GetIalphabetaMotor1(void);
    
    /* returns current Iq and Id values for Motor 1 */
    qd_t MC_GetIqdMotor1(void);
    
    /* returns Iq and Id reference values for Motor 1 */
    qd_t MC_GetIqdrefMotor1(void);
    
    /* returns current Vq and Vd values for Motor 1 */
    qd_t MC_GetVqdMotor1(void);
    
    /* returns current Valpha and Vbeta values for Motor 1 */
    alphabeta_t MC_GetValphabetaMotor1(void);
    
    /* returns the electrical angle of the rotor of Motor 1, in DDP format */
    int16_t MC_GetElAngledppMotor1(void);
    
    /* returns the current electrical torque reference for Motor 1 */
    int16_t MC_GetTerefMotor1(void);
    
    /* Sets the reference value for Id */
    void MC_SetIdrefMotor1( int16_t hNewIdref );
    
    /* re-initializes Iq and Id references to their default values */
    void MC_Clear_IqdrefMotor1(void);
    
    /* Acknowledge a Motor Control fault on Motor 1 */
    bool MC_AcknowledgeFaultMotor1( void );
    
    /* Returns a bitfiled showing faults that occured since the State Machine of Motor 1 was moved to FAULT_NOW state */
    uint16_t MC_GetOccurredFaultsMotor1(void);
    
    /* Returns a bitfield showing all current faults on Motor 1 */
    uint16_t MC_GetCurrentFaultsMotor1(void);
    
    /* returns the current state of Motor 1 state machine */
    State_t  MC_GetSTMStateMotor1(void);


##### MC_StartMotor1()

 调用该函数来启动电机的转动。如果电机当前的状态是空闲状态，则会立刻执行启动电机，进入启动状态；否则，丢弃该指令的执行。因此，使用这个指令前可以检查一下电机的当前运行状态。应用的时候，可以检查返回值以确认是否执行了电机的启动命令。从电机控制机制里的状态机来看，这个函数只是触发状态机从“IDLE”进入“IDLE-START”，如果这个命令成功执行则返回TRUE，否则FALSE。  
 需要注意的是，在使用这个函数之前，需要先调用以下的任意一个函数，否则可能会导致没有预估的错误行为。

*   MC_ProgramSpeedRampMotor1()
*   MC_ProgramTorqueRampMotor1()
*   MC_SetCurrentReferenceMotor1()

 在Workbench中的Monitor中右侧Generic有同样的按钮 “Star Motor”。

##### MC_StopMotor1()

 调用该函数来停止电机的转动。如果电机当前的状态是“RUN”或者“START”，则会立刻执行停止电机名利；否则，丢弃该指令的执行。使用这个指令的时候，可以检查返回值以确认是否执行了电机的停止命令。从电机控制机制里的状态机来看，这个函数只是触发状态机从任意状态进入“ANY_STOP”，检查这个命令实际的执行，可以检查状态机是否变成了“IDLE”状态。  
 在Workbench中的Monitor中右侧Generic有同样的按钮 “Stop Motor”。

##### MC_ProgramSpeedRampMotor1()

 该函数用于设置线性速度目标值，该函数有两个参数，hFinalSpeed是最终目标速度，hDuration是持续时间。最终目标速度的按照ST的转速定义是0.1Hz，转子的0.1HZ也就是0.1r/s。电机转速的常规表达是RPM,即转每分；ST的MCSDK中有三种速度的表达。这里使用的是 0.1转每秒。如果电机电机的转速是1000转，这里的值就是1000\[rpm\]/60s/10=16.7\[0.1rps\]。hDuration的时间是ms。hFinalSpeed的值如果是正值则表示正向转动，如果为负值则表示反向转动。  
 执行这个函数之后，电机的运行状态就是Speed模式。该函数是的电机速度的目标值在运行过程中可以线性的变化，而非跳跃式的突变。  
 调用这个函数的时候，设置并不一定执行，而是立即进入缓存队列中进行等待。当状态机变化到“START\_RUN”或者“RUN”的时候才会立即被执行。任何时刻缓存中的指令只能保存一个，如果在指令等待的过程中有新的指令，那缓存中的指令被新的指令覆盖替换掉。需要知道当前指令是否已经被准确执行，可以调用MC\_GetCommandStateMotor1()函数来获取缓存中最新指令的执行状态。  
 monitor中也有同样的操作选项，“Speed controller”。

##### MC_ProgramTorqueRampMotor1()

 该函数的的控制效果与上一个SpeedRamp是一样的，只是该函数作用的对象是电机转矩。当点击以转矩模式运行的时候，该函数就是用来设置转矩目标值的。  
 其中需要的两个参数，HFinalTorque是运行时候的电机转矩最终目标值，hDuarationms是持续时间。hDurationms的单位是ms。  
 该函数被调用之后，电机的运行会切换到转矩模式。hFinaltorque是16位有符号数表示的Iq值，转换成安培的haul没需要用这个公式：Is16=Iamp_65535_Rs*Aop/Vdd;

##### MC_SetCurrentReferenceMotor1()

 这个函数是直接设置Iq和Id电流参考值的函数，该函数只会在状态机是“START_RUN”或者“RUN”的时候生效，否则就是进入缓存队列等待。  
 该函数需要的参数Iqdref的变量类型时Curr\_Componets。如果其子成员ql\_Componet1的数值是负数，则表示电机反向转动。  
 在monitor中的Current controller设置内容一致。

##### MC_GetCommandStateMotor1()

 这个函数返回最新的指令的执行状态。部分指令在调用的时候，并不一定立马被执行了，而是先放入缓存队列当中，等到合适的时候才会被执行。这个函数就可以知道，缓存中的罪行指令的执行状态。  
 返回值会是下列中的一个：

*   MCI\_BUFFER\_EMPTY:缓存队列空
*   MCI\_COMMAND\_NOT\_ALREADY\_EXECUTED:指令未被执行
*   MCI\_COMMAND\_EXECUTED_SUCCESFULLY指令被成功执行
*   MCI\_COMMAND\_EXECUTED_UNSUCCESFULLY指令执行失败  
     指令被执行之后，无论成功与否，调用该指令都会将指令状态设置为MCI\_BUFFER\_EMPTY。

##### MC_StopSpeedRampMotor1()

 这个函数可以停止目标速度的线性递增。MC_ProgramSpeedRampMotor1()是执行电机目标速度线性控制的；而这个函数则是停止目标速度的线性变化。如果SpeedRamp的指令正在被执行，则该指令将会被立即停止，目标速度将会维持在当前值。  
 由于执行SpeedRamp的过程中被强行停止，所以当前停止后的目标速度是不确定的。  
 monitor中有相应的按钮“Stop Ramp”。

##### MC_HasRampCompletedMotor1()

 该函数用于获取目标速度的变化状态。如果已经执行完SpeedRamp命令则返回True，否则返回false。

##### MC_GetMecSpeedReferenceMotor1()

 该函数用于获取电机机械转速的实际目标值。在调用线性目标速度控制函数MC\_ProgramSpeedRampMotor1()之后可以调用MC\_StopSpeedRampMotor1()来终止指令的执行。终止之后，电机速度有一个不固定的目标值。调用该函数则可以返回这个不确定的目标值。

##### MC_GetMecSpeedAverageMotor1()

 这个函数用来获取电机的平均机械转速，返回值是电机转速的平均值，单位是0.1hz。比如返回值是300，则转每分表示的转速为 300_0.1_60=1800rpm.

##### MC_GetLastRampFinalSpeedMotor1()

 该函数返回用户设置的最终速度目标值。该函数并不是获取电机速度值，也不是获取电机实际速度目标值，而是调用MC_ProgramSpeedRampMotor1()之后所设置的最终目标速度值。

##### MC_GetControlModeMotor1()

 该函数用于获取当前运行模式。电机运行模式有两种，Torque转矩模式和Speed速度模式。其返回值是枚举变量STC\_Modality\_t中的STC\_TORQUE\_MODE或者STC\_SPEED\_MODE

##### MC_GetImposedDirectionMotor1()

 该函数用于获取由最新的指令所设置的电机转动方向值。最新的指令指的是调用控制函数MC\_ProgramSpeedRampMotor1()，MC\_ProgramTorqueRampMotor1()和MC_SetCurrentRenferenceMotor1()。当调用这些函数的时候会设置一个目标值，这个目标值的正负就是电机转动的方向。调用该函数就是获取这个电机的方向。如果电机方向为负则返回-1，否则返回1。

##### MC_GetSpeedSensorReliabilityMotor1()

 检查速度传感器是否可靠。电机的速度传感器反馈回来的数据如果超过了设置的数值范围，就会增加速度反馈错误次数。速度反馈错误次数超过了一定数值，就会认为速度传感器是不可靠的。  
 当速度传感器不可靠时，该函数返回FALSE，否则返回true。

##### MC_GetPhaseCurrentAmplitudeMotor1()

 该函数用于获取电机的相电流（0-to-peak）。其返回值是16位的有符号数s16A。s16A的数字量化为1\[s16A\]=Imax/32768。Imax是可测量的电流最大值，电流采样信号经过运放之后存在偏移值，满量程是0-Imax,不存在负值，所以数字量化的时候是32768而不是65535。  
 由于连接到ADC引脚的电流信号电压被抬升了1.65V,所以在计算Imax的时候需要除以2，最后得到的就是：Imax=Vdd/(2_Rs_Aop)。如果需要转换成安培（Amps）,则使用：Iamp=Is16a_Vdd/(65535_Rs*Aop)。

##### MC_GetPhaseVoltageAmplitudeMotor1()

 该函数用于获取瞬时的电机相电压（0-to-peak）。其返回值是16位有符号数s16V。s16V格式的定义如下：1\[s16V\]=Vmax/32768。如果转换成伏特则：Vvolts=Vs16v\*Vbus/(sqrt(3)\*3268)。

##### MC_GetIabMotor1()

 该函数用于获取电机Ia,Ib电流值。返回值是Curr_Components,这是一个有两个int16数的结构体。

##### MC_GetIalphabetaMotor1()

 该函数用于获取电机Iα,Iβ电流值。返回值是Curr_Components,这是一个有两个int16数的结构体。

##### MC_GetIqdMotor1()

 该函数用于获取电机Iq,Id电流值。

##### MC_GetIqdrefMotor1()

 该函数用于获取电机Iq,Id电流的目标值。

##### MC_GetVqdMotor1()

 该函数用于获取电流值的Vq,Vd。返回值是Volt_Components,这是一个有两个int16数的结构体。

##### MC_GetValphabetaMotor1()

 该函数用于获取电流值的Vα,Vβ。返回值是Volt_Components,这是一个有两个int16数的结构体。

##### MC_GetElAngledppMotor1()

 该函数用于获取电机转子的电角度。返回值是以dpp格式的电角度。  
 在MC API中使用的角度测量单位是s16degree,定义为1\[s16degree\]=2π\[rad\]/65535。

##### MC_GetTerefMotor1()

 该函数用于获取转矩的目标值。返回值是s16格式的Iq电流值。

##### MC_SetIdrefMotor1()

在Speed或Torque目标值爬坡变化的过程，是有FOC内部提供目标值实现的。在目标值爬坡过程，正常情况下电流的目标值Idref是由内部计算提供，然后用于电机控制，儿该函数则允许强行修改Idref的值。该函数对于弱磁控制或者MTPA最大转矩使能的时候是无效的。

##### MC\_Clear\_IqdrefMotor1()

 该函数可以将Iq,Id的目标值重新初始化为默认值。

##### MC_AcknowledgeFaultMotor1()

用户调用这个函数前，如果电机发生了故障，电机将停留在“FAULT_OVER”状态，并保留故障代码。在调用这个函数之后，状态机将清除故障代码的记录，并恢复到IDLE状态。  
 如果在调用该函数的时候状态机不是在错误状态，则该函数返回false，否则返回true。在workbench的monitor上有相同功能的按钮“Fault Ack”。

##### MC_GetOccurredFaultsMotor1()

 该函数用于获取状态机进入“FUALT_NOW”状态之后发生过的故障代码。返回一个16位数的故障代码，0表示没有错误。monitor上有相应的指示灯“Faults”。  
有如下的故障类型：

    #define  MC_NO_ERROR  (uint16_t)(0x0000u)      /**< @brief No error.*/
    #define  MC_NO_FAULTS  (uint16_t)(0x0000u)     /**< @brief No error.*/
    #define  MC_FOC_DURATION  (uint16_t)(0x0001u)  /**< @brief Error: FOC rate to high.*/
    #define  MC_OVER_VOLT  (uint16_t)(0x0002u)     /**< @brief Error: Software over voltage.*/
    #define  MC_UNDER_VOLT  (uint16_t)(0x0004u)    /**< @brief Error: Software under voltage.*/
    #define  MC_OVER_TEMP  (uint16_t)(0x0008u)     /**< @brief Error: Software over temperature.*/
    #define  MC_START_UP  (uint16_t)(0x0010u)      /**< @brief Error: Startup failed.*/
    #define  MC_SPEED_FDBK  (uint16_t)(0x0020u)    /**< @brief Error: Speed feedback.*/
    #define  MC_BREAK_IN  (uint16_t)(0x0040u)      /**< @brief Error: Emergency input (Over current).*/
    #define  MC_SW_ERROR  (uint16_t)(0x0080u)      /**< @brief Software Error.*/


##### MC_GetCurrentFaultsMotor1()

 该函数用于获取当前出现的错误状态。

##### MC_GetSTMStateMotor1()

 该函数用于获取状态机的当前状态。
