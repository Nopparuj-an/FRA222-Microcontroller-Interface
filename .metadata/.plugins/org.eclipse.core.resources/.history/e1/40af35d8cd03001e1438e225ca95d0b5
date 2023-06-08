/*
 * trajectory_trapezodal.c
 *
 *  Created on: Jun 3, 2023
 *      Author: user
 */

#include "trajectory_trapezoidal.h"
#include <math.h>

// PRIVATE TYPEDEF ================================================================================

// trajectory
// position;
float abs_distance = 0;
float initial_position = 0;
float position = 0;

// velocity
float velocity = 0;

// acceleration
float acceleration = 0;

// time
int sign = 0;

// set setpoint
float setpoint_now = 0;
float target = 0;

// USER CODE ======================================================================================

void Trajectory(float setpoint_now, float velocity_max, float acceleration_max, int *position_out, float *velocity_out, float *acceleration_out, int homemode) {
	// velocity
	static float max_velocity = 0;

	// calculation position
	static float position_acc = 0;
	static float position_const = 0;

	// calculation time
	static float time_acc = 0;
	static float time_const = 0;
	static float time_total = 0;
	static float time_trajectory = 0;
	static float time_err = 0;

	static float setpoint_past = 0;
	static float distance = 0;

	if(homemode == 1){
		setpoint_past = setpoint_now;
		time_trajectory = 0;
		abs_distance = 0;
		distance = 0;
		position = 0;
		initial_position = 0;
		return;
	}

	else if(homemode == 0)
	{
		// distance and +-(sign)
		if (setpoint_past != setpoint_now) {
			distance = setpoint_now - initial_position;
			setpoint_past = setpoint_now;
			if (distance >= 0) {
				sign = 1;
				abs_distance = distance;
			} else if (distance < 0) {
				sign = -1;
				abs_distance = distance * (-1);
			}
		} else {
			setpoint_past = setpoint_now;
		}

	// Define pattern of trapezoidal_trajectory
	if (abs_distance > ((velocity_max * velocity_max) / acceleration_max)) {
		time_acc = ((velocity_max - 0) / acceleration_max);
		time_const = ((1.0 / velocity_max) * ((abs_distance) - ((velocity_max * velocity_max) / acceleration_max)));
		time_total = (2 * time_acc) + (abs_distance - (velocity_max * velocity_max) / acceleration_max) / velocity_max;
		max_velocity = velocity_max * sign;
	}

	else {
		time_acc = sqrt(abs_distance / acceleration_max);
		time_total = time_acc * 2;
		time_const = 0;
		position_const = 0;
		max_velocity = acceleration_max * time_acc * sign;
	}

	//acceleration segment
	if ((0 <= time_trajectory) && (time_trajectory < time_acc)) {
		time_trajectory += 0.0001;
		position = initial_position + (0.5 * acceleration_max * (time_trajectory * time_trajectory) * sign);
		velocity = (acceleration_max * time_trajectory * sign);
		position_acc = position;
		acceleration = acceleration_max * sign;
	}

	//constant segment
	else if ((time_trajectory) < (time_total - time_acc)) {
		time_trajectory += 0.0001;
		position = position_acc + (max_velocity * (time_trajectory - time_acc));
		position_const = position - position_acc;
		velocity = (max_velocity);
		acceleration = 0;
	}

	//deceleration segment
	else if (((time_total - time_acc) <= time_trajectory) && (time_trajectory < time_total)) {
		time_trajectory += 0.0001;
		time_err = (time_trajectory - (time_acc + time_const));
		position = position_acc + position_const + (max_velocity * time_err) + (0.5 * (-1) * acceleration_max * (time_err * time_err) * sign);
		velocity = (-acceleration_max * sign * time_err) + (max_velocity);
		acceleration = -acceleration_max * sign;
		initial_position = position;
	}

	position = (int) position;

	if (setpoint_now == position) {
		time_trajectory = 0;
	}
	}

	*position_out = position;
	*velocity_out = velocity;
	*acceleration_out = acceleration;
}

