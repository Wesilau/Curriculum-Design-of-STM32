# Curriculum-Design-of-STM32 控制技术课程设计

> 任务：速度估计/电流检测及越限报警

## 1 软件及资料

* mc_api.md介绍库函数

## 2 实验内容

### 2.1 速度估计
0. 主要函数  
```
MC_GetMecSpeedAverageMotor1 ( void )
MC_ProgramSpeedRampMotor1( int16_t hFinalSpeed , uint16_thDurationms )
```

1. 电机循环运转

2. 锯齿速度波形
### 2.2 电流检测及越限报警


0. 主要函数  
```
MC_GetIabMotor1( void )
HAL_GPIO_TogglePin(GPIOA,GPIO_PIN_5)
```

2. 电流越限LD2亮

3. 解决电流越限的误报警
