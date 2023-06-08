/*
 * PIDControl.c
 *
 *  Created on: Jun 4, 2023
 *      Author: user
 */

#include "PIDControl.h"
#include <math.h>

// PRIVATE TYPEDEF ================================================================================

// USER CODE ======================================================================================

void PositionControlPID(float trajectory_setpoint, float position_now, float Kp, float Ki, float Kd, float *PID_out) {
	// variable
	static float error_first = 0;
	static float error_second = 0;
	static float error_third = 0;
	static float first = 0;
	static float second = 0;
	static float third = 0;

	// error position
	error_first = trajectory_setpoint - position_now;
	//Position.error[0] = PID_position - QEIReadRaw_now;

	// first error
	first = (Kp + Ki + Kd) * error_first;

	// second error
	second = (Kp + (2 * Kd)) * error_second;

	// third error
	third = Kd * error_third;

	// voltage
	*PID_out += first - second + third;

	// set present to past
	error_third = error_second;
	error_second = error_first;

}
