#ifndef INC_SPEAKER_H_
#define INC_SPEAKER_H_

// PRIVATE INCLUDE ================================================================================

// PRIVATE VARIABLE ===============================================================================

uint8_t speaker_queue = 0;
uint8_t speaker_data[8];
uint8_t speaker_state;

// PRIVATE FUNCTION PROTOTYPE =====================================================================

extern MB MBvariables;

void speaker_play(uint8_t folder, uint8_t track);
void speaker_UART_bitbang();
void speaker_logic();
uint8_t lastmodbusstate = 0;

// USER CODE ======================================================================================

void speaker_logic() {
	switch (speaker_state) {
	case 0:
		if (HAL_GetTick() > 5000) {
			speaker_state = 1;
			if (!MBvariables.heartbeat && !HAL_GPIO_ReadPin(GPIOB, GPIO_PIN_10)) {
				speaker_play(51, 1);
			} else if (!MBvariables.heartbeat) {
				speaker_play(51, 3);
				speaker_state = 2;
			} else if (!HAL_GPIO_ReadPin(GPIOB, GPIO_PIN_10)) {
				speaker_play(51, 2);
				speaker_state = 2;
			}
		}
		break;
	case 1:
		if (HAL_GetTick() > 5500) {
			if (MBvariables.heartbeat || HAL_GPIO_ReadPin(GPIOB, GPIO_PIN_10)) {
				speaker_state = 2;
			} else if (MBvariables.heartbeat) {
				speaker_play(51, 5);
				speaker_state = 2;
			} else if (HAL_GPIO_ReadPin(GPIOB, GPIO_PIN_10)) {
				speaker_play(51, 4);
				speaker_state = 2;
			}
		}
		break;
	case 2:
		if(lastmodbusstate && !MBvariables.heartbeat){
			speaker_play(51, 8);
		}
		break;
	}

	lastmodbusstate = MBvariables.heartbeat;
}

void speaker_UART_bitbang() {
	if (!speaker_queue) {
		return;
	}
	static uint8_t byte_n = 0;
	static uint8_t bit_n = 0;

	if (bit_n == 0) {
		HAL_GPIO_WritePin(GPIOA, GPIO_PIN_4, GPIO_PIN_RESET);
	} else if (bit_n == 9) {
		HAL_GPIO_WritePin(GPIOA, GPIO_PIN_4, GPIO_PIN_SET);
	} else {
		HAL_GPIO_WritePin(GPIOA, GPIO_PIN_4, ((speaker_data[byte_n] >> (bit_n - 1)) & 0x01) ? GPIO_PIN_SET : GPIO_PIN_RESET);
	}

	bit_n++;
	if (bit_n >= 10) {
		bit_n = 0;
		byte_n++;
	}

	if (byte_n >= 8) {
		byte_n = 0;
		speaker_queue = 0;
	}
}

void speaker_play(uint8_t folder, uint8_t track) {
	if (speaker_queue) {
		return;
	}
	speaker_data[0] = 0x7E;
	speaker_data[1] = 0xFF;
	speaker_data[2] = 0x06;
	speaker_data[3] = 0x0F;
	speaker_data[4] = 0x00;
	speaker_data[5] = folder;
	speaker_data[6] = track;
	speaker_data[7] = 0xEF;

	speaker_queue = 1;
}

#endif /* INC_SPEAKER_H_ */
