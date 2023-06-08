/*
 * Joystick.c
 *
 *  Created on: Jun 3, 2023
 *      Author: natch
 */
#include "Joystick.h"
#include "usart.h"

uint8_t RxBuffer[1];

void UARTInterruptConfig() {
	HAL_UART_Receive_IT(&huart1, RxBuffer, 1);
}

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
