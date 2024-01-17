# Save-time-in-C | Check Vibration

## Including the time.h libary

The line `#include <time.h>` in your code includes the standard C library `time.h`, which provides functions for manipulating date and time. 
These functions allow you to work with timestamps, get the current time, calculate time intervals, and much more.
It's an essential library for many types of C programs that work with time data.

## Include values
The line `char timeString[30], timeString1[30]; time_t rawtime[8],wptnow; time_t rawtime1[8],wptnow1;` in your code declares several variables.
`timeString` and `timeString1` are character arrays that can hold up to 30 characters each.
`rawtime`, `wptnow`, `rawtime1`, and `wptnow1` are variables of type `time_t`,
which is typically used to store system time values. The `rawtime` and `rawtime1` variables are arrays that can hold up to 8 `time_t` values each.

## Determine the times
The lines of code you provided are creating two `struct tm` variables, `wp` and `wp1`, which are used to hold time values in a format that is
easy to manipulate. The `tm_hour`, `tm_min`, and `tm_sec` fields of these structures are then set to specific values, representing the hours,
minutes, and seconds respectively. For `wp`, the time is set to 04:59:59, and for `wp1`, the time is set to 13:21:31.

`struct tm wp,wp1; 
wp.tm_hour = 04; 
wp.tm_min = 59; 
wp.tm_sec = 59;
wp1.tm_hour = 13;
wp1.tm_min = 21; 
wp1.tm_sec = 31;`


## Formatting in UTC time, shifts the array by one to update the values
The code you provided is doing two things. First, it's converting the `struct tm` variable `wp1` into a `time_t` variable `wptnow1` using the
`mktime` function. This function takes a pointer to a `struct tm` and returns the equivalent `time_t` value. Second, it's shifting the elements
of the `rawtime1` array one position to the right, starting from the fourth element and going down to the second. This could be used to make 
room for a new value at the beginning of the array.

`time_t wptnow1 = mktime(&wp1); 
for(int i = 4; i >=1; i--)
{
	rawtime1[i] = rawtime1[i-1];
}`

## The first array field is described
The line `rawtime1[0] = wptnow1;` in your code is assigning the value of `wptnow1` to the first element of the `rawtime1` array.
This could be used to store the newly calculated `time_t` value at the beginning of the array, after the previous values have
been shifted to the right.

`rawtime1[0] = wptnow1;`

## Converting to a char
The lines of code you provided are using the `strftime` function to format the time values stored in `rawtime` and `rawtime1` arrays. 
The `strftime` function formats the time represented in a `struct tm` into a string. It takes three arguments: a buffer to store the 
resulting string, the size of the buffer, and the format string that specifies how the time should be formatted. In your case, it's
being used to format the time into hours, minutes, and seconds (`"%H:%M:%S"`). The `localtime` function is used to convert a `time_t`
value into a `struct tm` represented in the local time zone. The `zeithochzaehlen` index is used to access the specific element in the
`rawtime` and `rawtime1` arrays.

`strftime(timeString, sizeof(timeString), "%H:%M:%S", localtime(&rawtime[zeithochzaehlen]));
strftime(timeString1, sizeof(timeString1), "%H:%M:%S", localtime(&rawtime1[zeithochzaehlen]));`

## Issue of the times
The code you provided is a loop that iterates over the first 8 characters of the `timeString1` array. For each iteration, it calls the 
function `lcd_printChar` with the current character from `timeString1`. This could be used to display the formatted time on an LCD screen,
one character at a time.

`for (int j = 0; j < 8; j++)
{
	lcd_printChar(timeString1[j]);
}`


## Check Vibration 

This function, `checkVibration()`, continuously checks for vibrations detected by a sensor connected to pin PD7. If a vibration is detected, it changes the LED status to red, initializes the LCD, and displays the warning message "Achtung" and "Erschuetterung !" on the LCD. This warning message is displayed for 200 milliseconds. To declare PD7 use this `PORTD |= PIND7; DDRD |= PD7;`

`while (!(PIND & (1 << PD7)))
	{
		ledstatus = ledrot;
		lcd_init();
		lcd_print("Achtung");
		lcd_setCursor(1, 0);
		lcd_print("Erschuetterung !");
		_delay_ms(200);
	}`
