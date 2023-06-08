#ifndef INC_MOTORENCODER_H_
#define INC_MOTORENCODER_H_

// PRIVATE INCLUDE ================================================================================

#include "tim.h"

// PRIVATE VARIABLE ===============================================================================

extern int32_t homeoffset;

// PRIVATE FUNCTION PROTOTYPE =====================================================================

void motor(float voltage);
int32_t getLocalPosition();
int32_t getRawPosition();

// USER CODE ======================================================================================

void motor(float voltage) {
	if (voltage > 0) {
		// forward
		if (voltage > 25000) {
			voltage = 25000;
		}
		HAL_GPIO_WritePin(GPIOA, GPIO_PIN_11, RESET);
	} else if (voltage < 0) {
		// backward
		voltage *= -1.0;
		if (voltage > 25000) {
			voltage = 25000;
		}
		HAL_GPIO_WritePin(GPIOA, GPIO_PIN_11, SET);
	} else {
		// stop
		voltage = 0;
	}

	__HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_1, voltage);
}

int32_t getLocalPosition() {
	return __HAL_TIM_GET_COUNTER(&htim2) - homeoffset;
}

int32_t getRawPosition() {
	return __HAL_TIM_GET_COUNTER(&htim2);
}

// USER CODE END ==================================================================================

#endif /* INC_MOTORENCODER_H_ */
