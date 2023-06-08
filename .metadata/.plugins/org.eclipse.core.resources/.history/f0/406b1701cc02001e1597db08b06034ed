/*
 * I2C_EndEffector.h
 *
 *  Created on: 1 มิ.ย. 2566
 *      Author: natch
 */

#ifndef INC_I2C_ENDEFFECTOR_H_
#define INC_I2C_ENDEFFECTOR_H_
#include "stm32f4xx_hal.h"
#include "math.h"
#include "string.h"
//End Effector Function
void ENDEFF_SOFT_RESET(I2C_HandleTypeDef* hi2c);
void ENDEFF_EMERGENCY(I2C_HandleTypeDef* hi2c);
void ENDEFF_EMERGENCY_QUIT(I2C_HandleTypeDef* hi2c);
void ENDEFF_TEST_MODE(I2C_HandleTypeDef* hi2c);
void ENDEFF_TEST_MODE_QUIT(I2C_HandleTypeDef* hi2c);
void ENDEFF_GRIPPER_RUNMODE(I2C_HandleTypeDef* hi2c);
void ENDEFF_GRIPPER_IDLE(I2C_HandleTypeDef* hi2c);
void ENDEFF_GRIPPER_PICK(I2C_HandleTypeDef* hi2c);
void ENDEFF_GRIPPER_PLACE(I2C_HandleTypeDef* hi2c);
//I2C Comm with base system
void I2C_TO_BASESYSTEM(uint8_t DataFrame,I2C_HandleTypeDef *hi2c);
#endif /* INC_I2C_ENDEFFECTOR_H_ */
