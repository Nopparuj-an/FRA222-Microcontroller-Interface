#ifndef INC_BASESYSTEMMODBUS_H_
#define INC_BASESYSTEMMODBUS_H_

// PRIVATE INCLUDE ================================================================================

// PRIVATE VARIABLE ===============================================================================

// PRIVATE TYPEDEF ================================================================================

typedef struct {
	// read only variables ============================================================================
	int16_t base_system_status;		// [0 set pick, 1 set place, 2 home, 3 tray mode, 4 point mode]
	int16_t goal_point_x;			// telling our system where to go
	int16_t goal_point_y;
	int16_t x_actual_position;
	int16_t x_actual_speed;

	// write only variable ============================================================================
	int16_t y_moving_status;		// [0 jog pick, 1 jog place, 2 home, 3 go pick, 4 go place, 5 go point]
	int16_t y_actual_position;
	int16_t y_actual_speed;
	int16_t y_actual_acceleration;
	int16_t pick_tray_origin_x;
	int16_t pick_tray_origin_y;
	int16_t pick_tray_orientation;
	int16_t place_tray_origin_x;
	int16_t place_tray_origin_y;
	int16_t place_tray_orientation;
	int16_t x_target_position;
	int16_t x_target_speed;
	int16_t x_target_acceleration_time;	// [1 100ms, 2 500ms, 3 1000ms]

	// read/write variables  ==========================================================================
	int16_t heartbeat;					// [0 base system disconnected, 1 base system is connected]
	int16_t end_effector_status;		// [0 laser on/off, 1 power, 2 picking, 3 placing]
	int16_t x_moving_status;			// [0 home, 1 run, 2 jog left -, 3 jog right +]
} MB;

// PRIVATE FUNCTION PROTOTYPE =====================================================================

void modbus_heartbeat_handler(u16u8_t *registerFrame, MB *variables);
void modbus_data_sync(u16u8_t *registerFrame, MB *variables);

// USER CODE ======================================================================================

void modbus_heartbeat_handler(u16u8_t *registerFrame, MB *variables) {
	static int8_t fail = 0;
	static uint32_t timestamp = 0;
	if (HAL_GetTick() >= timestamp) {
		timestamp = HAL_GetTick() + 200;

		// check if the base system send heartbeat
		if (registerFrame[0].U16 == 18537) {
			// success
			variables->heartbeat = 1;
			fail = 0;
		} else {
			// fail, count failure
			if (fail < 126) {
				fail++;
			}
			// if fail is too high then system is disconnected
			if (fail > 9) {
				variables->heartbeat = 0;
			}
		}

		// set heartbeat for base system to see
		registerFrame[0].U16 = 22881;
	}
}

void modbus_data_sync(u16u8_t *registerFrame, MB *variables) {
	// report data back to base system
	registerFrame[0x10].U16 = variables->y_moving_status;
	registerFrame[0x11].U16 = variables->y_actual_position;
	registerFrame[0x12].U16 = variables->y_actual_speed;
	registerFrame[0x13].U16 = variables->y_actual_acceleration;
	registerFrame[0x20].U16 = variables->pick_tray_origin_x;
	registerFrame[0x21].U16 = variables->pick_tray_origin_y;
	registerFrame[0x22].U16 = variables->pick_tray_orientation;
	registerFrame[0x23].U16 = variables->place_tray_origin_x;
	registerFrame[0x24].U16 = variables->place_tray_origin_y;
	registerFrame[0x25].U16 = variables->place_tray_orientation;
	registerFrame[0x41].U16 = variables->x_target_position;
	registerFrame[0x42].U16 = variables->x_target_speed;
	registerFrame[0x43].U16 = variables->x_target_acceleration_time;

	// get data from base system
	variables->goal_point_x = registerFrame[0x30].U16;
	variables->goal_point_y = registerFrame[0x31].U16;
	variables->x_actual_position = registerFrame[0x44].U16;
	variables->x_actual_speed = registerFrame[0x45].U16;

	static int16_t base_system_status_master_temp;
	if (base_system_status_master_temp != registerFrame[0x01].U16) {
		variables->base_system_status = registerFrame[0x01].U16;
		base_system_status_master_temp = variables->base_system_status;
	}

	// update read/write variable
	static int16_t end_effector_status_slave_temp;
	static int16_t end_effector_status_master_temp;
	if (end_effector_status_master_temp != registerFrame[0x02].U16) {
		// there is an update from master
		variables->end_effector_status = registerFrame[0x02].U16;
		end_effector_status_master_temp = variables->end_effector_status;
		end_effector_status_slave_temp = variables->end_effector_status;
	} else if (end_effector_status_slave_temp != variables->end_effector_status) {
		// there is an update locally
		registerFrame[0x02].U16 = variables->end_effector_status;
		end_effector_status_slave_temp = variables->end_effector_status;
		end_effector_status_master_temp = variables->end_effector_status;
	}
	static int16_t x_moving_status_slave_temp;
	static int16_t x_moving_status_master_temp;
	if (x_moving_status_master_temp != registerFrame[0x40].U16) {
		// there is an update from master
		variables->x_moving_status = registerFrame[0x40].U16;
		x_moving_status_master_temp = variables->x_moving_status;
		x_moving_status_slave_temp = variables->x_moving_status;
	} else if (x_moving_status_slave_temp != variables->x_moving_status) {
		// there is an update locally
		registerFrame[0x40].U16 = variables->x_moving_status;
		x_moving_status_slave_temp = variables->x_moving_status;
		x_moving_status_master_temp = variables->x_moving_status;
	}
}

// USER CODE END ==================================================================================

#endif /* INC_BASESYSTEMMODBUS_H_ */
