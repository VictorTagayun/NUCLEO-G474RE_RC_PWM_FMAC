# Simple test and study how FMAC (Filter Math ACcelerator) works on 3p3z feedback using simple low pass RC circuit  

### Project files included  
NUCLEO-G474RE_RC_PWM_FMAC         = original  
NUCLEO-G474RE_RC_PWM_FMAC_iocOnly = troubleshooting, initially no feedback  
NUCLEO-G474RE_RC_PWM_FMAC_redo    = testing for consistensy and re-doing  
NUCLEO-G474RE_RC_PWM_FMAC_redo2   = troubleshooting, initially no feedback, testing and re-doing  
NUCLEO-G474RE_RC_PWM_FMAC_redo3   = variable output (to do)  

### Overview  
Use FMAC as 3p3z feedback and RC circuit to filter the PWM duty of PWM output as a feedback.
Then there is no need for actual converter to test the ff:
1. functionality of 3p3z
2. the code generated if there is a feedback happening


## Software setup



## Harware setup

As shown, the output of the PWM is fed to the RC circuit which acts as low pass filter.
The filtered voltage is fed back to the ADC and then to the FMAC 3p3z internally.
The FMAC will calculate the PWM duty according to the set voltage and accordingly will change the PWM duty.

### RC circuit  
Ch1 = PWM Output  
Ch2 = Filtered signal and fed to the ADC  
![RC circuit](https://github.com/VictorTagayun/NUCLEO-G474RE_RC_PWM_FMAC/blob/main/waveforms%26pixx(NUCLEO-G474RE_RC_PWM_FMAC)/IMG_2066.JPG)

### Voltage risetime and then went to close loop  
Ch1 = PWM Output  
Ch2 = Filtered signal and fed to the ADC  
![Vout](https://github.com/VictorTagayun/NUCLEO-G474RE_RC_PWM_FMAC/blob/main/waveforms%26pixx(NUCLEO-G474RE_RC_PWM_FMAC)/DS1Z_QuickPrint40.jpg)

### Zoom in  
![Zoom in](https://github.com/VictorTagayun/NUCLEO-G474RE_RC_PWM_FMAC/blob/main/waveforms%26pixx(NUCLEO-G474RE_RC_PWM_FMAC)/DS1Z_QuickPrint39.jpg)

### Other reference :   
[Test and study how FMAC (Filter Math ACcelerator) works](https://github.com/VictorTagayun/NUCLEO-G474RE_FMAC_Study_and_Analysis)  
[DSP Software Study and Analysis](https://github.com/VictorTagayun/NUCLEO-G474RE_CMSIS_DSP_Tutorial)  
### Social Media : [LinkedIn](https://www.linkedin.com/posts/victortagayun_weekendhobbyabrelectronics-funwithelectronics-activity-6753952834942844928-eF8g/)

*Disclaimer:*  

*The projects posted here are for learning and educational purposes only.*  
*The purpose of a certain project may be for testing a module and may be just a part of a whole project.*  
*It should not be used in a production or commercial enviroonment.*  
*Any cause of injury and/or death is the sole responsibility of the user.*  
