# TMS320F28377D-EPWM-for-bldc-motor-drive
Create EPWM signal for bldc motor drive

bldc 모터를 구동하기 위한 oneshot ESC protocol를 EPWM을 활용하여 구현하였습니다. 
oneshot protocol
-pulse 주기: 2khz(500us)
-pulse 폭: 125~250us

epwm_ex_bldc/board.c의 EPWM_init() 함수 내
EPWM_setClockPrescaler(myEPWM0_BASE, EPWM_CLOCK_DIVIDER_1, EPWM_HSCLOCK_DIVIDER_1); //EPWMCLK = 100MHz ,  TBCLK = EPWMCLK / (HSPCLKDIV * CLKDIV) = 100MHz
EPWM_setTimeBasePeriod(myEPWM0_BASE, 49999);//TPWM=(TBPRD+1)*(100MHz)=2khz
EPWM_setTimeBaseCounter(myEPWM0_BASE, 0);	
EPWM_setTimeBaseCounterMode(myEPWM0_BASE, EPWM_COUNTER_MODE_UP);	//count up
EPWM_disablePhaseShiftLoad(myEPWM0_BASE);	// 동기화 및 위상차 제어 하지 않음
EPWM_setPhaseShift(myEPWM0_BASE, 0);	
EPWM_setCounterCompareShadowLoadMode(myEPWM0_BASE, EPWM_COUNTER_COMPARE_A, EPWM_COMP_LOAD_ON_CNTR_ZERO);	// CMPA 레지스터가 Shadow 모드로 동작하도록 설정하는 것입니다.
//CPU가 CMPA 값을 쓸 때 Shadow 레지스터에 저장하고, 지정된 시점(예: TBCTR == 0)에만 Active CMPA로 복사되도록 설정->타이머 카운터가 0과 일치할 때, CMPA 레지스터의 Shadow 레지스터 값을 Active 레지스터에 반영
EPWM_setActionQualifierAction(myEPWM0_BASE, EPWM_AQ_OUTPUT_A, EPWM_AQ_OUTPUT_HIGH, EPWM_AQ_OUTPUT_ON_TIMEBASE_ZERO); // 타이머 카운터 값이 0과 일치할 때, EPWMxA 포트를 High 로 변경	
EPWM_setActionQualifierAction(myEPWM0_BASE, EPWM_AQ_OUTPUT_A, EPWM_AQ_OUTPUT_LOW, EPWM_AQ_OUTPUT_ON_TIMEBASE_UP_CMPA);  // 타이머 상승계수 중 CMPA 값과 일치할 때, EPWMxA 포트를 Low 로 변경 ->EPWM신호 생성

epwm_ex_bldc/epwm_ex_bldc.c 의 ESC_Calibration(); 함수 내
EPWM_setCounterCompareValue(myEPWM0_BASE, EPWM_COUNTER_COMPARE_A, 25000);// 타이머 상승계수 중 CMPA 값과 일치할 때, Low로 변경, EPWM=25000/49999=250/500(us)
DELAY_US(7000000);

EPWM_setCounterCompareValue(myEPWM0_BASE, EPWM_COUNTER_COMPARE_A, 12500); //EPWM=12500/49999=125/500(us)
DELAY_US(8000000);	 No newline at end of file

--------------------------------------------------------------------------------------------------------------
English ver.
The OneShot ESC protocol for driving a BLDC motor has been implemented using EPWM.
OneShot Protocol
- Pulse frequency: 2 kHz (500 µs)
- Pulse width: 125–250 µs

Within the `EPWM_init()` function in `epwm_ex_bldc/board.c`:
`EPWM_setClockPrescaler(myEPWM0_BASE, EPWM_CLOCK_DIVIDER_1, EPWM_HSCLOCK_DIVIDER_1);` // EPWMCLK = 100 MHz, TBCLK = EPWMCLK / (HSPCLKDIV * CLKDIV) = 100 MHz
`EPWM_setTimeBasePeriod(myEPWM0_BASE, 49999);` // TPWM = (TBPRD + 1) * (100 MHz) = 2 kHz
`EPWM_setTimeBaseCounter(myEPWM0_BASE, 0);`
`EPWM_setTimeBaseCounterMode(myEPWM0_BASE, EPWM_COUNTER_MODE_UP);` // Count up
`EPWM_disablePhaseShiftLoad(myEPWM0_BASE);` // Synchronization and phase-shift control disabled
`EPWM_setPhaseShift(myEPWM0_BASE, 0);`
`EPWM_setCounterCompareShadowLoadMode(myEPWM0_BASE, EPWM_COUNTER_COMPARE_A, EPWM_COMP_LOAD_ON_CNTR_ZERO);` // Configures the CMPA register to operate in shadow mode. // Configure the system so that when the CPU writes a value to CMPA, it is stored in a shadow register and only copied to the active CMPA register at a specified point (e.g., when TBCTR == 0). This ensures the active register is updated with the shadow register's value when the timer counter matches zero.
EPWM_setActionQualifierAction(myEPWM0_BASE, EPWM_AQ_OUTPUT_A, EPWM_AQ_OUTPUT_HIGH, EPWM_AQ_OUTPUT_ON_TIMEBASE_ZERO); // Set the EPWMxA port to HIGH when the timer counter matches zero.
EPWM_setActionQualifierAction(myEPWM0_BASE, EPWM_AQ_OUTPUT_A, EPWM_AQ_OUTPUT_LOW, EPWM_AQ_OUTPUT_ON_TIMEBASE_UP_CMPA);  // Set the EPWMxA port to LOW when the timer counter matches the CMPA value during the up-count phase; this generates the EPWM signal.

Within the ESC_Calibration() function in epwm_ex_bldc/epwm_ex_bldc.c:
EPWM_setCounterCompareValue(myEPWM0_BASE, EPWM_COUNTER_COMPARE_A, 25000); // Set to LOW when the timer matches the CMPA value during the up-count phase; EPWM = 25000/49999 = 250/500 (us).
DELAY_US(7000000);

EPWM_setCounterCompareValue(myEPWM0_BASE, EPWM_COUNTER_COMPARE_A, 12500); // EPWM = 12500/49999 = 125/500 (us).
DELAY_US(8000000);