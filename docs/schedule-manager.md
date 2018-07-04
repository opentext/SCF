# Schedule manager
The schedule manager handles all scheduled events in an SCF instance. Scheduled events include for example scanning connectors for new input, schedule spooling of queues, checking for heartbeat failures of Communications Server jobs and many other things. The schedule manager has no customizable configuration of it's own but in some places in the user interface you will see examples of the syntax used to define schedules so therefor this syntax will be described here.

## Schedule syntax
The schedule syntax described in [bachus naur form](https://en.wikipedia.org/wiki/Backus%E2%80%93Naur_form):

```bnf

; <task> is the top level definition of a scheduled task
<task> := 'T' 'II' <start> <end> <interval>

; <start> is an exact time stamp that defines the earliest time when a scheduled task is allowed to start.
; To specify that there is no start time (start as soon as possible), write '*'
<start> := <timestamp>

; <end> is an exact time stamp that defines the time after which a scheduled task will no longer be in use.
; To specify that there is no end time stamp (never stop scanning), write '*'
<end> := <timestamp>

; time stamps are expressed as year month day hour minute second fractions (milliseconds) like this:
; YYYYMMDDhhmmssfff
; e.g. 20180803080000000
<timestamp> := '*' | <digit> <digit> <digit> <digit> <digit> <digit> <digit> <digit> <digit> <digit> <digit> <digit> <digit> <digit> <digit> <digit> <digit>

<intervals> := <interval> | <interval> <intervals>
<interval> := <and> <or> <step>

; The number of <intervals> must match the <count> number
<and> := '&' <count> <intervals>

; The number of <intervals> must match the <count> number
<or> := '|' <count> <intervals>

; A step is a unit, start, stop and interval
<step> := <unit> <intervalstart> <intervalstop> <intervalstep>

; The following units are defined:
;   Y  = year - valid values between 0 and 9999
;   MY = month of year - valid values between 1 and 12
;   WY = week of year - valid values between 1 and 53
;   WM = week of month - valid values between 1 and 5
;   DY = day of year - valid values between 1 and 366
;   DM = day of month - valid values between 1 and 31
;   DW = day of week - valid values between 1 and 7 where Monday is 1 and Sunday is 7
;   H  = hour - valid values between 0 and 23
;   MH = minutes - valid values between 0 and 59
;   S  = seconds - valid values between 0 and 59
;   MS = milliseconds - valid values between 0 and 999
<unit> := "Y" | "MY" | "WY" | "WM" | "DY" | "DM" | "DW" | "H" | "MH" | "S" | "MS"

; The interval start time is either a * meaning the minimum default value (see units above) or a number.
; If a number is specified then no scanning will be done before this value of the interval, e.g. for hours if 10 is specified then scanning will start at 10 AM.
<intervalstart> := "*" | <digits>

; The interval stop time is either a * meaning the maximum default value (see units above) or a number.
; If a number is specified then no scanning will be done at or after this value of the interval, e.g. for day of week if 6 is specified then scanning will not be done on weekends.
<intervalstop> := "*" | <digits>

; A * which is interpreted as 1 or a number which. The number of units between each scan, e.g. for minutes if 5 is specified then scanning is done every 5 minutes.
<intervalstep> := "*" | <digits>

<count> := <digits>
<digits> := <digit> | <digit> <digits>
<digit> := "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"

```
