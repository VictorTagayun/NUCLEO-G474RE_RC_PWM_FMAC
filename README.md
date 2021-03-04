# Test and study how FMAC (Filter Math ACcelerator) works on 3p3z feedback  

### Project files included  
NUCLEO-G474RE_RC_PWM_FMAC         = original  
NUCLEO-G474RE_RC_PWM_FMAC_iocOnly = troubleshooting, initially no feedback  
NUCLEO-G474RE_RC_PWM_FMAC_redo    = testing for consistensy and re-doing  
NUCLEO-G474RE_RC_PWM_FMAC_redo2   = troubleshooting, initially no feedback, testing and re-doing  
NUCLEO-G474RE_RC_PWM_FMAC_redo3   = variable output (to do)  

### Overview  
Using FMAC as 3p3z feedback and use RC circuit to filter the PWM duty of PWM output as a feedback.
Then there is no need for actual converter to test the ff:
1. functionality of 3p3z
2. the code generated if there is a feedback happening


## Software setup



## Harware setup

As shown, the output of the PWM is fed to the RC circuit which acts as low pass filter.
The filtered voltage is fed back to the ADC and then to the FMAC 3p3z internally.
The FMAC will calculate the PWM duty according to the set voltage and accordingly will change the PWM duty.

RC circuit  
![RC circuit](https://github.com/VictorTagayun/NUCLEO-G474RE_RC_PWM_FMAC/blob/main/waveforms%26pixx(NUCLEO-G474RE_RC_PWM_FMAC)/IMG_2066.JPG)

Voltage risetime and then went to close loop  
![Vout](https://github.com/VictorTagayun/NUCLEO-G474RE_RC_PWM_FMAC/blob/main/waveforms%26pixx(NUCLEO-G474RE_RC_PWM_FMAC)/DS1Z_QuickPrint40.jpg)

Zoom in  
![Zoom in](https://github.com/VictorTagayun/NUCLEO-G474RE_RC_PWM_FMAC/blob/main/waveforms%26pixx(NUCLEO-G474RE_RC_PWM_FMAC)/DS1Z_QuickPrint39.jpg)

More info : [LinkedIn](https://www.linkedin.com/posts/victortagayun_weekendhobbyabrelectronics-funwithelectronics-activity-6753952834942844928-eF8g/)
