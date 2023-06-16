/*
 * user_function.c
 *
 *  Created on: Feb 12, 2023
 *      Author: 08809
 */

#include <LowpassFilter.h>

// USER DEFINE FUNCTION BEGIN

float ComputeLowpassConstant(uint16_t CutoffFreq, uint16_t SamplingFreq) {
	return CutoffFreq / ((float) (CutoffFreq + SamplingFreq));
}

//float C1 = ComputeLowpassConstant(20, 10000);
//float C2 = ComputeLowpassConstant(10, 10000);

void lowpass_filter(int QEIReadRaw_now, float *velocity_measure_filter_now, float *acceleration_measure_filter_now){
    static int QEIReadRaw_past = 0;
    static float velocity_measure_filter_past = 0;
    float dt = 0.0001;

    float C1 = 20.0 / (20.0 + 10000.0);
    float C2 = 10.0 / (10.0 + 10000.0);

    float velocity_measure = (QEIReadRaw_now - QEIReadRaw_past) / dt;
    *velocity_measure_filter_now = (C1 * velocity_measure) + ((1 - C1) * (*velocity_measure_filter_now));

    float acceleration_measure = (*velocity_measure_filter_now - velocity_measure_filter_past) / dt;
    *acceleration_measure_filter_now = (C2 * acceleration_measure) + ((1 - C2) * (*acceleration_measure_filter_now));

    velocity_measure_filter_past = *velocity_measure_filter_now;

    QEIReadRaw_past = QEIReadRaw_now;
}
// USER DEFINE FUNCTION END
