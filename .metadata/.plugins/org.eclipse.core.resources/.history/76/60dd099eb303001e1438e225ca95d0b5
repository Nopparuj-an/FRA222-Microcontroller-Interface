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

void UARTInterruptConfig(){
	HAL_UART_Receive_IT(&huart1, RxBuffer, sizeof(RxBuffer));
}

void Joystick_Received(UART_HandleTypeDef *huart,int* receivedByte){
	if(huart == &huart1){
		if(RxBuffer[0] == 1 && RxBuffer[5] == 1)
		{
			for(int i = 0; i< BUFFER_SIZE;i++) receivedByte[i] = RxBuffer[i];
			if(RxBuffer[1] > UINT8_MAX/2) receivedByte[1] -= UINT8_MAX+1;
			if(RxBuffer[2] > UINT8_MAX/2) receivedByte[2] -= UINT8_MAX+1;
		}
		else for(int j = 0; j< BUFFER_SIZE;j++) RxBuffer[j] = 0;
		HAL_UART_Receive_IT(&huart1, RxBuffer, sizeof(RxBuffer));
	}
}
