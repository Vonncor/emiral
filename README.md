emiral
======

clignotement de la led
/**
  ******************************************************************************
  * @file    main.c
  * @author  Ac6
  * @version V1.0
  * @date    01-December-2013
  * @brief   Default main function.
  ******************************************************************************
*/


#include "stm32l0xx.h"
#include "stm32l0xx_nucleo.h"
#include "stm32l0xx_hal.h"
#include "stm32l0xx_hal_gpio.h"
#include "stm32l053xx.h"

TIM_Base_InitTypeDef Tim6_Init = {0x18FF,TIM_COUNTERMODE_UP,0x00FA,TIM_CLOCKDIVISION_DIV1};
TIM_HandleTypeDef Tim6_HandleTypeDef;
void ErrorHandler(void);
void SystemClock_Config(void);
void TIM6_Config(void);
void TIM6_DAC_IRQHandler(void);
void SysTick_Handler(void);


int main(void)
{
	/* Initialisation Timer et interruptions */
    HAL_Init();
    __TIM6_CLK_ENABLE();
    SystemClock_Config();
	TIM6_Config();

  	BSP_LED_Init(LED2); // initialisation des leds

	 if(HAL_TIM_Base_Init(&Tim6_HandleTypeDef) != HAL_OK)
	  {
	    /* Initialization Error */
	    ErrorHandler();
	  }
	if(HAL_TIM_Base_Start_IT(&Tim6_HandleTypeDef) != HAL_OK) // démarre une base de temps , l'argument est une structure de config
	{
		/* Initialization Error */
		ErrorHandler();
	}



	/*Suite Code*/
	while(1);
		//Check=HAL_NVIC_GetPendingIRQ(TIM6_DAC_IRQn);



}


/*Fonction d'init*/
void TIM6_Config(void){
	Tim6_HandleTypeDef.Instance = TIM6;
	//Tim6_HandleTypeDef.Channel=HAL_TIM_ACTIVE_CHANNEL_1;
	Tim6_HandleTypeDef.Lock=HAL_UNLOCKED;
	Tim6_HandleTypeDef.State=HAL_TIM_STATE_READY;
	Tim6_HandleTypeDef.Init=Tim6_Init;




	HAL_NVIC_SetPriority(TIM6_DAC_IRQn, 0, 1);
	HAL_NVIC_EnableIRQ(TIM6_DAC_IRQn); //autoriser le NVIC

}




void SystemClock_Config(void)
{

  RCC_OscInitTypeDef RCC_OscInitStruct;
  RCC_ClkInitTypeDef RCC_ClkInitStruct;

  __PWR_CLK_ENABLE();

  __HAL_PWR_VOLTAGESCALING_CONFIG(PWR_REGULATOR_VOLTAGE_SCALE1);

  RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSI;
  RCC_OscInitStruct.HSIState = RCC_HSI_ON;
  RCC_OscInitStruct.HSICalibrationValue = 16;
  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
  RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSI;
  RCC_OscInitStruct.PLL.PLLMUL = RCC_PLLMUL_4;
  RCC_OscInitStruct.PLL.PLLDIV = RCC_PLLDIV_2;
  HAL_RCC_OscConfig(&RCC_OscInitStruct);

  RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_SYSCLK;
  RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
  RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
  RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV1;
  RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;
  HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_1);

  __SYSCFG_CLK_ENABLE();

}



void ErrorHandler(void)
{
  /* Infinite loop */
  while(1);
  }





/*Fonctions d'interruptions*/

void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim) {

	GPIO_PinState etat;

	etat = HAL_GPIO_ReadPin(LED2_GPIO_PORT,LED2_PIN);

	if (etat==0){
		BSP_LED_On(LED2);
	}
	else {
		BSP_LED_Off(LED2);
	}
}

void TIM6_DAC_IRQHandler(void) {

	HAL_TIM_IRQHandler(&Tim6_HandleTypeDef);
	HAL_NVIC_ClearPendingIRQ(TIM6_DAC_IRQn);

	//Tim6_Init.Period=0xFF06;
}

void SysTick_Handler(void)
{
  HAL_IncTick();
  HAL_SYSTICK_IRQHandler();
}
