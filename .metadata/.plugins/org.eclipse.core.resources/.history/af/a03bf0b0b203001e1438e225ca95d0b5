#ifndef INC_RGB_H_
#define INC_RGB_H_

// PRIVATE INCLUDE ================================================================================

#include <WS2812B.h>

// PRIVATE FUNCTION PROTOTYPE =====================================================================

uint32_t LERP(uint32_t start, uint32_t end, float t);
void RGB_Rainbow(void);

// USER CODE ======================================================================================

void RGB_Rainbow(void) {
	static uint32_t previousTime = 0;
	static uint8_t hue = 0;

	uint32_t currentTime = HAL_GetTick();
	uint32_t elapsedTime = currentTime - previousTime;

	if (elapsedTime >= 50) {
		previousTime = currentTime;

		// Update hue
		hue++;
		if (hue > 255) {
			hue = 0;
		}

		// Calculate color gradient
		uint32_t colorStart = hue;
		uint32_t colorEnd = (hue + 85) % 255;

		// Set LED colors based on color gradient
		for (int i = 0; i < MAX_LED; i++) {
			uint32_t color = LERP(colorStart, colorEnd, (float) i / MAX_LED);
			Set_LED(i, color, 255 - color, 0);
		}

		// Send LED data to update colors
		WS2812_Send();
		Set_Brightness(5);
	}
}

uint32_t LERP(uint32_t start, uint32_t end, float t) {
	return (uint32_t) ((1 - t) * start + t * end);
}

#endif /* INC_RGB_H_ */
