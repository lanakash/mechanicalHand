#include	"msp430.h"
#include	<math.h>

#define LED1 			BIT0
#define LED2 			BIT6

#define BUTTON 			BIT3

#define F				  0
#define B 				1
#define FS 				2
#define BS 				3

#define MAX				2500
#define MIN 				500

volatile unsigned int State;
volatile float Curr;

void InitializeButton(void);
void PreApplicationMode(void);
void CurrentPosition(void);

void main(void)
{
	WDTCTL = WDTPW + WDTHOLD;			// Stop WDT
	P1DIR |= BIT2;                  					// Set P1.0 to output direction
	P1SEL |= BIT2;						// P1.0 to TA0.1
	P2DIR |= BIT2;						//Set P2.2
	P2SEL |= BIT2;						//P2.2 to TA0.1

	/* next three lines to use internal calibrated 1MHz clock: */
	BCSCTL1 = CALBC1_1MHZ;			// Set range 1MHz
	DCOCTL = CALDCO_1MHZ;			// Set DCO step + modulation
	BCSCTL2 &= ~(DIVS_3);                     		// SMCLK = DCO = 1MHz
	BCSCTL3 |= LFXT1S_2;				// SMCLK = VLO

	/* Set up initial states and Initialization: */
	Curr = 500;
	State = BS;	
	CurrentPosition();				
	InitializeButton();

	/* Set ports for leds: */
	P1DIR |= LED1 + LED2;
	P1OUT &= ~(LED1 + LED2);

	__enable_interrupt();                     	// Enable interrupts.


	/* Main: Each loop moves motors a small amount or maintains position */
	while⁪(1)
	{
		if (State == F){
			P1OUT |= LED2;  				//Green led on if in forward state
			if (Curr < MAX+1){		//moves motor forward if not at max position
				Curr +=0.5;        	 //Varing Curr (*) will change speed
				TA0CCR0 = 25000-1; 			// PWM Period
				TA0CCTL1 = OUTMOD_7;			// CCR1 reset/set
				TA0CTL = TASSEL_2 + MC_1; 		// SMCLK, up mode
				TA0CCR1 = Curr; 				// CCR1 PWM duty cycle
				TA1CCR0 = 25000-1; 			// PWM Period
				TA1CCTL1 = OUTMOD_7;			// CCR1 reset/set
				TA1CTL = TASSEL_2 + MC_1; 		// SMCLK, up mode
				TA1CCR1 = Curr; 				// CCR1 PWM duty cycle
				__delay_cycles(50);	   
			}else{
				CurrentPosition();			// Maintain current position
			}

		}else if (State == B){
			P1OUT |= LED1;   					//Red led on if in back state
			if (Curr > MIN-1){	      //moves motor backwards if not at minimum position					Curr -=0.5;					//(*)		
				TA0CCR0 = 25000-1; 			// PWM Period
				TA0CCTL1 = OUTMOD_7;			// CCR1 reset/set
				TA0CTL = TASSEL_2 + MC_1; 		// SMCLK, up mode
				TA0CCR1 = Curr; 				// CCR1 PWM duty cycle
				TA1CCR0 = 25000-1; 			// PWM Period
				TA1CCTL1 = OUTMOD_7;			// CCR1 reset/set
				TA1CTL = TASSEL_2 + MC_1; 		// SMCLK, up mode
				TA1CCR1 = Curr; 				// CCR1 PWM duty cycle
				__delay_cycles(50);			
			}else{
				CurrentPosition();			// Maintain current position
			}

		}else{
			__delay_cycles(5000);
		}
	}
}

void InitializeButton(void) 					// Configure Push Button
{
	P1DIR &= ~BUTTON;
	P1OUT |= BUTTON;
	P1REN |= BUTTON;
	P1IES |= BUTTON;
	P1IFG &= ~BUTTON;
	P1IE |= BUTTON;
	__delay_cycles(5000);
}

void CurrentPosition(void)						// Set Position
{
	TA0CCR0 = 25000-1;						// PWM Period
	TA0CCTL1 = OUTMOD_7;						// CCR1 reset/set
	TA0CTL = TASSEL_2 + MC_1; 					// SMCLK, up mode
	TA0CCR1 = Curr; 							// CCR1 PWM duty cycle
	TA1CCR0 = 25000-1; 						// PWM Period
	TA1CCTL1 = OUTMOD_7;						// CCR1 reset/set
	TA1CTL = TASSEL_2 + MC_1; 					// SMCLK, up mode
	TA1CCR1 = Curr; 							// CCR1 PWM duty cycle
	__delay_cycles(50);

}

/* *************************************************************
         Port Interrupt for Button Press
    Changes State when button is pressed
 * *********************************************************** */
void __attribute__ ((interrupt(PORT1_VECTOR))) port1_isr (void)
{

 /*disables port1 interrupts for a little while so that it doesn't try to respond to two consecutive button pushes right together. The watchdog timer interrupt will re-enable port1 interrupts */

 	P1IFG = 0;  							// clear out interrupt flag for port 1
	P1IE &= ~BUTTON;         					// Disable port 1 interrupts
	WDTCTL = WDT_ADLY_250;   				// set up watchdog timer duration
	IFG1 &= ~WDTIFG;         					// clear interrupt flag
	IE1 |= WDTIE;           			 		// enable watchdog interrupts
	TACCTL1 = 0;            			 		// turn off timer 1 interrupts
	
	P1OUT &= ~(LED1+LED2);  				 // turn off the leds

	/* Changes State: */
	if (State == F) {						
		State = FS;
	}else if (State == FS){
		State = B;
	}else if (State == B){
		State = BS;
	}else if (State == BS){
		State = F;
	}
}

// WDT Interrupt Service Routine used to de-bounce button press
void __attribute__ ((interrupt(WDT_VECTOR))) wdt_isr (void)
{
	IE1 &= ~WDTIE;					// disable watchdog interrupt
	IFG1 &= ~WDTIFG;             			// clear interrupt flag
	WDTCTL = WDTPW + WDTHOLD;		// put WDT back in hold state
	P1IE |= BUTTON;             			// Debouncing complete reenable port 1 interrupts
}
