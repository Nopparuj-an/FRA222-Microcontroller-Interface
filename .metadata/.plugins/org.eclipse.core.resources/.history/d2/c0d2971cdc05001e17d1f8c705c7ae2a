#ifndef INC_MAINLOGIC_H_
#define INC_MAINLOGIC_H_

// PRIVATE INCLUDE ================================================================================

#include <I2C_EndEffector.h>
#include "i2c.h"

// PRIVATE TYPEDEF ================================================================================

typedef enum {
	MSwait, MSidle, MSpick, MSplace, MShome, MStray, MSpoint, MStestY, MStestXY
} MachineState;

// PRIVATE VARIABLE ===============================================================================

MachineState state = MSidle;

uint8_t PID_enable = 1;
uint8_t home_status = 0;
uint8_t jog_enable = 0;
uint8_t jog_point_n = 0; // 0 - 2
uint8_t tray_point_n = 0; // 0 - 8
uint8_t tray_wait_mode = 0; // 0 ready, 1 wait for move to pick, 2 pick wait, 3 wait for move to place, 4 place wait
uint32_t tray_delay;

extern int receivedByte[4];
extern MB MBvariables;

float voltage;
extern int32_t setpoint_x;
extern int32_t setpoint_y;
extern int32_t setpointtraj_y;
int32_t traj_velocity;
int32_t traj_acceleration;
extern float KP;
extern float KI;
extern float KD;

Coordinate corners[3];
Coordinate pick[9];
Coordinate place[9];
Coordinate origin_pick;
float angle_pick;
Coordinate origin_place;
float angle_place;

// PRIVATE FUNCTION PROTOTYPE =====================================================================

void main_logic(MB *variables);
void interrupt_logic();
void end_effector_gripper(MB *variables, uint8_t mode);	// 0 pick, 1 place
void end_effector_laser(MB *variables, uint8_t mode);	// 0 off, 1 on
void home_handler();
void data_report(MB *variables);
void x_spam_position(MB *variables);
uint8_t move_finished(uint32_t tolerance);
void preset_data_y_only();
void preset_data_xy();

// USER CODE ======================================================================================

void main_logic(MB *variables) {
	ENE_I2C_UPDATE(&variables->end_effector_status, &hi2c1);
	RGB_logic();
	data_report(variables);

	static uint32_t wait_timer;
	switch (state) {
	case MSwait:
		if (HAL_GetTick() - wait_timer > 1500) {
			state = MSidle;
		}
		break;
	case MSidle:
		wait_timer = HAL_GetTick();
		variables->y_moving_status = 0;
		jog_enable = 0;

		if (variables->base_system_status & 0b1) {
			// pick mode
			variables->base_system_status = 0;
			state = MSpick;
			variables->y_moving_status = 1;
			jog_enable = 1;
		}

		if (variables->base_system_status & 0b10) {
			// place mode
			variables->base_system_status = 0;
			state = MSplace;
			variables->y_moving_status = 2;
			jog_enable = 1;
		}

		if (variables->base_system_status & 0b100) {
			// home mode
			variables->base_system_status = 0;
			state = MShome;
			variables->y_moving_status = 4;
			variables->x_target_position = 0;
			variables->x_moving_status = 2;
			//variables->x_moving_status = 1;
		}

		if (variables->base_system_status & 0b1000) {
			// start tray mode
			variables->base_system_status = 0;
			state = MStray;
			tray_point_n = 0;
			tray_delay = HAL_GetTick();
		}

		if (variables->base_system_status & 0b10000) {
			// point mode
			variables->base_system_status = 0;
			state = MSpoint;
			variables->y_moving_status = 32;
		}
		break;
	case MSpick: // MSpick or MSplace
	case MSplace:
		variables->x_target_position = setpoint_x;
		x_spam_position(variables);
		break;
	case MShome:
		if (!home_status) {
			home_status = 1;
			PID_enable = 0;
			voltage = -8000;
		}
		break;
	case MStray:
		variables->x_target_position = setpoint_x;
		x_spam_position(variables);
		switch (tray_wait_mode) {
		case 0:
			// move to pick
			variables->y_moving_status = 8;
			setpoint_x = pick[tray_point_n].x * 10;
			setpoint_y = pick[tray_point_n].y / 0.03;
			variables->x_moving_status = 2;
			tray_wait_mode = 1;
			break;
		case 1:
			// wait for move to finish then pick
			if (move_finished(10)) {
				end_effector_gripper(variables, 0);
				tray_wait_mode = 2;
				tray_delay = HAL_GetTick() + 2200;
			}
			break;
		case 2:
			// wait for pick to finish then move to place
			if (HAL_GetTick() >= tray_delay) {
				variables->y_moving_status = 16;
				setpoint_x = place[tray_point_n].x * 10;
				setpoint_y = place[tray_point_n].y / 0.03;
				variables->x_moving_status = 2;
				tray_wait_mode = 3;
			}
			break;
		case 3:
			// wait for move to place then place
			if (move_finished(10)) {
				end_effector_gripper(variables, 1);
				tray_wait_mode = 4;
				tray_delay = HAL_GetTick() + 2200;
			}
			break;
		case 4:
			// wait for place to finish then reset to state 0
			if (HAL_GetTick() >= tray_delay) {
				tray_wait_mode = 0;
				tray_point_n++;
			}
			break;
		}
		if (tray_point_n >= 9) {
			state = MShome;
		}
		break;
	case MSpoint:
		setpoint_y = variables->goal_point_y / 0.3;
		variables->x_target_position = variables->goal_point_x;
		variables->x_moving_status = 2;

		if (abs(setpoint_y - getLocalPosition()) < 10) {
			state = MSwait;
		}
		break;
	case MStestY:
		preset_data_y_only();
		state = MSidle;
		break;
	case MStestXY:
		preset_data_xy();
		state = MSidle;
		break;
	}
}

void interrupt_logic() {
	// Call trajectory function
	Trajectory(setpoint_y, 34000, 80000, (int*) &setpointtraj_y, (float*) &traj_velocity, (float*) &traj_acceleration, 0);

	// Call PID function
	if (PID_enable) {
		static int count = 0;
		count++;
		if (count >= 5) {
			PositionControlPID(setpointtraj_y, setpoint_y, getLocalPosition(), KP, KI, KD, &voltage);
			count = 0;
		}
	}

	// Call motor function
	motor(voltage);
}

void end_effector_gripper(MB *variables, uint8_t mode) {
	if ((variables->end_effector_status & 0b0010) == 0) {
		return;
	}

	if (!mode) {
		// pick
		variables->end_effector_status |= 0b0100;
	} else {
		// place
		variables->end_effector_status |= 0b1000;
	}
}

void end_effector_laser(MB *variables, uint8_t mode) {
	if (!mode) {
		// off
		variables->end_effector_status &= 0b1110;
	} else {
		// on
		variables->end_effector_status |= 0b0001;
	}
}

void home_handler() {
	if (!home_status) {
		return;
	}
	if (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_3) == GPIO_PIN_SET) {
		return;
	}
	motor(0);
	homeoffset = getRawPosition() + 11500;
	setpointtraj_y = -11500;
	setpoint_y = -11500;
	Trajectory(setpoint_y, 34000, 80000, (int*) &setpointtraj_y, (float*) &traj_velocity, (float*) &traj_acceleration, 1);
	home_status = 0;
	PID_enable = 1;
	state = MSwait;
	setpoint_y = 0;
}

void data_report(MB *variables) {
	variables->y_actual_position = getLocalPosition() * 0.3;
	variables->y_actual_speed = traj_velocity * 0.3;
	variables->y_actual_acceleration = traj_acceleration * 0.3;
}

void x_spam_position(MB *variables) {
	if ((variables->x_actual_position - variables->x_target_position) != 0 && variables->x_moving_status == 0) {
		variables->x_moving_status = 2;
	}
}

void joystick_callback() {
	if (!jog_enable) {
		return;
	}

	setpoint_x += receivedByte[0];
	setpoint_y += receivedByte[1];

	if (setpoint_x > 1400) {
		setpoint_x = 1400;
	} else if (setpoint_x < -1400) {
		setpoint_x = -1400;
	}

	if (setpoint_y > 11667) {
		setpoint_y = 11667;
	} else if (setpoint_y < -11667) {
		setpoint_y = -11667;
	}

	if (receivedByte[2]) {
		corners[jog_point_n].x = setpoint_x / 10.0;
		corners[jog_point_n].y = setpoint_y * 0.03;
		jog_point_n++;
	}
	if (jog_point_n >= 3) {
		if (state == MSpick) {
			localize(corners, pick, &origin_pick, &angle_pick);
			MBvariables.pick_tray_orientation = (360.0 - (angle_pick * 180.0 / M_PI)) * 100.0;
			MBvariables.pick_tray_origin_x = origin_pick.x * 10;
			MBvariables.pick_tray_origin_y = origin_pick.y * 10;
		} else {
			localize(corners, place, &origin_place, &angle_place);
			MBvariables.place_tray_orientation = (360.0 - (angle_place * 180.0 / M_PI)) * 100.0;
			MBvariables.place_tray_origin_x = origin_place.x * 10;
			MBvariables.place_tray_origin_y = origin_place.y * 10;
		}
		state = MSwait;
		jog_point_n = 0;
	}
}

uint8_t move_finished(uint32_t tolerance) {
	if (abs(getLocalPosition() - setpoint_y) < tolerance && abs(MBvariables.x_actual_position - setpoint_x) < tolerance) {
		return 1;
	}
	return 0;
}

void preset_data_y_only() {
	for (int i = 0; i < 9; i++) {
		pick[i].y = 38.0 + 38.0 * i;
		place[i].y = -(38.0 + 38.0 * i);
	}
}

void preset_data_xy() {
	corners[0].x = 7.1;
	corners[0].y = -68.0;
	corners[1].x = 52.7;
	corners[1].y = -29.4;
	corners[2].x = 21.2;
	corners[2].y = 8.2;
	localize(corners, pick, &origin_pick, &angle_pick);
	MBvariables.pick_tray_orientation = (360.0 - (angle_pick * 180.0 / M_PI)) * 100.0;
	MBvariables.pick_tray_origin_x = origin_pick.x * 10;
	MBvariables.pick_tray_origin_y = origin_pick.y * 10;

	corners[0].x = -90.3;
	corners[0].y = -170.0;
	corners[1].x = -46.1;
	corners[1].y = -149.4;
	corners[2].x = -69.6;
	corners[2].y = -95.3;
	localize(corners, place, &origin_place, &angle_place);
	MBvariables.place_tray_orientation = (360.0 - (angle_place * 180.0 / M_PI)) * 100.0;
	MBvariables.place_tray_origin_x = origin_place.x * 10;
	MBvariables.place_tray_origin_y = origin_place.y * 10;
}

// USER CODE END ==================================================================================

#endif /* INC_MAINLOGIC_H_ */
