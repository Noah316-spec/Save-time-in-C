# Zeiten speichern in C


#define F_CPU 16000000UL // Definiert die Frequenz der CPU als 16 MHz

// Einbinden der Bibliothek
#include "hd44780.h" //LCD
#include "i2c_master.h" //I2C
#include "pcf8574.h" // PCF8574 I/O Expander Bibliothek
#include "uart.h"  // uart 
#include "adc32.h" // 32-Bit ADC Bibliothek
#include "mcp4725.h" // MCP4725 DAC Bibliothek

// Einbinden von Standardbibliotheken
#include <time.h> // Zeitfunktionen
#include <avr/io.h> // I/O Funktionen für AVR Mikrocontroller
#include <util/delay.h> // Verzögerungsfunktionen


// Deklarieren und initialisieren
volatile uint8_t buttons = 0xff;
//////////////////////////////////////////////////////////////////////////
volatile int i1=0 ,ledstatus, ledgruen = 0b01111111, ledgelb = 0b10111111, ledrot = 0b11011111; // Leds werden durch ein low geschaltet. Ledstatus ist der rückgabevariable. i1 ein übergabevariable
volatile double adctemp= 0.0, adchumid =0.0; 
//////////////////////////////////////////////////////////////////////////

char timeString[30];
time_t rawtime[8],tnow;



int checkhumidandtemp(double x, double y) // Abfrage der Humid und Temperatur
{
	struct tm current_time;  // Struktur für aktuelle Zeit
	current_time.tm_hour = 12;  // 12:00:00
	current_time.tm_min = 59;
	current_time.tm_sec = 59;
	
	
	if(((x >= 20)&&(x<= 22))&&((y>= 40)&&(y<= 60))) // Abfrage für den optimalen Bereich 
	{ 
		ledstatus = ledgruen; // led wird grün 
	}
    else if(((((x < 20) && (x >= 16)) || ((x > 22) && (x <= 26))) && ((y >= 30) && (y <= 70))) || ((((y < 40) && (y >= 30)) || ((y > 60) && (y <= 70))) && ((x < 26) && (x > 16)))) // Abfrage für den toleriert Bereich
	{
		 ledstatus = ledgelb;  // led wird gelb 
	} 
	else if ((x<16)||(y<30)||(x>26)||(y>70) && (ledstatus == ledgelb)) // Abfrage für den roten Bereich
	{
		 tnow = mktime(&current_time);
		 for(int i = 4; i >=1;i--)
		 {
			 rawtime[i]=rawtime[i-1];
		 }
		 rawtime[0]= tnow;
		 
		 ledstatus = ledrot; // led wird rot
	}
	else
	{ 	
	ledstatus= ledrot; // wen nichts stimmt soll die led rot werden überprüfen bzw. Erschuetterung ist 
	} 
	
	return ledstatus; // rückgabewert
	
	
}
//////////////////////////////////////////////////////////////////////////
void abfrageErschuetterung()
{

	while (!(PIND & (1<<PD7))) // Prüft ob der PD7 auf low ist, zustand änderung springt er raus
	{
		ledstatus = ledrot; // LED wird als Rot angezeigt damit der Fehler signalisiert wird
		lcd_init(); // initialiseren des Displays 
		lcd_print("Achtung"); // Print Achtung
		lcd_setCursor(1,0); // position der schrift wird festgelegt (zweite zeile)
		lcd_print("Erschuetterung !");
		_delay_ms(200); // Verzögerung um ein flüssigeres Bild zu bekommen
	}
	
}
//////////////////////////////////////////////////////////////////////////
 
int main(void)
{
	PORTD |= PIND7;	// setzt den Pin des Port D auf high 
	DDRD |= PD7; // PD7 als Ausgang 
	
	adc_init(); // Analog digital COnverter initialiseren 
    while (1) 
    {
		abfrageErschuetterung();
		
		adchumid = adc_readvoltage(0); // gibt die Feuchtigkeit zurück, kanal 0 = adchumid
		_delay_ms(10); // vermeidung von fehlern beim Konvertieren
		adctemp = adc_readvoltage(1); // gibt die Temperatur zurück, kanal 1 = adctemp
		
		i1= checkhumidandtemp(adctemp, adchumid); // Funktionsaufruf von checkhumidandtemp plus wert übergabe
		pcf8574_set_outputs(0x21,i1); //steuerung der leds 0x21, adresse vom PCF(für die LEDs)
		
		lcd_init(); 
		lcd_print("Temp:");
		lcd_setCursor(0,10);	
		lcd_printDouble(adctemp,1); // adctemp wird übergeben
		lcd_printChar(DEGREE); // definierter Wert eines Gradzeichen
		lcd_print("C");
		lcd_setCursor(1,0);
		lcd_print("Humid:");
		lcd_setCursor(1,10);
		lcd_printDouble(adchumid,1);
		lcd_printChar(PERCENT); // definierter Wert eines Prozentzeichen
		_delay_ms(200); //Zeitverzögerung und besseres/Flüssigere anzeige am LCD	
		
		buttons = pcf8574_get_inputs(0x20);
		if (buttons == 0b11111011)
		{
			volatile uint32_t counter = 0;
			int i = 0;
			int abfrage = 1;
			
			
			if(abfrage == 1)
			{
				while(i<= 4)
				{
					strftime(timeString,sizeof(timeString), "%H:%M:%S", localtime(&rawtime[i]));
					buttons = pcf8574_get_inputs(0x20);
					if(buttons == 0b11110111)
					{
						i++;
						counter = 0;
						_delay_ms(250);
					}
					else
					{
						lcd_init();
						lcd_print("Zeit ");
						lcd_printInt(i);
						lcd_print(": ");
						lcd_setCursor(1,0);
						if (counter == 600)
						{
							i = 5;
							buttons = 0b11101111;
							abfrage = 0;
							
						}
						for (int j = 0; j<8 ;j++)
						{
							lcd_printChar(timeString[j]);
						}
						
					}
					counter++;
					_delay_ms(10);
				}
				while ((buttons != 0b11011111)&&(buttons != 0b11101111))
				{
					lcd_init();
					buttons = pcf8574_get_inputs(0x20);
					lcd_print("Zeitenloeschen ?");
					lcd_setCursor(1,0);
					lcd_print("Y(S5)/N(S6)");
					_delay_ms(100);
				}
				
				if (buttons == 0b11101111)
				{
					rawtime[0] = 0;
					rawtime[1] = 0;
					rawtime[2] = 0;
					rawtime[3] = 0;
					rawtime[4] = 0;
				}
				else if(buttons == 0b11011111)
				{
					
				}
			}
		}
		
    }
	return 0;
}

