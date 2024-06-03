---
layout: post
title: RPM filter implementation
permalink: /docs/drone/filters/RPM_filter_impl
nav_order: 2
parent: Filters
grand_parent: Drone
---

<!-- comment or image allows {: .no_toc} to work correctly  (don't ask me why) -->

{: .no_toc}

# RPM Filter Implementation

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

# Intro

After implementing [BDshot](../ESC/ESC_prot_impl_2_2) and [notch filters](filters_impl#biquad_filters) (biquad version), it is time to combine them for the ultimate filter for drones. It doesn't mean that you can forget about all other filters but this is a really great solution that makes filtering much easier.

As we know, the main source of the noise in measurements of the gyro or accelerometer comes from motors and propellers. Since we know the RPM of each motor (thanks BDshot) we know the main frequencies of the noise in real time. Therefore we can apply very precisely notch filters to cut out those frequencies and thus obtain de-noised measurements with close-to-zero delay.

## Structure

The basic idea is to calculate coefficients for a set of notch filters each time the new RPMs are available and apply them for current measurements (gyro or accelerometer). Thus we need a notch filter for each motor (`MOTORS_COUNT = 4` in my case) for each measurement (each axis needs separate 4 filters). That gives us 12 notch filters in total to filter gyro measurements.

However, besides the basic frequencies of the motors we need to take care of the 1st and 2nd harmonics that are typically present in our measurements. They are caused by n-blade propellers that interact with frame n-time more often than motors themselves (more about it [here](../../math/vibrations/vibrations_theory#harmonics)). Bearing this in mind we need `MOTORS_COUNT * RPM_MAX_HARMONICS` notch filters for each measurement (12 for X/Y/Z gyro axis so in total 36 notch filters).

```c
// filters.h
#define MOTORS_COUNT 4
#define RPM_MAX_HARMONICS 3

typedef struct{
	biquad_Filter_t notch_filters[MOTORS_COUNT][RPM_MAX_HARMONICS];  // each axes (X,Y,Z) for each motor and its harmonics
	float weight[MOTORS_COUNT][RPM_MAX_HARMONICS];   // weight used to fade out filter (0 - filter is off, 1 - is used in 100%)
	float q_factor;                         // q_factor for all notches
	uint8_t harmonics;                      // number of filtered harmonics
	uint16_t frequency;	                // frequency that filter is running
	uint16_t max_filtered_frequency;	// maximal frequency that can be filtered (Nyquist frequency ~0.5*frequency)
} RPM_filter_t;
```

## Initialization

Basically, we need to initialize all notch filters and set their variables. We need to decide about the Q-factor of the notch filters - `RPM_Q_FACTOR`. Since we know very well central frequency we can use high values (like 300-1000). The `sampling frequency_Hz` needs to be provided to match your system filtering frequency (you need to take care of the timing externally).

```c
// filters.h
void RPM_filter_init(RPM_filter_t* filter, uint16_t sampling_frequency_Hz);

#define RPM_Q_FACTOR 400

// filters.c
void RPM_filter_init(RPM_filter_t* filter, uint16_t sampling_frequency_Hz){
	filter->harmonics = RPM_MAX_HARMONICS;
	filter->q_factor = RPM_Q_FACTOR;
	filter->frequency = sampling_frequency_Hz;
	filter->max_filtered_frequency = 0.48 * sampling_frequency_Hz;
	const float default_freq = 100; // only for initialization doesn't really matter
	// initialize notch filters:
	for (uint8_t motor = 0; motor < MOTORS_COUNT; motor++){
		for (uint8_t harmonic = 0; harmonic < RPM_MAX_HARMONICS; harmonic++){
			biquad_filter_init(&(filter->notch_filters[motor][harmonic]), BIQUAD_NOTCH, default_freq, filter->q_factor, sampling_frequency_Hz);
			filter->weight[motor][harmonic] = 1;
		}
	}
}
```

## Filtering

This is when the real magic happens. Let's start with a higher level - we need to update notch filters' coefficients and then apply filtering to the new measurements (`gyro_raw[]`):

```c
	// update coefficients:
	RPM_filter_update(&rpm_filter_gyro_X, motors_rpm);
	RPM_filter_copy_coefficients(&rpm_filter_gyro_X, &rpm_filter_gyro_Y);
	RPM_filter_copy_coefficients(&rpm_filter_gyro_X, &rpm_filter_gyro_Z);
	// next apply rpm filtering:
	gyro_filtered[0] = RPM_filter_apply(&rpm_filter_gyro_X, gyro_raw[0]);
	gyro_filtered[1] = RPM_filter_apply(&rpm_filter_gyro_Y, gyro_raw[1]);
	gyro_filtered[2] = RPM_filter_apply(&rpm_filter_gyro_Z, gyro_raw[2]);
```

Now, let's dive into the `RPM_filter_update` function. We iterate through all filters and update coefficients for new frequencies (`biquad_filter_update`). Since for all axes (gyro X, Y, Z) coefficients for notch filters are the same there is no reason to calculate them separately so they are just copied with `RPM_filter_copy_coefficients` and `biquad_filter_copy_coefficients` functions. Since we don't want to filter frequencies lower than `100\ [Hz]` we add `RPM_MIN_FREQUENCY_HZ` and for smoother operation, we add `RPM_FADE_RANGE_HZ` which will fade out the RPM filter effect while approaching `RPM_MIN_FREQUENCY_HZ`. Array `motors_rpm[]` contains the current RPMs of the motors and you need to provide these values with good BDshot protocol handling.

```c
// filters.h
void RPM_filter_update(RPM_filter_t* filter, uint32_t* motors_rpm);
void RPM_filter_copy_coefficients(RPM_filter_t* copy_from_filter, RPM_filter_t* copy_to_filter);
void biquad_filter_copy_coefficients(biquad_Filter_t* copy_from_filter, biquad_Filter_t* copy_to_filter);

#define RPM_MIN_FREQUENCY_HZ 100 // all frequencies <= than this value will not be filtered by RPM filter
#define RPM_FADE_RANGE_HZ 50	// fade out notch when approaching RPM_MIN_FREQUENCY_HZ (turn it off for RPM_MIN_FREQUENCY_HZ)

// filters.c
void RPM_filter_update(RPM_filter_t* filter){
	const uint8_t sec_in_min = 60; // for conversion from Hz to rpm
	float frequency;			   // frequency for filtering
	// each motor introduces its own frequency (with harmonics) but for every axes noises are the same;
	for (uint8_t motor = 0; motor < MOTORS_COUNT; motor++){
		for (uint8_t harmonic = 0; harmonic < RPM_MAX_HARMONICS; harmonic++){
			frequency = (float)motors_rpm[motor] * (harmonic + 1) / sec_in_min;
			if (frequency > RPM_MIN_FREQUENCY_HZ){
				if (frequency < filter->max_filtered_frequency){
					// each axis has the same noises from motors, so compute it once and next copy values:
					biquad_filter_update(&(filter->notch_filters[motor][harmonic]), BIQUAD_NOTCH, frequency, filter->q_factor, filter->frequency);
					// fade out if reaching minimal frequency:
					if (frequency < (RPM_MIN_FREQUENCY_HZ + RPM_FADE_RANGE_HZ)){
						filter->weight[motor][harmonic] = (float)(frequency - RPM_MIN_FREQUENCY_HZ) / (RPM_FADE_RANGE_HZ);
					}
					else{
						filter->weight[motor][harmonic] = 1;
					}
				}
				else{
					filter->weight[motor][harmonic] = 0;
				}
			}
			else{
				filter->weight[motor][harmonic] = 0;
			}
		}
	}
}

// simple function to copy RPM filter coefficients
void RPM_filter_copy_coefficients(RPM_filter_t* copy_from_filter, RPM_filter_t* copy_to_filter){
	for (uint8_t motor = 0; motor < MOTORS_COUNT; motor++){
		for (uint8_t harmonic = 0; harmonic < RPM_MAX_HARMONICS; harmonic++){
			biquad_filter_copy_coefficients(&(copy_from_filter->notch_filters[motor][harmonic]), &(copy_to_filter->notch_filters[motor][harmonic]));
			copy_to_filter->weight[motor][harmonic] = copy_from_filter->weight[motor][harmonic];
		}
	}
}

// simple function to copy biquad filter coefficients
void biquad_filter_copy_coefficients(biquad_Filter_t* copy_from_filter, biquad_Filter_t* copy_to_filter)
{
	copy_to_filter->a0 = copy_from_filter->a0;
	copy_to_filter->a1 = copy_from_filter->a1;
	copy_to_filter->a2 = copy_from_filter->a2;

	copy_to_filter->b0 = copy_from_filter->b0;
	copy_to_filter->b1 = copy_from_filter->b1;
	copy_to_filter->b2 = copy_from_filter->b2;
}
```

After updating the notch filters' coefficients we can apply them for new measurements. Besides the well-known function `biquad_filter_apply_DF1` (you can use `biquad_filter_apply_DF2`), we use weights to decrease impact or turn off the RPM filter if the frequency of the notch filter is beyond the useful range.

```c
// filters.h
float RPM_filter_apply(RPM_filter_t* filter, float input);

// filters.c
float RPM_filter_apply(RPM_filter_t* filter, float input){
	float result = input;
	// Iterate over all notches on axis and apply each one to value.
	// Order of application doesn't matter because biquads are linear time-invariant filters.
	for (uint8_t motor = 0; motor < MOTORS_COUNT; motor++){
		for (uint8_t harmonic = 0; harmonic < RPM_MAX_HARMONICS; harmonic++){
			result = filter->weight[motor][harmonic] * biquad_filter_apply_DF1(&(filter->notch_filters[motor][harmonic]), result) + (1 - filter->weight[motor][harmonic]) * result;
		}
	}
	return result;
}
```

## Cleanup

We use biquad filters implementation which doesn't need any memory freeing.

## Example

```c
// main.c
#include"filters.h"
// ...
int main(){
	// setup
	// ...
	// define all variables and constants:
	#define FREQUENCY_MAIN_LOOP 2000 // [Hz]
	RPM_filter_t rpm_filter_gyro_X;
	RPM_filter_t rpm_filter_gyro_Y;
	RPM_filter_t rpm_filter_gyro_Z;
	// initialize filter:
	RPM_filter_init(&rpm_filter_gyro_X, FREQUENCY_MAIN_LOOP);
	RPM_filter_init(&rpm_filter_gyro_Y, FREQUENCY_MAIN_LOOP);
	RPM_filter_init(&rpm_filter_gyro_Z, FREQUENCY_MAIN_LOOP);
	while(true){
		float gyro_raw[3]; // variable for raw measurement
		uint32_t motors_rpm[MOTORS_COUNT]; // motor's RPM values (from BDshot)
		// perform filtering (you need to provide new values (gyro_raw and motors_rpm) and take care of timing (to match declared values))
		float gyro_filtered[3];
		// update coefficients:
		RPM_filter_update(&rpm_filter_gyro_X, motors_rpm);
		RPM_filter_copy_coefficients(&rpm_filter_gyro_X, &rpm_filter_gyro_Y);
		RPM_filter_copy_coefficients(&rpm_filter_gyro_X, &rpm_filter_gyro_Z);
		// next apply rpm filtering:
		gyro_filtered[0] = RPM_filter_apply(&rpm_filter_gyro_X, gyro_raw[0]);
		gyro_filtered[1] = RPM_filter_apply(&rpm_filter_gyro_Y, gyro_raw[1]);
		gyro_filtered[2] = RPM_filter_apply(&rpm_filter_gyro_Z, gyro_raw[2]);
	}
	return 0;
}

```

# Summary

In this post, we've implemented RPM filtering. I hope It wasn't too complicated and you managed to add this feature to your project.
