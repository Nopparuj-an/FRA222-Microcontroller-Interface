/*
 * Joystick.c
 *
 *  Created on: Jun 3, 2023
 *      Author: natch
 */
#include "Joystick.h"
#include "usart.h"

#define BUFFER_SIZE 6
uint8_t RxBuffer[1];

void UARTInterruptConfig() {
	HAL_UART_Receive_IT(&huart1, RxBuffer, 1);
}

//void Joystick_Received(int* receivedByte){
//	if(RxBuffer[0] == 1 && RxBuffer[5] == 1)
//	{
//		for(int i = 0; i< BUFFER_SIZE;i++) receivedByte[i] = RxBuffer[i];
//		if(RxBuffer[1] > UINT8_MAX/2) receivedByte[1] -= UINT8_MAX+1;
//		if(RxBuffer[2] > UINT8_MAX/2) receivedByte[2] -= UINT8_MAX+1;
//	}
//	else for(int j = 0; j< BUFFER_SIZE;j++) RxBuffer[j] = 0;
//	HAL_UART_Receive_IT(&huart1, RxBuffer, sizeof(RxBuffer));
//}

void Joystick_Received(int *receivedByte) {
	static int count;
	static enum {
		START, COUNT
	} Joy_State = START;
	switch (Joy_State) {
	case (START):
		if (RxBuffer[0] == 69) {
			Joy_State = COUNT;
		}
		break;

	case (COUNT):
		if (RxBuffer[0] == 69) {
			for (int i = 0; i < sizeof(receivedByte); i++) {
				receivedByte[i] = 0;
			}
			count = 0;
		} else if (RxBuffer[0] == 71 && count < 4) {
			for (int i = 0; i < sizeof(receivedByte); i++) {
				receivedByte[i] = 0;
			}
			count = 0;
		} else if (RxBuffer[0] == 71 && count == 4) {
			count = 0;
			Joy_State = START;
			// All data received
		} else {
			receivedByte[count] = RxBuffer[0];
			if (receivedByte[count] > UINT8_MAX / 2)
				receivedByte[count] -= UINT8_MAX + 1;
			count++;
		}
		break;
	}
	HAL_UART_Receive_IT(&huart1, RxBuffer, 1);
}
