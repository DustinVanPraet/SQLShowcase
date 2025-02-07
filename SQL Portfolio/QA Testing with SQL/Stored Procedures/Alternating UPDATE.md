#### During QA Testing, some bugs/issues can come from two turns being played in tandem. So it's safe practice to test the previous event as well as the expected crash event. In order to reproduce the bug/issue, this query takes two specific turns using their serial numbers provided in the debug logs. It when then UPDATE the database to alternate between the two provided turns. 

```sql

DECLARE @cnt			         INT = 0;
DECLARE @1stpulltabIDstart INT;
DECLARE @2ndPulltabIDstart INT;
DECLARE @1stALTpulltabID   INT;
DECLARE @2ndALTpulltabID   INT;
DECLARE @1stSerialNumber   VARCHAR(50);
DECLARE @2ndSerialNumber   VARCHAR(50);
DECLARE @gamesetID		     INT;
DECLARE @numberOfCycles	   INT;

--The following 6 variables are all that you change. You only need the pulltabID's or the S/N's but
--you don't need both. 

SET @gamesetID			    = 663;	   --Gameset that you're going to be updating
SET @1stALTpulltabID	  = 16598014 --PulltabID you want to alternate
SET	@2ndALTpulltabID	  = 16597992 --2nd PulltabID you want to alternate
SET @1stSerialNumber    = ''	     --Serial_number you want to alternate	
SET @2ndSerialNumber    = ''	     --2nd Serial_number you want to alternate	
SET @numberOfCycles		  = 50		   --Number of times you want the turns to be repeated


SET @1stpulltabIDstart = (	SELECT	TOP(1) pulltab_id
							              FROM	pulltab_tickets
							              WHERE	gameset_id = @gamesetID
							              AND		has_been_played = 0)
SET @2ndPulltabIDstart = (	SELECT	TOP 1 (SELECT TOP(1) pulltab_id
                											      FROM	pulltab_tickets
                											      WHERE	gameset_id = @gamesetID
                											      AND		has_been_played = 0) + 1
							              FROM pulltab_tickets)

WHILE @cnt < @numberOfCycles * 2 -- Change to the number of times you want turns to be repeated.   

	BEGIN	
		UPDATE	pulltab_tickets
		SET		win_amount		= (	SELECT win_amount
									FROM pulltab_tickets
									--Input 1st pulltabID or S/N you want to alternate
									WHERE	pulltab_id = @1stALTpulltabID
									OR		serial_number = @1stSerialNumber
								   ),
				  win_data_json	= ( SELECT win_data_json
									FROM pulltab_tickets
									--Input  1st pulltabID or S/N you want to alternate
									WHERE pulltab_id = @1stALTpulltabID
									OR		serial_number = @1stSerialNumber
								   )
		FROM	pulltab_tickets
		WHERE	pulltab_id = @1stpulltabIDstart

		UPDATE	pulltab_tickets
		SET		win_amount		= (	SELECT win_amount
									FROM pulltab_tickets
									--Input pulltabID or S/N you want to alternate
									WHERE pulltab_id = @2ndALTpulltabID
									OR		serial_number = @2ndSerialNumber
								   ),
				win_data_json	= ( SELECT win_data_json
									FROM pulltab_tickets
									--Input pulltabID or S/N you want to alternate
									WHERE pulltab_id = @2ndALTpulltabID
									OR		serial_number = @2ndSerialNumber
								   )
		FROM	pulltab_tickets
		WHERE	pulltab_id = @2ndPulltabIDstart


		SET		@cnt += 1
		SET		@1stpulltabIDstart += 2
		SET		@2ndPulltabIDstart += 2

	END

SELECT	*
FROM	pulltab_tickets 
WHERE	gameset_id = @gamesetID
AND		has_been_played = 0
```

