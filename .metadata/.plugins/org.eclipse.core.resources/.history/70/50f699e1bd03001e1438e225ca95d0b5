/*
 * Joystick.c
 *
 *  Created on: Jun 3, 2023
 *      Author: natch
 */
#include "Joystick.h"
#include "usart.h"

#define BUFFER_SIZE 6

uint8_t RxBuffer[BUFFER_SIZE];

void UARTInterruptConfig() {
	HAL_UART_Receive_IT(&huart1, RxBuffer, sizeof(RxBuffer));
}

void Joystick_Received(int *receivedByte) {
	if (RxBuffer[0] == 1 && RxBuffer[5] == 1) {
		for (int i = 0; i < BUFFER_SIZE; i++) {
			receivedByte[i] = RxBuffer[i];
		}
		if (RxBuffer[1] > UINT8_MAX / 2) {
			receivedByte[1] -= UINT8_MAX + 1;
		}
		if (RxBuffer[2] > UINT8_MAX / 2) {
			receivedByte[2] -= UINT8_MAX + 1;
		}
	} else {
		memset(RxBuffer, 0, sizeof(RxBuffer));
		HAL_UART_MspDeInit(&huart1);
		HAL_UART_MspInit(&huart1);
		MX_USART1_UART_Init();
	}

	HAL_UART_Receive_IT(&huart1, RxBuffer, sizeof(RxBuffer));
}
