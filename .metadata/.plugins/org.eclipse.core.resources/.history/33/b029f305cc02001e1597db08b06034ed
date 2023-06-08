/*
 * I2C_EndEffector.c
 *
 *  Created on: 1 มิ.ย. 2566
 *      Author: natch
 */

#include "I2C_EndEffector.h"
#include "math.h"
#include "ModBusRTU.h"

#define SLAVE_ADDR (0x15<<1) //End Effector Address

char binary[8];

void ENDEFF_SOFT_RESET(I2C_HandleTypeDef *hi2c) {
	static uint8_t data[4] = { 0x00, 0xFF, 0x55, 0xAA };
	HAL_I2C_Master_Transmit(hi2c, SLAVE_ADDR, data, 4, HAL_MAX_DELAY);
	HAL_Delay(10);
}
void ENDEFF_EMERGENCY(I2C_HandleTypeDef *hi2c) {
	static uint8_t data[1] = { 0xF0 };
	HAL_I2C_Master_Transmit(hi2c, SLAVE_ADDR, data, 1, HAL_MAX_DELAY);
	HAL_Delay(10);
}
void ENDEFF_EMERGENCY_QUIT(I2C_HandleTypeDef *hi2c) {
	static uint8_t data[4] = { 0xE5, 0x7A, 0xFF, 0x81 };
	HAL_I2C_Master_Transmit(hi2c, SLAVE_ADDR, data, 4, HAL_MAX_DELAY);
	HAL_Delay(10);
}
void ENDEFF_TEST_MODE(I2C_HandleTypeDef *hi2c) {
	static uint8_t data[2] = { 0x01, 0x11 };
	HAL_I2C_Master_Transmit(hi2c, SLAVE_ADDR, data, 2, HAL_MAX_DELAY);
	HAL_Delay(10);
}
void ENDEFF_TEST_MODE_QUIT(I2C_HandleTypeDef *hi2c) {
	static uint8_t data[2] = { 0x01, 0x00 };
	HAL_I2C_Master_Transmit(hi2c, SLAVE_ADDR, data, 2, HAL_MAX_DELAY);
	HAL_Delay(10);
}
void ENDEFF_GRIPPER_RUNMODE(I2C_HandleTypeDef *hi2c) {
	static uint8_t data[2] = { 0x10, 0x13 };
	HAL_I2C_Master_Transmit(hi2c, SLAVE_ADDR, data, 2, HAL_MAX_DELAY);
	HAL_Delay(10);
}
void ENDEFF_GRIPPER_IDLE(I2C_HandleTypeDef *hi2c) {
	static uint8_t data[2] = { 0x10, 0x8C };
	HAL_I2C_Master_Transmit(hi2c, SLAVE_ADDR, data, 2, HAL_MAX_DELAY);
	HAL_Delay(10);
}
void ENDEFF_GRIPPER_PICK(I2C_HandleTypeDef *hi2c) {
	static uint8_t data[2] = { 0x10, 0x5A };
	HAL_I2C_Master_Transmit(hi2c, SLAVE_ADDR, data, 2, HAL_MAX_DELAY);
	HAL_Delay(2000);
}
void ENDEFF_GRIPPER_PLACE(I2C_HandleTypeDef *hi2c) {
	static uint8_t data[2] = { 0x10, 0x69 };
	HAL_I2C_Master_Transmit(hi2c, SLAVE_ADDR, data, 2, HAL_MAX_DELAY);
	HAL_Delay(2000);
}
//void decimalToBinary(uint8_t decimal) {
//	int index = 0;
//	for (int i = 0; i < sizeof(binary); i++) {
//		binary[i] = 0;
//	}
//	if (decimal == 0) {
//		binary[index++] = '0';
//	} else {
//		while (decimal > 0) {
//			binary[index++] = (decimal & 1) ? '1' : '0';
//			decimal >>= 1;
//		}
//	}
//	binary[index] = '0'; // Add null terminator to indicate the end of the string
//}
void I2C_TO_BASESYSTEM(uint8_t DataFrame,I2C_HandleTypeDef *hi2c) {
	int index = 0;
	for (int i = 0; i < sizeof(binary); i++) {
		binary[i] = 0;
	}
	if (DataFrame == 0) {
		binary[index++] = '0';
	} else {
		while (DataFrame > 0) {
			binary[index++] = (DataFrame & 1) ? '1' : '0';
			DataFrame >>= 1;
		}
	}
	binary[index] = '0'; // Add null terminator to indicate the end of the string

	static int lastTest;
	static int lastRun;
	static int lastPick;
	if (binary[0] == '1' && lastTest == 0) {
		ENDEFF_TEST_MODE(hi2c);
		lastTest = 1;
	} else if (binary[0] == '0' && lastTest == 1) {
		ENDEFF_TEST_MODE_QUIT(hi2c);
		lastTest = 0;
	} else if (binary[1] == '1' && lastRun == 0) {
		ENDEFF_GRIPPER_RUNMODE(hi2c);
		lastRun = 1;
	} else if (binary[1] == '0' && lastRun == 1) {
		ENDEFF_GRIPPER_IDLE(hi2c);
		lastRun = 0;
	} else if (binary[2] == '1' && binary[1] == '1' && lastPick == 0) {
		ENDEFF_GRIPPER_PICK(hi2c);
		DataFrame = 0b0010;
		lastPick = 1;
	} else if (binary[3] == '1' && binary[1] == '1' && lastPick == 1) {
		ENDEFF_GRIPPER_PLACE(hi2c);
		DataFrame = 0b0010;
		lastPick = 0;
	}
}
//void TransmitToBase(char* Data,uint8_t result)
//{
//	  char NewData[8];
//	  result = 0;
//	  for(int i = 0; i < 4;i++)
//	  {
//		  if(i == 0) NewData[2] = Data[i];
//		  else if(i == 1) NewData[3] = Data[i];
//		  else if(i == 2) NewData[1] = Data[i];
//		  else if(i == 3) NewData[0] = Data[i];
//	  }
//
//	  for(int j = 0; j<sizeof(NewData);j++)
//	  {
//		  if(NewData[j] == '\0') j = sizeof(NewData);
//		  else result += ((int)NewData[j]-'0')*pow(2,j);
//	  }
//}
