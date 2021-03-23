# Simple test and study how FMAC (Filter Math ACcelerator) works on 3p3z feedback using simple low pass RC circuit  

### Project files included  
NUCLEO-G474RE_RC_PWM_FMAC         = original  
NUCLEO-G474RE_RC_PWM_FMAC_iocOnly = troubleshooting, initially no feedback  
NUCLEO-G474RE_RC_PWM_FMAC_redo    = testing for consistensy and re-doing  
NUCLEO-G474RE_RC_PWM_FMAC_redo2   = troubleshooting, initially no feedback, testing and re-doing  
NUCLEO-G474RE_RC_PWM_FMAC_redo3   = variable output (to do)  
NUCLEO-G474RE_RC_PWM_FMAC_redo4   = more consistent documentations 

### Overview  
Use FMAC as 3p3z feedback and RC circuit to filter the PWM duty of PWM output as a feedback.
Then there is no need for actual converter to test the ff:
1. functionality of 3p3z
2. the code generated if there is a feedback happening


## Software setup   

### GPIO Default pins 

* GPIOA5 = LED  
* GPIOC11 is used for troubleshooting  

### HRTIM TimE will produce output for PWM and trigger ADC1-14

* GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
* pTimeBaseCfg.Period = 27200;
* pTimeBaseCfg.PrescalerRatio = HRTIM_PRESCALERRATIO_MUL8;
* pCompareCfg.CompareValue = 8, if less than 8, output will be always high
* Set Active source = Period
* Set Reset source = CMP1
* ADC trigger1 on TimE Period 
* Setup TimE in main.c last section  

	```
	if(HAL_OK != HAL_HRTIM_WaveformOutputStart(&hhrtim1, HRTIM_OUTPUT_TE1))
	{
		Error_Handler();
	}

	if(HAL_OK != HAL_HRTIM_WaveformCounterStart(&hhrtim1, HRTIM_TIMERID_TIMER_E))
	{
		Error_Handler();
	}
	```
	
### ADC triggered by HRTIM TimE Period and CMP1    

* Setup 1 Regular Conversion mode   
	* Set External trigger by HRTIM trig 1 event
* Enable DMA
	* Mode = Circular on Memory
	* Data Width = Word
* Go back ADC_Setting
	* DMA Continous Request = Enabled because of DMA
* User Constant
	* Add "Vtarget", and target value
* Offset
	* add 1 offset
	* Offset = Vtarget
		* target value for FMAC, also will be added/subtracted to the ADC value
	* Offset Sign = Negative
		* Negative = will subtract to the value of ADC Data
		* Positive = will add to the value of ADC Data
* Enable NVIC DMA Global IT with Call HAL handler, for debugging testing the value
* Add callback for Regular Conversion mode for debugging

	```
	HAL_ADC_ConvCpltCallback(ADC_HandleTypeDef *hadc)
	{
	  /* Prevent unused argument(s) compilation warning */
	  UNUSED(hadc);

	  /* NOTE : This function should not be modified. When the callback is needed,
				function HAL_ADC_ConvCpltCallback must be implemented in the user file.
	   */

	//  HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, SET);
	  HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_11);
	}
	```

* Enable ADC Calibration in main.c 

	```
	/* Run the ADC calibration in single-ended mode */
	if (HAL_ADCEx_Calibration_Start(&hadc1, ADC_SINGLE_ENDED) != HAL_OK)
	{
		/* Calibration Error */
		Error_Handler();
	}
	```

* Enable ADC DMAmain.c, use temporary variable _adc_data_ to check if triggered  
	
	```
	/*##- Enable Regular ADC DMA Channel ##############################*/
	if (HAL_ADC_Start_DMA(&hadc1, adc_data, 1) != HAL_OK)
	{
		/* ADC conversion start error */
		Error_Handler();
	}
	```

### FMAC Setup  

* enable FMAC  
* enable NVIC IT
* disable NVIC IT call handler
* use FMAC_Buck_VoltageMode_HW_AN5305 as guide
* copy main.h all private defines /* USER CODE BEGIN Private defines */
* copy main.c all private variables /* USER CODE BEGIN PV */
* copy main.c all FMAC Configure peripheral /* USER CODE BEGIN 2 */ before ADC config
* add extern void VT_FMAC_Controller(void); in estm32g4xx_it.c
* call VT_FMAC_Controller(void) from FMAC IT
* add volatile void VT_FMAC_Controller(void) in main.c 

	```
	volatile void VT_FMAC_Controller(void)
	{
		uint32_t tmp = READ_REG(hfmac.Instance->RDATA);
		tmp = (tmp > 0x00007FFF ? 0 : tmp);
		
		// Period is 27200, limit max duty to 27000
		if (tmp > 27200)
			tmp = 27000;
		if (tmp <= 10)
			tmp = 10;

		__HAL_HRTIM_SETCOMPARE( &hhrtim1, HRTIM_TIMERINDEX_TIMER_E, HRTIM_COMPAREUNIT_1, tmp);
	}
	```
	
* Update ADC DMA 

	```
	/* Start ADC and DMA */
	Fmac_Wdata = (uint32_t *) FMAC_WDATA;
	HAL_ADC_Start_DMA(&hadc1,Fmac_Wdata,1);
	```

## Harware setup

As shown, the output of the PWM is fed to the RC circuit which acts as low pass filter.
The filtered voltage is fed back to the ADC and then to the FMAC 3p3z internally.
The FMAC will calculate the PWM duty according to the set voltage and accordingly will change the PWM duty.

### RC circuit  
![](https://github.com/VictorTagayun/NUCLEO-G474RE_RC_PWM_FMAC/blob/main/waveforms%26pixx(NUCLEO-G474RE_RC_PWM_FMAC)/RC-ckt.png)

Ch1 = PWM Output  
Ch2 = Filtered signal and fed to the ADC  

![RC circuit](https://github.com/VictorTagayun/NUCLEO-G474RE_RC_PWM_FMAC/blob/main/waveforms%26pixx(NUCLEO-G474RE_RC_PWM_FMAC)/IMG_2066.JPG)

### Voltage risetime and then went to close loop  
Ch1 = PWM Output  
Ch2 = Filtered signal and fed to the ADC  
![Vout](https://github.com/VictorTagayun/NUCLEO-G474RE_RC_PWM_FMAC/blob/main/waveforms%26pixx(NUCLEO-G474RE_RC_PWM_FMAC)/DS1Z_QuickPrint40.jpg)

### Zoom in  
![Zoom in](https://github.com/VictorTagayun/NUCLEO-G474RE_RC_PWM_FMAC/blob/main/waveforms%26pixx(NUCLEO-G474RE_RC_PWM_FMAC)/DS1Z_QuickPrint39.jpg)


### Other related topics : 

[Test and study how FMAC (Filter Math ACcelerator) works](https://github.com/VictorTagayun/NUCLEO-G474RE_FMAC_Study_and_Analysis)  

[Use FMAC for Realtime FIR Low/High Pass filter (IIR filter will be on future Project)](https://github.com/VictorTagayun/NUCLEO-G474RE_RealTime_FIR_IIR_FMAC)

[Use PI/PID for feedbac control](https://github.com/VictorTagayun/NUCLEO-G474RE_RC_PWM_FMAC)

[DSP Software Study and Analysis](https://github.com/VictorTagayun/NUCLEO-G474RE_CMSIS_DSP_Tutorial)  


### Social Media : 

[LinkedIn](https://www.linkedin.com/posts/victortagayun_weekendhobbyabrelectronics-funwithelectronics-activity-6753952834942844928-eF8g/)

*Disclaimer:*
[Updated Disclaimer](https://github.com/VictorTagayun/GlobalDisclaimer)

*The projects posted here are for my Personal reference, learning and educational purposes only.*
*The purpose of a certain project may be for testing a module and may be just a part of a whole project.*
*It should not be used in a production or commercial environment.*
*Any cause of injury and/or death is the sole responsibility of the user.*