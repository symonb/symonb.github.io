---
layout: default
title: Basic filters implementation
permalink: /docs/drone/filters/filters_impl
nav_order: 1
parent: Filters
grand_parent: Drone
toc: true
---

<!-- comment or image allows {: .no_toc} to work correctly  (don't ask me why) -->

{: .no_toc}

# Basic Filters Implementation in C

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

# Intro

After a deep dive into filters theory in the previous [post](../../math/filters/filters_theory). It is time to transform the gained knowledge into real code. I will present the implementation for the [FIR](#fir_filters), and [IIR](#iir_filters) filters in a general form that allows using them for any application and exploring filter designing. Moreover, a simpler version of [biquad](#biquad_filters) filters will be presented for more real-time applications. There will be a lot of code but most of it is pretty straightforward (if you've read theory) and easy to use for your application.

# General structure

All filter types will have a similar structure with coefficients arrays and buffers for inputs or outputs. Also, for every type, there will be defined special functions to interact with objects. In general, we can divide those functions into 3 categories:

- Initialization - allocating memory and calculating all coefficients of the filters,
- Filtering/Applying - applying a filer in the main program (actually filtering a signal),
- Cleanup - freeing memory and resetting parameters in case of deactivation of the filter.

Moreover, we will define a function for updating filters' structure which is essential for some filtering (moving Notch filter or [RPM filter](RPM_filter.md)).

# FIR Filters {#fir_filters}

Let's start with the simplest one. We need to define a structure combining coefficients and inputs but we don't want to decide about the order of the filter in advance. Let's create a variable for the order of the filter `lenght` and pointers for future array addresses `* buffer` and `* impulse_response`. Also, let's save the last output filtered value to have quick access to that `output`. The structure below allows us to dynamically allocate memory so it will be suitable for filters of any length. Because we will be using a circular buffer we need `buffer_index` to keep track of the last measurement.

## Structure

```c
// filters.h
typedef struct{
	uint8_t length;             //	how many previous samples will be taken into calculation
	float* buffer;              //	buffer for input measurements
	uint8_t buffer_index;       //	variable to control position of the latest measurement
	float* impulse_response;    //	coefficients for inputs values
	float output;               //	final output value
} FIR_Filter_t;
```

## Initialization

Next, we need to initialize our filter and allocate memory for its arrays:

```c
// filters.h
void FIR_filter_init(FIR_Filter_t* fir, uint8_t order);

// filters.c
void FIR_filter_init(FIR_Filter_t* fir, uint8_t order){
	fir->length = order;
	//	Allocate memory for arrays
	fir->buffer = (float*)malloc(sizeof(*(fir->buffer)) * fir->length);
	fir->impulse_response = (float*)malloc(
		sizeof(*(fir->impulse_response)) * fir->length);
	//	Clear filter buffer
	for (uint8_t i = 0; i < order; i++){
		fir->buffer[i] = 0.f;
		fir->impulse_response[i] = 0.f;
	}
	//	Clear buffer index
	fir->buffer_index = 0;
	//	Clear filter output
	fir->output = 0.0f;
}
```

## Filtering

After that filter is ready to use but its coefficients (`impulse_response[]`) are all set to $0$. Therefore we need to update them manually one by one. An example is shown at the [end](#fir_example). After updating coefficients we can perform a filtering process. We need to calculate a convolution - basically, we multiply all the last measurements with corresponding coefficients (bearing in mind the circular buffer implementation).

```c
// filters.h
float FIR_filter_apply(FIR_Filter_t* fir, float input)

// filters.c
float FIR_filter_apply(FIR_Filter_t* fir, float input){
	//	Add new data to buffer
	fir->buffer[fir->buffer_index] = input;
	//	Increment buffer_index and loop if necessary
	fir->buffer_index++;
	if (fir->buffer_index == fir->length){
		fir->buffer_index = 0;
	}
	//	Reset old output value
	fir->output = 0.f;
	uint8_t sum_index = fir->buffer_index;
	//	Decrement sum_index and loop if necessary
	for (uint8_t i = 0; i < fir->length; i++){
		if (sum_index > 0){
			sum_index--;
		}
		else{
			sum_index = fir->length - 1;
		}
		//	Compute new output (via convolution)
		fir->output += fir->impulse_response[i] * fir->buffer[sum_index];
	}
	return fir->output;
}
```

## Cleanup

For drone applications, we will set up and initialize filters at the beginning and use them as long as the drone program is running (so we don't have to bother with freeing allocated memory). But you may need filters only for some time and then they can be discarded. In such a case you need to remember to free the memory allocated during the initialization. Moreover, this is good practice anyway - to write a function freeing memory every time you dynamically allocate memory.

```c
// filters.h
void FIR_filter_cleanup(FIR_Filter_t* fir);

// filters.c
void FIR_filter_cleanup(FIR_Filter_t* fir){
	// Free allocated memory
	free(fir->buffer);
	free(fir->impulse_response);
	// Reset length and buffer index
	fir->length = 0;
	fir->buffer_index = 0;
	// Reset output
	fir->output = 0.0f;
}
```

## Example {#fir_example}

Now it is time to show an example of how to use it in practice:

```c
// main.c
#include"filters.h"
// ...

int main(){
    // setup
    // ...
    // define all variables and constants:
    #define GYRO_FILTERS_ORDER 6
    const float GYRO_FILTER_X_COEF[GYRO_FILTERS_ORDER] = { 0.135250f, 0.216229f, 0.234301f, 0.234301f, 0.216229f, 0.135250f };
    FIR_Filter_t filter_gyro_X;
    // initialize filter:
    FIR_filter_init(&filter_gyro_X, GYRO_FILTERS_ORDER);
    // update coefficients:
    for (uint8_t i = 0; i < filter_gyro_X.length; i++){
        filter_gyro_X.impulse_response[i] = GYRO_FILTER_X_COEF[i];
    }
    while(true){
        // variable for raw measurement:
        float raw_input_x;
        // perform filtering (you need to provide new values and take care of timing)
        float filtered_value = FIR_filter_apply(&filter_gyro_X, raw_input_x);
        // you can get last filtered value from filter also:
        filtered_value = filter_gyro_X.output;
    }
    // frre memory
    FIR_filter_cleanup(&filter_gyro_X);
    return 0;
}
```

# IIR Filters {#iir_filters}

IIR filter is pretty similar to FIR but a bit extended since it needs inputs and outputs saved.

## Structure

```c
// filters.h
typedef struct{
	uint8_t filter_order;           //  how many previous samples will be taken into calculation
	float* buffer_input;            //  buffer for input measurements
	float* buffer_output;           //  buffer for output values from filter
	uint8_t buffer_index;           //  variable to control position of the latest measurement
	float* forward_coefficients;    //  coefficients for inputs values
	float* feedback_coefficients;   //  coefficients for previous outputs values
	float output;                   //  final output value
} IIR_Filter_t;
```

## Initialization

Initialization is similar but you need to update both coefficients' arrays (example at the [end](#iir_example)):

```c
// filters.h
void IIR_filter_init(IIR_Filter_t* iir, uint8_t order);
// filters.c

void IIR_filter_init(IIR_Filter_t* iir, uint8_t order){
	iir->filter_order = order;
	//	Allocate memory for arrays
	iir->buffer_input = (float*)malloc(
		sizeof(*(iir->buffer_input)) * (iir->filter_order + 1));
	iir->buffer_output = (float*)malloc(
		sizeof(*(iir->buffer_output)) * (iir->filter_order));
	iir->forward_coefficients = (float*)malloc(
		sizeof(*(iir->forward_coefficients)) * (iir->filter_order + 1));
	iir->feedback_coefficients = (float*)malloc(
		sizeof(*(iir->feedback_coefficients)) * (iir->filter_order));
	//	Clear filter buffers and coefficients
	for (uint8_t i = 0; i < (order + 1); i++){
		// FORWARD PART:
		iir->buffer_input[i] = 0.f;
		iir->forward_coefficients[i] = 0.f;
		// FEEDBACK PART:
		iir->buffer_output[i] = 0.f;
		iir->feedback_coefficients[i] = 0.f;
	}
	//	Clear buffer index
	iir->buffer_index = 0;
	//	Clear filter output
	iir->output = 0.0f;
}
```

## Filtering

```c
// filters.h
float IIR_filter_apply(IIR_Filter_t* iir, float input);

// filters.c
float IIR_filter_apply(IIR_Filter_t* iir, float input){
	//	Add new data to buffer_input:
	iir->buffer_input[iir->buffer_index] = input;
	//	Reset old output value:
	iir->output = 0.f;
	// save position of the latest sample:
	uint8_t sum_index = iir->buffer_index;
	//	FORWARD PART:
	//	Decrement sum_index and loop if necessary
	for (uint8_t i = 0; i < iir->filter_order + 1; i++){
		//	Compute forward part of a new output:
		iir->output += iir->forward_coefficients[i] * iir->buffer_input[sum_index];
		if (sum_index > 0){
			sum_index--;
		}
		else{
			sum_index = iir->filter_order;
		}
	}
	//	FEEDBACK PART:
	sum_index = iir->buffer_index;
	//	Decrement sum_index and loop if necessary
	for (uint8_t i = 0; i < iir->filter_order; i++){
		if (sum_index > 0){
			sum_index--;
		}
		else{
			sum_index = iir->filter_order - 1;
		}
		// Add feedback part of a new output
		iir->output -= iir->feedback_coefficients[i] * iir->buffer_output[sum_index];
	}
	//	Add new data to buffer_output
	iir->buffer_output[iir->buffer_index] = iir->output;
	//	Increment buffer_index and loop if necessary
	iir->buffer_index++;
	if (iir->buffer_index > iir->filter_order){
		iir->buffer_index = 0;
	}
	return iir->output;
}
```

## Cleanup

```c
// filters.h
void IIR_filter_cleanup(IIR_Filter_t* iir);
//filters.c
void IIR_filter_cleanup(IIR_Filter_t* iir){
	// Free allocated memory
	free(iir->buffer_input);
	free(iir->buffer_output);
	free(iir->forward_coefficients);
	free(iir->feedback_coefficients);
	// Reset length and buffer index
	iir->filter_order = 0;
	iir->buffer_index = 0;
	// Reset output
	iir->output = 0.0f;
}
```

## Example {#iir_example}

```c
// main.c
#include"filters.h"
// ...
int main(){
    // setup
    // ...
    // define all variables and constants:
    #define GYRO_FILTERS_ORDER 2
    float GYRO_FILTER_X_FORW_COEF[GYRO_FILTERS_ORDER + 1] = { 0.02008336f, 0.04016673f, 0.02008336f };
    const float GYRO_FILTER_X_BACK_COEF[GYRO_FILTERS_ORDER] = { -1.56101807f, 0.6413515f };
    IIR_Filter_t filter_gyro_X;
    // initialize filter:
    IIR_filter_init(&filter_gyro_X, GYRO_FILTERS_ORDER);
    // update coefficients:
	for (uint8_t i = 0; i < filter_gyro_X.filter_order + 1; i++){
		filter_gyro_X.forward_coefficients[i] = GYRO_FILTER_X_FORW_COEF[i];
	}
	for (uint8_t i = 0; i < filter_gyro_X.filter_order; i++){
		filter_gyro_X.feedback_coefficients[i] = GYRO_FILTER_X_BACK_COEF[i];
	}
    while(true){
        // variable for raw measurement:
        float raw_input_x;
        // perform filtering (you need to provide new values and take care of timing)
        float filtered_value = IIR_filter_apply(&filter_gyro_X, raw_input_x);
        // you can get last filtered value from filter also:
        filtered_value = filter_gyro_X.output;
    }
    // free memory
    IIR_filter_cleanup(&filter_gyro_X);
    return 0;
}
```

# Biquad Filters {#biquad_filters}

The above implementations allow you great flexibility and you can use them for any filter (low-pass, high-pass, band-pass, etc.). However, for real-time applications, we need something more computationally efficient and maybe less versatile. Moreover, often second-order filters are good enough for simple filtration (IIR filters especially) and at the same time, they are simple enough for convenient design. Biquad filters are just 2nd order IIR filters and we will implement a more condensed version of IIR filter functions for better performance (you can stick with the previous implementation but new ones will be simpler to use and in most cases, biquad filters are good enough anyway).

## Structure

Since we know all the required variables we can define them explicitly. Moreover, since we have a defined structure (for low-pass or high-pass etc.) we can easily calculate the required coefficients on the fly (with some knowledge about frequencies). Besides frequencies, we have one parameter to set - the Q-factor. Of course, you can define it as you want but for many scenarios, we can set $ Q=\frac{1}{\sqrt{2}}$ which corresponds 2nd order Butterworth (low-pass or high-pass):

```c
// filters.h
typedef struct{
	float frequency;    // cut-off/center frequency
	float Q_factor;	    // quality factor
	float a0;
	float a1;
	float a2;
	float b0;
	float b1;
	float b2;
	float x1;   //  last input
	float x2;   //	2nd last input
	float y1;   //	last output
	float y2;   //	2nd last output
} biquad_Filter_t;

typedef enum{
    BIQUAD_LPF,
    BIQUAD_HPF;
    BIQUAD_NOTCH,
    BIQUAD_BPF,
} biquad_Filter_type;

```

## Initialization

For initialization we don't need to allocate any memory since variables were defined explicitly but we need to calculate coefficients. In most cases, they will stay the same for the rest of the program so we only need to do it once. So function `biquad_filter_init` in reality is just a wrapper (with a small addition) for the function`biquad_filter_update` which handles all the calculations and updates coefficients.

```c
// filters.h
void biquad_filter_init(biquad_Filter_t* filter, biquad_Filter_type filter_type, float center_frequency_Hz, float quality_factor, uint16_t sampling_frequency_Hz);
static void biquad_filter_update(biquad_Filter_t* filter, biquad_Filter_type filter_type, float filter_frequency_Hz, float quality_factor, uint16_t sampling_frequency_Hz);

// filters.c
void biquad_filter_init(biquad_Filter_t* filter, biquad_Filter_type filter_type, float filter_frequency_Hz, float quality_factor, uint16_t sampling_frequency_Hz){
	biquad_filter_update(filter, filter_type, filter_frequency_Hz, quality_factor, sampling_frequency_Hz);
	// set previous values as 0:
	filter->x1 = 0;
	filter->x2 = 0;
	filter->y1 = 0;
	filter->y2 = 0;
}

static void biquad_filter_update(biquad_Filter_t* filter, biquad_Filter_type filter_type, float filter_frequency_Hz, float quality_factor, uint16_t sampling_frequency_Hz){
	// save filter info:
	filter->frequency = filter_frequency_Hz;
	filter->Q_factor = quality_factor;
	const float omega = 2.f * M_PI * filter_frequency_Hz / sampling_frequency_Hz;
	const float sn = sinf(omega);
	const float cs = cosf(omega);
	const float alpha = sn / (2.0f * filter->Q_factor);
	// implementation from datasheet:https://www.ti.com/lit/an/slaa447/slaa447.pdf <-- not everything good
	// or even better: http://shepazu.github.io/Audio-EQ-Cookbook/audio-eq-cookbook.html <-- probably resource for above
	/*
	general formula for 2nd order filter:
                   b0*z^-2 + b1*z^-1 + b2
	H(z^-1) = ------------------------
                   a0*z^-2 + a1*z^-1 + a2

	main analog prototypes:
	Low-pass:
                     1
	H(s) = ---------------
                s^2 + s/Q + 1
	High-pass:
                    s^2
	H(s) = ---------------
	        s^2 + s/Q + 1
	Notch:
	           s^2 + 1
	H(s) = ---------------
	        s^2 + s/Q + 1
	BPF (skirt gain where Q is max. gain):
	             s
	H(s) = ---------------
	        s^2 + s/Q + 1

	Q is a quality factor
	In this datasheet conversion from s -> z domain is done with Bilinear transform with pre-warping
	*/
	filter->a0 = 1 + alpha;
	switch (filter_type){
	case BIQUAD_LPF:
		filter->b0 = (1 - cs) * 0.5f;
		filter->b1 = 1 - cs;
		filter->b2 = filter->b0;
		filter->a1 = -2 * cs;
		filter->a2 = 1 - alpha;
		break;
	case BIQUAD_HPF:
		filter->b0 = (1 + cs) * 0.5f;
		filter->b1 = -(1 + cs);
		filter->b2 = (1 + cs) * 0.5f;
		filter->a1 = -2 * cs;
		filter->a2 = 1 - alpha;
		break;
	case BIQUAD_NOTCH:
		filter->b0 = 1;
		filter->b1 = -2 * cs;
		filter->b2 = 1;
		filter->a1 = filter->b1;
		filter->a2 = 1 - alpha;
		break;
	case BIQUAD_BPF:
		filter->b0 = alpha;
		filter->b1 = 0;
		filter->b2 = -alpha;
		filter->a1 = -2 * cs;
		filter->a2 = 1 - alpha;
		break;
	}
	// oust a0 coefficient:
	filter->b0 /= filter->a0;
	filter->b1 /= filter->a0;
	filter->b2 /= filter->a0;
	filter->a1 /= filter->a0;
	filter->a2 /= filter->a0;
}
```

## Filtering

Filtering is the same as before in terms of the function prototype but implementation allows for some improvements - check the _Implementation_ paragraph on [Wikipedia](https://en.wikipedia.org/wiki/Digital_biquad_filter). In reality, you can use either of these functions and for most cases, it will be good enough (although DF2 is a little better):

```c
// filters.h
float biquad_filter_apply_DF1(biquad_Filter_t* filter, float input);
float biquad_filter_apply_DF2(biquad_Filter_t* filter, float input);

// filters.c
float biquad_filter_apply_DF1(biquad_Filter_t* filter, float input){
	// direct form 1 (the most straighforward):
	const float result = filter->b0 * input + filter->b1 * filter->x1 + filter->b2 * filter->x2 - filter->a1 * filter->y1 - filter->a2 * filter->y2;
	/* shift x1 to x2, input to x1 */
	filter->x2 = filter->x1;
	filter->x1 = input;
	/* shift y1 to y2, result to y1 */
	filter->y2 = filter->y1;
	filter->y1 = result;
	return result;
}

float biquad_filter_apply_DF2(biquad_Filter_t* filter, float input){
	//	this is transposed direct form 2 is a little more precised for float number implementation:
	const float result = filter->b0 * input + filter->x1;
	filter->x1 = filter->b1 * input - filter->a1 * result + filter->x2;
	filter->x2 = filter->b2 * input - filter->a2 * result;
    //y1, y2, x1 and x2 are not saved in the filter structure x1 and x2 are used to save s1 and s2 (https://en.wikipedia.org/wiki/Digital_biquad_filter)
    // MOD: if you want you can use y1 and y2 variables to save last outputs:
    // filter->y2 = filter->y1;
    // filter->y1 = result;
	return result;
}
```

## Cleanup

Since we were not allocating any memory with `malloc` there is no need to free anything.

## Example

```c
// main.c
#include"filters.h"
// ...
int main(){
    // setup
    // ...
    // define all variables and constants:
    #define BIQUAD_LPF_CUTOFF_GYRO 90 // [Hz]
    #define BIQUAD_LPF_Q (1.f / sqrtf(2)) // Q factor for Low-pass filters
    #define FREQUENCY_MAIN_LOOP 2000 // [Hz]
    biquad_Filter_t filter_gyro_X;
    // initialize filter:
    biquad_filter_init(&filter_gyro_X, BIQUAD_LPF, BIQUAD_LPF_CUTOFF_GYRO, BIQUAD_LPF_Q, FREQUENCY_MAIN_LOOP);
    while(true){
        // variable for raw measurement:
        float raw_input_x;
        // perform filtering (you need to provide new values and take care of timing)
        float filtered_value = biquad_filter_apply_DF2(&filter_gyro_X, raw_input_x);
        // WARNING! you can get last filtered value from filter ONLY if you use biquad_filter_apply_DF1 or modified DF2:
        // filtered_value = filter_gyro_X.y1; // filter_gyro_X.y1 is not used currently
    }
    return 0;
}
```

# Summary

And that's it. Maybe there should be some screenshots before and after filtering but who cares really? It works, you can trust me or just check it yourself.

For my drone, I was using each of these filters (and even have the option to change between them in the code) but the most convenient are biquad filters since you don't have to calculate coefficients manually (you can calculate coefficients for IIR and FIR filter as well in code but there is no point if you will use 2nd order filters).

But the most important thing for a good filtration is to precisely apply notch filters and for that, we have [RPM filter](RPM_filter_impl)!
