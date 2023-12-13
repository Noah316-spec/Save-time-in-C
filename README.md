# ProbePruefung2023
Abfrage checkhumidundtemp
if(((x <= 20)&&(x<= 22))&&((y>= 40)&&(y<= 60))) // optimal
	{
		ledstatus = ledgreen;
	}
else if(((((x < 20) && (x >= 16)) || ((x > 22) && (x <= 26))) && ((y >= 30) && (y <= 70))) || ((((y < 40) && (y >= 30)) || ((y > 60) && (y <= 70))) && ((x < 26) && (x > 16)))) //toleriert
	{
		ledstatus = ledgelb;
	}
	else if ((x<16)||(y<30)||(x>26)||(y>70))
	{
		ledstatus = ledrot;
	}
	else
	{
		ledstatus= ledrot;
	}
	return ledstatus;
