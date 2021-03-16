# From DAC-DMA to ADC to DAC out

Different strategies from DAC-DMA to ADC to DAC out 

Almost same as (https://github.com/VictorTagayun/NUCLEO-G474RE_DAC_DMA_LL-HAL_TIM6_HRTIM)[https://github.com/VictorTagayun/NUCLEO-G474RE_DAC_DMA_LL-HAL_TIM6_HRTIM] for the 2 Freq generator by DAC-DMA but will output the ADC data to DAC by ADC IT  

## Setup

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

	if(HAL_OK != HAL_HRTIM_WaveformOutputStart(&hhrtim1, HRTIM_OUTPUT_TE1))
	{
		Error_Handler();
	}

	if(HAL_OK != HAL_HRTIM_WaveformCounterStart(&hhrtim1, HRTIM_TIMERID_TIMER_E))
	{
		Error_Handler();
	}
	
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

* Enable ADC Calibration in main.c 

	/* Run the ADC calibration in single-ended mode */
	if (HAL_ADCEx_Calibration_Start(&hadc1, ADC_SINGLE_ENDED) != HAL_OK)
	{
		/* Calibration Error */
		Error_Handler();
	}

* Enable ADC DMAmain.c, use temporary variable _adc_data_ to check if triggered  
	
	/*##- Enable Regular ADC DMA Channel ##############################*/
	if (HAL_ADC_Start_DMA(&hadc1, adc_data, 1) != HAL_OK)
	{
		/* ADC conversion start error */
		Error_Handler();
	}

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
	
* Update ADC DMA 

	/* Start ADC and DMA */
	Fmac_Wdata = (uint32_t *) FMAC_WDATA;
	HAL_ADC_Start_DMA(&hadc1,Fmac_Wdata,1);


