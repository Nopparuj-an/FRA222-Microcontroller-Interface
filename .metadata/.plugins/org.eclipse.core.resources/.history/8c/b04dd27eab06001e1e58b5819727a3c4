#ifndef INC_RGB_H_
#define INC_RGB_H_

// PRIVATE INCLUDE ================================================================================

#include <WS2812B.h>
#include <math.h>
#include <MainLogic.h>

// PRIVATE FUNCTION PROTOTYPE =====================================================================

void RGB_logic(MachineState state, uint8_t point, uint8_t emergency);
void RGB_off();
void RGB_Rainbow();
void RGB_Power_Status();
void RGB_Bootup();
void RGB_BreathingPattern(uint32_t period, uint8_t R, uint8_t G, uint8_t B);
void RGB_TrayProgress(uint8_t point);

// PRIVATE VARIABLE ===============================================================================

uint32_t LEDtime = 0;
MachineState laststate = MSidle;

// USER CODE ======================================================================================

void RGB_logic(MachineState state, uint8_t point, uint8_t emergency) {
	// Run on 200 Hz
	static uint32_t timestamp;
	if (HAL_GetTick() - timestamp < 5) {
		return;
	}
	timestamp = HAL_GetTick() + 5;

	// Main logic here
	if (emergency) {
		laststate = MSwait;
		RGB_BreathingPattern(500, 255, 0, 0);
		WS2812_Send();
		Set_Brightness(45);
		return;
	} else {
		switch (state) {
		case MSwait:
			if (laststate != MSwait) {
				LEDtime = 0;
			}
			RGB_BreathingPattern(500, 0, 255, 0);
			laststate = MSwait;
			break;
		case MSidle:
			if (laststate != MSidle) {
				LEDtime = 0;
			}
			RGB_Rainbow();
			laststate = MSidle;
			break;
		case MSpick:
		case MSplace:
			RGB_BreathingPattern(500, 255, 255, 255);
			laststate = MSpick;
			break;
		case MShome:
			RGB_BreathingPattern(500, 0, 0, 255);
			laststate = MShome;
			break;
		case MStray:
			if (laststate != MStray) {
				RGB_off();
			}
			RGB_TrayProgress(point);
			laststate = MStray;
			break;
		case MSpoint:
			laststate = MSpoint;
			RGB_BreathingPattern(500, 255, 165, 0);
			break;
		default:
			break;
		}
	}

//	for (int i = 16; i < 24; i++) {
//		Set_LED(i, 0, 0, 0);
//	}

	Set_Brightness(45);
	WS2812_Send();
}

void RGB_off() {
	for (int i = 0; i < MAX_LED; i++) {
		Set_LED(i, 0, 0, 0);
	}
}

void RGB_Rainbow() {
	static uint32_t startTime = 0;
	static const uint32_t transitionDuration = 5000; // Transition duration in milliseconds

	if (HAL_GetTick() - startTime >= transitionDuration) {
		startTime = HAL_GetTick();
	}

	// Calculate time elapsed in the current transition
	uint32_t elapsed = HAL_GetTick() - startTime;

	// Calculate the normalized progress (0.0 to 1.0) within the transition
	float progress = (float) elapsed / transitionDuration;

	// Calculate the hue angle based on the progress
	float hueAngle = 360.0f * progress;

	// Set LED colors based on the hue angle
	for (int i = 0; i < MAX_LED; i++) {
		// Calculate the hue value for the current LED
		float ledHue = hueAngle + (i * (360.0f / MAX_LED));

		// Convert hue to RGB using HSV color model
		float huePrime = fmodf(ledHue / 60.0f, 6.0f);
		float chroma = 1.0f;
		float x = chroma * (1.0f - fabsf(fmodf(huePrime, 2.0f) - 1.0f));

		float red, green, blue;

		if (huePrime >= 0.0f && huePrime < 1.0f) {
			red = chroma;
			green = x;
			blue = 0.0f;
		} else if (huePrime >= 1.0f && huePrime < 2.0f) {
			red = x;
			green = chroma;
			blue = 0.0f;
		} else if (huePrime >= 2.0f && huePrime < 3.0f) {
			red = 0.0f;
			green = chroma;
			blue = x;
		} else if (huePrime >= 3.0f && huePrime < 4.0f) {
			red = 0.0f;
			green = x;
			blue = chroma;
		} else if (huePrime >= 4.0f && huePrime < 5.0f) {
			red = x;
			green = 0.0f;
			blue = chroma;
		} else {
			red = chroma;
			green = 0.0f;
			blue = x;
		}

		// slow fade in
		if (LEDtime == 0) {
			LEDtime = HAL_GetTick();
		}

		float intensity;
		if (HAL_GetTick() - LEDtime < 4000) {
			intensity = (HAL_GetTick() - LEDtime) / 4000.0;
		} else {
			intensity = 1;
		}

		intensity = sqrt(intensity);

		// Scale RGB values to 0-255 range
		uint8_t r = (uint8_t) (red * 255.0 * intensity);
		uint8_t g = (uint8_t) (green * 255.0 * intensity);
		uint8_t b = (uint8_t) (blue * 255.0 * intensity);

		// Set LED color
		Set_LED(i, r, g, b);
	}
}

void RGB_Bootup(void) {
	for (int i = 0; i < 60; i++) {
		Set_LED(i, 255, 0, 0);
		HAL_Delay(10);
		Set_Brightness(45);
		WS2812_Send();
	}
}

void RGB_BreathingPattern(uint32_t period, uint8_t R, uint8_t G, uint8_t B) {
	if (LEDtime == 0) {
		LEDtime = HAL_GetTick();
	}

	uint32_t elapsedTime = HAL_GetTick() - LEDtime;
	float intensity;

	intensity = 0.5 * (1.0 + sinf((2.0 * PI * elapsedTime) / period));

	// slow fade in
	if (LEDtime == 0) {
		LEDtime = HAL_GetTick();
	}

	float intensity2;
	if (HAL_GetTick() - LEDtime < 4000) {
		intensity2 = (HAL_GetTick() - LEDtime) / 4000.0;
	} else {
		intensity2 = 1;
	}

	intensity2 = sqrt(intensity2);

	for (int i = 0; i < MAX_LED; i++) {
		Set_LED(i, R * intensity * intensity2, G * intensity * intensity2, B * intensity * intensity2);
	}
}

void RGB_TrayProgress(uint8_t point) {
	float percentage = (point + 1.0) / 9.0;

	for (int i = 0; i < (int) (60.0 * percentage); i++) {
		Set_LED(i, 0, 255, 0);
	}
}

#endif /* INC_RGB_H_ */
