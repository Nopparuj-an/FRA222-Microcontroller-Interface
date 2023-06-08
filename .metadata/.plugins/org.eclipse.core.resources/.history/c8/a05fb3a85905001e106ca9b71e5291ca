#ifndef INC_LOCALIZATION_H_
#define INC_LOCALIZATION_H_

// PRIVATE INCLUDE ================================================================================

#include <math.h>
#include <stdio.h>

// PRIVATE TYPEDEF ================================================================================

typedef struct {
	float x;
	float y;
} Coordinate;

// PRIVATE FUNCTION PROTOTYPE =====================================================================

Coordinate subtractCoordinates(Coordinate coord1, Coordinate coord2);
void localize(Coordinate *inputs, Coordinate *outputs, Coordinate *origin, float *angle);
float calculateNorm(Coordinate coord);
Coordinate negateCoordinate(Coordinate coord);

// USER CODE ======================================================================================

void localize(Coordinate *inputs, Coordinate *outputs, Coordinate *origin, float *angle) {
	Coordinate point1 = inputs[0];
	Coordinate point2 = inputs[1];
	Coordinate point3 = inputs[2];
	Coordinate vectA = subtractCoordinates(point2, point1);
	Coordinate vectB = subtractCoordinates(point3, point2);
	Coordinate vectC = subtractCoordinates(point3, point1);
	float lenA = calculateNorm(vectA);
	float lenB = calculateNorm(vectB);
	float lenC = calculateNorm(vectC);
	Coordinate vector1;
	Coordinate vector2;

	if (lenA < lenB && lenB < lenC && lenA < lenC) {
		// CASE 1
		*origin = point2;
		vector1 = vectB;
		vector2 = negateCoordinate(vectA);
	} else if (lenC < lenB && lenB < lenA && lenC < lenA) {
		// CASE 2
		*origin = point3;
		vector1 = negateCoordinate(vectB);
		vector2 = negateCoordinate(vectC);
	} else if (lenA < lenC && lenC < lenB && lenA < lenB) {
		// CASE 3
		*origin = point1;
		vector1 = vectC;
		vector2 = vectA;
	} else if (lenC < lenA && lenA < lenB && lenC < lenB) {
		// CASE 4
		*origin = point1;
		vector1 = vectA;
		vector2 = vectC;
	} else if (lenB < lenC && lenC < lenA && lenB < lenA) {
		// CASE 5
		*origin = point3;
		vector1 = negateCoordinate(vectC);
		vector2 = negateCoordinate(vectB);
	} else if (lenB < lenA && lenA < lenC && lenB < lenC) {
		// CASE 6
		*origin = point2;
		vector1 = negateCoordinate(vectA);
		vector2 = vectB;
	}

	// calculate vector angle
	if (vector1.x == 0) {
		if (vector1.y > 0) {
			*angle = M_PI / 2.0;
		} else {
			*angle = 1.5 * M_PI;
		}
	} else if (vector1.y == 0) {
		if (vector1.x >= 0) {
			*angle = 0;
		} else {
			*angle = M_PI;
		}
	} else {
		*angle = fabs(atan(vector1.y / vector1.x));
		if (vector1.x < 0 && vector1.y < 0) {
			*angle = M_PI + *angle;
		} else if (vector1.x < 0) {
			*angle = M_PI - *angle;
		} else if (vector1.y < 0) {
			*angle = 2.0 * M_PI - *angle;
		}
	}

	// calculate for matrix directions
	float dir = vector1.x * vector2.y - vector1.y * vector2.x;
	if (dir < 0) {
		*angle = *angle + M_PI;
		dir = -1.0;
	} else {
		dir = 1.0;
	}

	// create 9 points
	float X[] = { 10.0, 30.0, 50.0 };
	float Y[] = { 10.0, 25.0, 40.0 };
	for (int i = 0; i < 3; i++) {
		for (int j = 0; j < 3; j++) {
			double angleValue = (double) (*angle);  // Dereference and convert to double
			outputs[i * 3 + j].x = origin->x + X[i] * dir * cos(angleValue) - Y[j] * sin(angleValue);
			outputs[i * 3 + j].y = origin->y + X[i] * dir * sin(angleValue) + Y[j] * cos(angleValue);
		}
	}

	// dirty fix for angle error
	if (dir < 0){
		*angle = *angle - M_PI;
	}
}

Coordinate subtractCoordinates(Coordinate coord1, Coordinate coord2) {
	Coordinate result;
	result.x = coord1.x - coord2.x;
	result.y = coord1.y - coord2.y;
	return result;
}

float calculateNorm(Coordinate coord) {
	float norm = sqrt(coord.x * coord.x + coord.y * coord.y);
	return norm;
}

Coordinate negateCoordinate(Coordinate coord) {
	Coordinate result;
	result.x = -coord.x;
	result.y = -coord.y;
	return result;
}

// USER CODE END ==================================================================================

#endif /* INC_LOCALIZATION_H_ */
