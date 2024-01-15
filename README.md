#Zeitenspeichern in Atmel

#Import dependencies from lib

include <time.h>
char timeString[30], timeString1[30]; time_t rawtime[8],wptnow;
time_t rawtime1[8],wptnow1;

#Feststellen der Zeit

struct tm wp,wp1; // Struktur für aktuelle Zeit 
wp.tm_hour = 04; // Stunde einstellen
wp.tm_min = 59; // Minute einstellen
wp.tm_sec = 59; // Sekunde einstellen
wp1.tm_hour = 13; // Stunde einstellen
wp1.tm_min = 21; // Minute einstellen
wp1.tm_sec = 31; // Sekunde einstellen

#Speichern der Werte 
time_t wptnow1 = mktime(&wp1); // Die Funktion 'mktime' formt das 'struct' in eine UTC-Zeit um und schreibt das in 'wptnow1' (time_t)
			 
for(int i = 4; i >=1; i--) // Verschiebt das Array um eins, damit die Werte aktualisiert werden.  
{
	rawtime1[i] = rawtime1[i-1];
}
			 
rawtime1[0] = wptnow1; // Beschreibt das erste Feld

#Umformen des Strings 

strftime(timeString, sizeof(timeString), "%H:%M:%S", localtime(&rawtime[zeithochzaehlen]));
strftime(timeString1, sizeof(timeString1), "%H:%M:%S", localtime(&rawtime1[zeithochzaehlen]));

#Ausgabe der Zeiten
for (int j = 0; j < 8; j++)
{
	lcd_printChar(timeString1[j]);
}

