# Timer Cheat Sheet
By Nopparuj
<br/><br/>


## Read timer count
#### User code begin 2
```C
// Start timer in basic mode
HAL_TIM_Base_Start(&htim2);
```


#### While loop
```C
// Get counter timestamp
__HAL_TIM_GET_COUNTER(&htim2);
```


## Timer Interrupts
Don't forget to enable `TIMX global interrupts` in `NVIC`

#### User code begin 2
```C
// Start timer in interrupt mode
HAL_TIM_Base_Start_IT(&htim3);
```

#### User code begin 4
```C
// Call back function
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim) {
	if (htim == &htim3) {
		HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
	}
}
```


## Calling ADC with timers
In ADC settings enable `Timer X Trigger Out event`

Enable `DMA Continuous Requests`

Add `Circular` DMA in DMA Settings
##### In Timer settings select `Update Event` in `TRGO/Trigger Event Selection`

#### USER CODE BEGIN PV
```C
// ADC DMA buffer
uint16_t ADCBuffer[10];
```

#### User code begin 2
```C
// Start timer in basic mode and ADC in DMA mode
HAL_ADC_Start_DMA(&hadc1, (uint32_t*) ADCBuffer, 10);
HAL_TIM_Base_Start(&htim3);
```


## Input Capture Mode
In timer settings, select `Input Capture direct mode`

Divide the clock using `prescaler` so you get the correct counting frequency

In `Input Capture Channel X`>`IC Selection` select `Direct` mode

Go to DMA settings and add `Circular` DMA

#### USER CODE BEGIN PV
```C
// ADC DMA buffer
#define IC_BUFFER_SIZE 20
uint32_t InputCaptureBuffer[IC_BUFFER_SIZE];
float averageRisingedgePeriod;
float frequency;
```
#### USER CODE BEGIN PFP
```C
float IC_Calc_Period();
```

#### User code begin 2
```C
HAL_TIM_Base_Start(&htim2);
HAL_TIM_IC_Start_DMA(&htim2, TIM_CHANNEL_1, InputCaptureBuffer, IC_BUFFER_SIZE);
```


#### User code begin 4
```C
// Calculate Average Period
float IC_Calc_Period() {
	uint32_t currentDMAPointer = IC_BUFFER_SIZE
			- __HAL_DMA_GET_COUNTER(htim2.hdma[1]);

	uint32_t lastValidDMAPointer = (currentDMAPointer - 1 + IC_BUFFER_SIZE)
			% IC_BUFFER_SIZE;

	uint32_t i = (lastValidDMAPointer + IC_BUFFER_SIZE - 5) % IC_BUFFER_SIZE;

	int32_t sumdiff = 0;
	while (i != lastValidDMAPointer) {
		uint32_t firstCapture = InputCaptureBuffer[i];

		uint32_t NextCapture = InputCaptureBuffer[(i + 1) % IC_BUFFER_SIZE];
		sumdiff += NextCapture - firstCapture;
		i = (i + 1) % IC_BUFFER_SIZE;
	}

	return sumdiff / 5.0;
}
```

#### While loop
```C
static uint32_t timestamp = 0;
if (HAL_GetTick() >= timestamp) {
	timestamp = HAL_GetTick() + 5;
	averageRisingedgePeriod = IC_Calc_Period();
	frequency = (1.0 / averageRisingedgePeriod) * 12.0 * 1000.0; // 12 teeth, revolution per second
}
```


## Output Comparator
In timer settings, select `PWM Generation CHX`

Adjust `PSC` and `ARR` accordingly.

#### USER CODE BEGIN PV
```C
uint32_t duty = 500;
```

#### USER CODE BEGIN 2
```C
// Start timers in basic mode
HAL_TIM_Base_Start(&htim1);

// Start timer channel in PWM mode
HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_1);
```
#### While loop
```C
static uint32_t timestamp = 0;
if (HAL_GetTick() >= timestamp) {
	timestamp = HAL_GetTick() + 500;
	__HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_1, duty);
}
```


## QEI
In timer settings, select `Encoder Mode`

In `Encoder mode`, select `Encoder Mode T1 and T2`

#### USER CODE BEGIN PV
```C
uint32_t QEIReadRaw;
uint32_t degrees;
```

#### USER CODE BEGIN 2
```C
// Start timer in encoder mode
HAL_TIM_Encoder_Start(&htim2, TIM_CHANNEL_1 | TIM_CHANNEL_2);
```

#### While loop
```C
QEIReadRaw = __HAL_TIM_GET_COUNTER(&htim2);
degrees = QEIReadRaw * 360.0 / 3072.0;
```


## Printf
Go to `System Core`>`SYS` and select Debug `Trace Asynchronous Sw`

In `Debug Configurations` enable ST-Link and `scan`

Enable `Serial Wire Viewer` and input `Core Clock`

#### USER CODE BEGIN 4
```C
int _write(int file, char *ptr, int len) {
	int i;

	for (i = 0; i < len; i++) {
		ITM_SendChar(*ptr++);
	}
	return len;
}
```

#### While loop
```C
printf("Position = %d\n", QEIReadRaw);
```

### Usage
In `SWV ITM Data Console` check box `0` on `ITM Stimulus Ports`


## Bonus: Calculating angular velocity with QEI
Set a timer to count at 1MHz (for 1 microsecond resolution)

Enable interrupts in NVIC

#### USER CODE BEGIN PV
```C
uint32_t QEIReadRaw;
uint32_t degrees;

typedef struct _QEIStructure {
	uint32_t data[2];
	uint64_t timestamp[2];

	float QEIPosition;
	float QEIVelocity;
} QEIStructureTypedef;

QEIStructureTypedef QEIData = { 0 };

uint64_t _micros = 0;
```

#### USER CODE BEGIN PFP
```C
inline uint64_t micros();
void QEIEncoderPositionVelocity_Update();
```
#### USER CODE BEGIN 2
```C
HAL_TIM_Encoder_Start(&htim2, TIM_CHANNEL_1 | TIM_CHANNEL_2);
HAL_TIM_Base_Start(&htim5);
```

#### USER CODE BEGIN 4
```C
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim) {
	if (htim = &htim5) {
		_micros += UINT32_MAX;
	}
}

uint64_t micros() {
	return __HAL_TIM_GET_COUNTER(&htim5) + _micros;
}

void QEIEncoderPositionVelocity_Update(){
	// collect data
	QEIData.timestamp[0] = micros();
	uint32_t counterPosition = __HAL_TIM_GET_COUNTER(&htim2);
	QEIData.data[0] = counterPosition;

	// calculation
	QEIData.QEIPosition = counterPosition % 3072;

	int32_t diffPosition = QEIData.data[0] - QEIData.data[1];
	float difftime = (QEIData.timestamp[0] - QEIData.timestamp[1]);

	// wrapping
	if(diffPosition > QEI_PERIOD>>1) diffPosition -= QEI_PERIOD;
	if(diffPosition < -(QEI_PERIOD>>1)) diffPosition += QEI_PERIOD;

	// calculate speed
	QEIData.QEIVelocity = (diffPosition * 1000000) / difftime;

	QEIData.data[1] = QEIData.data[0];
	QEIData.timestamp[1] = QEIData.timestamp[0];
}
```

#### While loop
```C
static uint64_t timestamp2 = 0;
int64_t currentTime = micros();
if(currentTime > timestamp2){
	timestamp2 = currentTime + 100000;
	QEIEncoderPositionVelocity_Update();
}
```


## DSP

#### Creating a Matrix
```C
float32_t A_f32[16] = {
	1,  2,  3,  4,
	5,  6,  7,  8,
	9,  10, 11, 12,
	13, 14, 15, 16
};

arm_matrix_instance_f32 A;
arm_mat_init_f32(&A, 4, 4, (float32_t*) &A_f32);

float32_t B_f32[16];
arm_matrix_instance_f32 B;
arm_mat_init_f32(&B, 4, 4, (float32_t*) &B_f32);
```

#### Matrix Operation
Go to `Project Icon`>`ProPerties`>`Settings`>`MCU GCC Assembler`>`Preprocessor`>`Add Icon`
```
ARM_MATH_MATRIX_CHECK
```
```C
volatile arm_status matstatus;
matstatus = arm_mat_trans_f32(&A, &At);
matstatus = arm_mat_mult_f32(&A, &B, &AmB);
matstatus = arm_mat_add_f32(&A, &B, &AaB);
````


## PID Controller

#### USER CODE BEGIN PV
```C
arm_pid_instance_f32 PID = { 0 };
float position = 0;
float setposition = 0;
float Vfeedback = 0;
```

#### USER CODE BEGIN 2
```C
PID.Kp = 0.1;
PID.Ki = 0.00001;
PID.Kd = 0.1;
arm_pid_init_f32(&PID, 0);
```

#### While loop
```C
static uint32_t timestamp = 0;
if (timestamp < HAL_GetTick()) {
	timestamp = HAL_GetTick() + 10;

	Vfeedback = arm_pid_f32(&PID, setposition - position);
	position = PlantSimulation(Vfeedback);
}
```

#### User code begin 4
```C
float PlantSimulation(float Vin) {
	static float speed = 0;
	static float position = 0;
	float current = Vin - speed * 0.0123;
	float torque = current * 0.456;
	float acc = torque * 0.789;
	speed += acc;
	position += speed;
	return position;
}
```


## Debug graph
Go to `System Core`>`SYS` and select Debug `Trace Asynchronous Sw`

In `Debug Configurations` enable ST-Link and `scan`

Enable `Serial Wire Viewer` and input `Core Clock`

#### SWV Data Trace Timeline
In `Settings`>`Comparator 0` Check box to enable

Input name of the variable and select `Access` to `Write`