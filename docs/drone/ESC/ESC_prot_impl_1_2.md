---
layout: post
title: ESC protocols implementation 1
permalink: /docs/drone/ESC/ESC_prot_impl_1_2
nav_order: 2
parent: ESC
grand_parent: Drone
---

<style type="text/css">
  p {
    text-align: justify;
  }
</style>

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>


# Implementation on STM32
Implementation of the analog protocols is really simple since STM timers give us PWM generators with which we define a period of a signal and can decide about the duty cycle. We will change it on the fly after calculating new values in the main loop. For OneShot, we also set up PWM for generating only one period (one pulse mode).

# PWM 

Let's see the setup function for PWM:

```c
#define FREQUENCY_ESC_UPDATE 490	//	frequency of ESC updating
void setup_PWM()
{	//	Enable GPIOA clock:
	RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;
	//	Setup GPIO pins AF mode:
	GPIOA->MODER &= ~GPIO_MODER_MODER2;
	GPIOA->MODER |= GPIO_MODER_MODER2_1;
	GPIOA->MODER &= ~GPIO_MODER_MODER3;
	GPIOA->MODER |= GPIO_MODER_MODER3_1;
	//	Set alternate functions:
	GPIOA->AFR[0] &= ~0x0000FF00;
	GPIOA->AFR[0] |= 0x00001100;	// TIM2 channel 3 and channel 4
	//	Set speed (11 - max speed):
	GPIOA->OSPEEDR |= (GPIO_OSPEEDER_OSPEEDR2 |GPIO_OSPEEDER_OSPEEDR3);
    
	//	Enable TIM2 clock:
	RCC->APB1ENR |= RCC_APB1ENR_TIM2EN;
	//	Register is buffered (any changes will affect the next period):
	TIM2->CR1 |= TIM_CR1_ARPE;
	//	PWM mode 1 and output compare 3, preload enable:
	TIM2->CCMR2 |= TIM_CCMR2_OC3M_1 | TIM_CCMR2_OC3M_2 | TIM_CCMR2_OC3PE;
	//	PWM mode 1 and output compare 4, preload enable:
	TIM2->CCMR2 |= TIM_CCMR2_OC4M_1 | TIM_CCMR2_OC4M_2 | TIM_CCMR2_OC4PE;
	//	channel's polarity -> when channels are active (for PWM 1 mode when CNT<CCR) signal is high:
	TIM2->CCER &= ~TIM_CCER_CC3P;
	TIM2->CCER &= ~TIM_CCER_CC4P;
	//	Channel 3 enable:
	TIM2->CCER |= TIM_CCER_CC3E;
	//	Channel 4 enable:
	TIM2->CCER |= TIM_CCER_CC4E;
	//	Counter counts every 0.5 [us] (typically 1 step is 1 [us]).
	//	For better resolution and easier Dshot implementation it is 0.5[us].
	//	Notice that lowest motor_value (2000) is still 1[ms] long as in typical PWM):
	TIM2->PSC = 84/2 - 1;	// TIM2 source clock is 84 [MHz]
	TIM2->ARR = 2000000 / FREQUENCY_ESC_UPDATE - 1; // 1 period of PWM
	TIM2->CCR3 = 2000; // PWM length channel 3 (1 [ms])
	TIM2->CCR4 = 2000; // PWM length channel 4 (1 [ms])
	//	TIM2 enabling:
	TIM2->EGR |= TIM_EGR_UG;
	TIM2->CR1 |= TIM_CR1_CEN;
 }
```

The above setup is only for 2 motors connected to TIM2 but the code for different timers is the same (check which GPIO pins connect to which timers' channels' outputs). Now when you want to change the PWM duty cycle (probably after the main loop) you write:
```c
 void update_motors()
 {	//	motors values are in range 2000-4000:
 	TIM2->CCR4 = motor_1_value; // value motor 1
	TIM3->CCR3 = motor_2_value; // value motor 2
	TIM3->CCR4 = motor_3_value; // value motor 3
	TIM2->CCR3 = motor_4_value; // value motor 4
 }
 ```

# OneShot125

OneShot125 setup is quite different. We want one pulse of PWM so let's activate One Pulse Mode for the timer. However, we can not generate a high signal for duty-cycle and then low (STM after overflow will go back to high). One Pulse mode allows for generating delay and then a PWM signal (with length calculated based on ARR and CCR values). So maybe we can generate a part of the desired PWM frame which stays low? (for 250 [us] PWM it would be 0 and for 100 [us] duty cycle - 150 [us]). Unfortunately, the delay can not be 0 and to be safe let's use 1 [us]. Now our frame length is 251 [us] and we set (in CCR) how long it will stay low (after this time it will go high and after overflow back to low). ESC has to wait until the end of each frame (not the best but it is max 126 [us] additional delay so better than a whole frame).

```c
void setup_OneShot125()
{	//	Enable GPIOA clock:
	RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;
	//	Setup GPIO pins AF mode:
	GPIOA->MODER &= ~GPIO_MODER_MODER2;
	GPIOA->MODER |= GPIO_MODER_MODER2_1;
	GPIOA->MODER &= ~GPIO_MODER_MODER3;
	GPIOA->MODER |= GPIO_MODER_MODER3_1;
	//	Set alternate functions:
	GPIOA->AFR[0] &= ~0x0000FF00;
	GPIOA->AFR[0] |= 0x00001100;	// TIM2 channel 3 and channel 4
	//	Set speed (11 - max speed):
	GPIOA->OSPEEDR |= (GPIO_OSPEEDER_OSPEEDR2 |GPIO_OSPEEDER_OSPEEDR3);
    
	//	Enable TIM2 clock:
	RCC->APB1ENR |= RCC_APB1ENR_TIM2EN;
	TIM2->CR1 |= TIM_CR1_OPM; //	ONE_PULSE_MODE
	//	PWM 1 mode channel 3:
	TIM2->CCMR2 |= TIM_CCMR2_OC3M_1 | TIM_CCMR2_OC3M_2;
	//	PWM 1 mode channel 4:
	TIM2->CCMR2 |= TIM_CCMR2_OC4M_1 | TIM_CCMR2_OC4M_2;
	//	Channel's polarity -> when channels are active (for PWM 1 mode when CNT<CCR) signal is low:
	TIM2->CCER |= TIM_CCER_CC3P;
	TIM2->CCER |= TIM_CCER_CC4P;
	//	Channel 3 output enable:
	TIM2->CCER |= TIM_CCER_CC3E;
	//	Channel 4 output enable:
	TIM2->CCER |= TIM_CCER_CC4E;
	//	OneShot is 8x faster regular PWM. 
	//	1 step is 1/16 [us] (normally 1 [us]):
	TIM2->PSC = 84 / 16 - 1;
	TIM2->ARR = 3999 + 16;	//	PWM frame length (250+1 [us]) cause min delay required in pulse mode
	TIM2->CCR3 = 4015 - 2000 + 1;	//	PWM duration channel 3 (125 [us])
	TIM2->CCR4 = 4015 - 2000 + 1;	//	PWM duration channel 4 (125 [us])
	//	TIM2 enabling:
	TIM2->EGR |= TIM_EGR_UG;
	TIM2->CR1 |= TIM_CR1_CEN;
 }
 ```

Since PWM generating turns off after each period after updating the duty cycle you need to restart the timers:

```c
void update_motors()
 {	//	Motors values are in range 2000-4000:
 	TIM2->CCR4 = 4015 - motor_1_value + 1; // value motor 1
	TIM3->CCR3 = 4015 - motor_2_value + 1; // value motor 2
	TIM3->CCR4 = 4015 - motor_3_value + 1; // value motor 3
	TIM2->CCR3 = 4015 - motor_4_value + 1; // value motor 4
 	//	Reload and start timers:
	TIM2->EGR |= TIM_EGR_UG;
	TIM3->EGR |= TIM_EGR_UG;
	TIM2->CR1 |= TIM_CR1_CEN;
	TIM3->CR1 |= TIM_CR1_CEN;
 }
 ```

# DShot

Digital protocols are a bit more sophisticated. We need to generate for each frame 16 bits which can have 2 possible lengths of high signal and 2 bits of low signal for reset.  To do this efficiently DMA usage is required. We will make a table of 18 values for the duty cycle of each bit (16 bits of frame and 2 bits of reset). DMA will transfer these values into the timer register after each overflow (which will generate a request).  Also before sending the motor value we need to calculate CRC and add it to the end of the frame after the telemetry bit. Let's start with the setup:

```c
#define DSHOT_MODE 300 // 150/300/600/1200
#define DSHOT_BUFFER_LENGTH 18 // 16 bits of Dshot and 2 for clearing
#define DSHOT_PWM_FRAME_LENGTH 35 // number of steps for each bit
#define DSHOT_1_LENGTH 26 // number of steps for high signal for 1-bit
#define DSHOT_0_LENGTH 13 // number of steps for high signal for 0-bit

void setup_Dshot()
{	//	Enable GPIOA clock:
	RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;
	//	Setup GPIO pin AF mode:
	GPIOA->MODER &= ~GPIO_MODER_MODER3;
	GPIOA->MODER |= GPIO_MODER_MODER3_1;
	//	Set alternate functions:
	GPIOA->AFR[0] &= ~0x0000F000;
	GPIOA->AFR[0] |= 0x00001000;	// TIM2 channel 4
	//	Set speed (11 - max speed):
	GPIOA->OSPEEDR |= GPIO_OSPEEDER_OSPEEDR3;
	
	//	Enable TIM2 clock:
	RCC->APB1ENR |= RCC_APB1ENR_TIM2EN;
	//	Register is buffered and only overflow generate interrupt:
	TIM2->CR1 |= TIM_CR1_ARPE | TIM_CR1_URS;
	//	PWM mode 1 and output compare 4 preload enable:
	TIM2->CCMR2 |= TIM_CCMR2_OC4M_1 | TIM_CCMR2_OC4M_2 | TIM_CCMR2_OC4PE;
	//	TIM2 is 84 [MHz]:
	TIM2->PSC = 84000 / DSHOT_MODE / DSHOT_PWM_FRAME_LENGTH - 1;
	TIM2->ARR = DSHOT_PWM_FRAME_LENGTH - 1;
	TIM2->DIER |= TIM_DIER_CC3DE | TIM_DIER_CC4DE; // DMA request enable for 4th channel
	TIM2->CCR4 = 10; // PWM duration channel 4
	//	Channel 4 output enable:
	TIM2->CCER |= TIM_CCER_CC4E;
	//	TIM2 enabling:
	TIM2->EGR |= TIM_EGR_UG;
	TIM2->CR1 |= TIM_CR1_CEN;
    
	//	DMA setup:
	RCC->AHB1ENR |= RCC_AHB1ENR_DMA1EN; 
	//	Motor 1:
	DMA1_Stream6->CR |= DMA_SxCR_CHSEL_0 | DMA_SxCR_CHSEL_1 // DMA channel selection
		| DMA_SxCR_MSIZE_1 | DMA_SxCR_PSIZE_1 | DMA_SxCR_MINC // memory data size, peripheral data size (size of register can vary between timers), memory increment
		| DMA_SxCR_DIR_0 | DMA_SxCR_TCIE | DMA_SxCR_PL_0; // memory->peripheral, TC interrupt, priority set
	//	Clear flag:
	DMA1->HIFCR |= DMA_HIFCR_CTCIF6;
    
	//	NVIC setup:
	NVIC_EnableIRQ(DMA1_Stream6_IRQn);
 }
 ```
For other motors, the setup is pretty similar but there is one important difference. Some timers have 32-bit registers and some 16-bit. When you transfer 16-bit data to a 16-bit register DMA works as expected the same for 32-bit peripheral from 32-bit memory (remember to set the correct size in DMA setup). But when you want to send to a 32-bit register from a 16-bit memory DMA will send 2 following packages to the register (to fill 32-register). So you want to trick DMA and say that the register is also 16-bit and sending 16 bits to it should be flawless? NO, for some STM (F2/F4/F7) when you perform 16-bit access to a 32-bit register data will be doubled (check _AHB/APB_ bridges (APB) chapter in Reference Manual). So you have to use memory the same size as you register*. Now we need to prepare our DShot buffer and start DMA transmission:

{: .note}
  *You could use bigger memory e.g. 32-bit memory for a 16-bit register (you set both 32-bit sizes in DMA config). FOR TIMERS CCR registers are always 32-bit long and for 16-bit timers upper half is reserved (you will try to write to this part but it is read-only so it will be discarded). But in my opinion, it is a bad idea to try writing somewhere you shouldn't. Be cautious for other registers which can be truly 16-bit and writing 32-bit data will overwrite the next register. I prefer matching data size with peripheral size.
  
```c
uint32_t dshot_buffer_1[DSHOT_BUFFER_LENGTH];	// for TIM3 it has to be uint16_t
void update_motors(timeUs_t current_time)
{	//	Prepare the buffer:
	fill_Dshot_buffer(prepare_Dshot_package(motor_1_value), dshot_buffer_1);
	//	Setup DMA:		
	DMA1_Stream6->PAR = (uint32_t)(&(TIM2->CCR4));
	DMA1_Stream6->M0AR = (uint32_t)(dshot_buffer_1);
	DMA1_Stream6->NDTR = DSHOT_BUFFER_LENGTH;
	//	Start the transmission:
	DMA1_Stream1->CR |= DMA_SxCR_EN;
 }
 
 void fill_Dshot_buffer(uint16_t m1_value, uint32_t *buffer) // for TIM3 pointer should be uint16_t*
 {
 	for (uint8_t i = 2; i < DSHOT_BUFFER_LENGTH; i++)
	{
		if ((1 << (i - 2)) & m1_value)
			buffer[DSHOT_BUFFER_LENGTH - 1 - i] = DSHOT_1_LENGTH;
		else
			buffer[DSHOT_BUFFER_LENGTH - 1 - i] = DSHOT_0_LENGTH;
	}
	// Make 0 pulse after Dshot frame:
	buffer[DSHOT_BUFFER_LENGTH - 1] = 0;
	buffer[DSHOT_BUFFER_LENGTH - 2] = 0;
 }
 
 uint16_t prepare_Dshot_package(uint16_t value)
 {	// Value is in range of 2000-4000 so it needs to be transformed into Dshot range (48-2047)
	value -= 1953;
	if (value > 0 && value < 48)
		value = 48;
	return ((value << 5) | calculate_Dshot_checksum(value));
 }
 
 uint16_t calculate_Dshot_checksum(uint16_t value)
 {	// 12th bit for telemetry on/off (1/0):
	value = value << 1;
	return (value ^ (value >> 4) ^ (value >> 8)) & 0x0F;
 }
One last thing is the DMA interrupt code after the completion of transmission:
 void DMA1_Stream6_IRQHandler(void)
 {
	if (DMA1->HISR & DMA_HISR_TCIF6)
		DMA1->HIFCR |= DMA_HIFCR_CTCIF6;
 }
 ```

And that's it! Basic implementation of PWM, OneShot, and DShot protocols. In the [next post](ESC_prot_impl_2_2), I will describe BDShot in more detail and its implementation with bit-banging (it will be much more complicated). See you there!