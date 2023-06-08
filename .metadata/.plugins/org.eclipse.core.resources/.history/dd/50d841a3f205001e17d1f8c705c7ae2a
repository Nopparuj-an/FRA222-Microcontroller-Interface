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

void Joystick_Transmit(int16_t Position_x, int16_t Position_y) {
	static int LastPos_x;
	static int LastPos_y;
	static uint8_t data[4];

	data[0] = Position_x >> 8;
	data[1] = Position_x & 0xFF;
	data[2] = Position_y >> 8;
	data[3] = Position_y & 0xFF;

	if (Position_x != LastPos_x || Position_y != LastPos_y) {
		static uint32_t timestamp = 0;
		if (HAL_GetTick() - timestamp > 50) {
			timestamp = HAL_GetTick() + 50;
			HAL_UART_Transmit_DMA(&huart1, data, sizeof(data));
		} else {
			return;
		}
	}

	LastPos_x = Position_x;
	LastPos_y = Position_y;
}

void Joystick_Received(int *receivedByte) {
	static int count;
	static uint8_t tempData[6];
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
			for (int i = 0; i < sizeof(tempData); i++) {
				tempData[i] = 0;
			}
			count = 0;
		} else if (RxBuffer[0] == 71 && count < sizeof(tempData)) {
			for (int i = 0; i < sizeof(tempData); i++) {
				tempData[i] = 0;
			}
			count = 0;
		} else if (RxBuffer[0] == 71 && count == 6) {
			count = 0;
			receivedByte[0] = (tempData[1] << 8) | tempData[0];
			receivedByte[1] = (tempData[3] << 8) | tempData[2];
			receivedByte[2] = tempData[4];
			receivedByte[3] = tempData[5];
			if (receivedByte[0] > UINT16_MAX / 2)
				receivedByte[0] -= UINT16_MAX + 1;
			else if (receivedByte[1] > UINT16_MAX / 2)
				receivedByte[1] -= UINT16_MAX + 1;
			Joy_State = START;
			// All data received
			joystick_callback();
		} else {
			tempData[count] = RxBuffer[0];
			count++;
		}
		break;
	}
	HAL_UART_Receive_IT(&huart1, RxBuffer, 1);
}
