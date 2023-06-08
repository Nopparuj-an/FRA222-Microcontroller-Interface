#ifndef INC_RGB_H_
#define INC_RGB_H_

// PRIVATE INCLUDE ================================================================================

#include <WS2812B.h>
#include <math.h>
#include <MainLogic.h>

// PRIVATE FUNCTION PROTOTYPE =====================================================================

void RGB_Rainbow();
void RGB_Power_Status();
void RGB_logic(MachineState state, uint8_t point);
void RGB_Bootup();

// USER CODE ======================================================================================

void RGB_logic(MachineState state, uint8_t point) {
	// Run on 200 Hz
	static uint32_t timestamp;
	if (HAL_GetTick() - timestamp < 5) {
		return;
	}
	timestamp = HAL_GetTick() + 5;

	// Main logic here
	RGB_Rainbow();

	for(int i = 16; i < 24; i++){
		Set_LED(i, 0, 0, 0);
	}

	WS2812_Send();
	Set_Brightness(45);
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

		// Scale RGB values to 0-255 range
		uint8_t r = (uint8_t) (red * 255);
		uint8_t g = (uint8_t) (green * 255);
		uint8_t b = (uint8_t) (blue * 255);

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

#endif /* INC_RGB_H_ */
