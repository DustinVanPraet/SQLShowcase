## Improving the SELECT/UPDATE processes in the database to improve testing practices

*Disclaimer:* *All turns, pools, and payouts displayed in this project do not accurately reflect the games as played in the field in any way and these are simply used to showcase SQL skills for Quality Assurance testing purposes.* 


### This will show the growth and changes that were made and how the position has become more efficient. The 1st part of this will display some spruced up versions of how we initially shown to assist the QA team. The 2nd part will show the ways that SQL queries became more complex but overall faster reducing QA wait times by more than 80%. 

Within the gambling industry for casino slot machine games, all states have different regualtions and requirements. Some of these states require a record of every turn played by every machine and this is recorded in a SQL server database for each individual location which may have many different machines that can play many different games. 
<br>
<br>
In the QA department, all posible multipliers and seeds need to be tested across each individual game to experience and test different features that may not occur but 1 time out of 50,000 turns played. To circumvent this issue while also testing in a Live environment, we will write UPDATE statements to manipulate records to force low chance turns into playing when the terminal reaches out to the database for a turn. 
<br>
<br>
Below is an Entity Relationship Diagram displaying some tables within the DB:
<br>
<br>
![dbSchema](https://github.com/user-attachments/assets/70058011-f689-489b-b1f7-b72eeb62d99a)
<br>
<br>
In the image you can see 4 tables. These are not all of the tables but they are the main tables focused on for QA testing. A simple explaination would be that a game can have many game_forms(game_forms are like bet amounts ie $2.00, $1.00, $0.50 etc.), a game_form can have many gamesets, and gameset can have many tickets. 

### Scenario:
In this example, a member of the QA team is requesting a specific order of turns for testing. They want to test the different multipliers and how they react when played in that specific order. The QA member asks for a 10x, 1x, 5x, and 50x turn for FLAMINJOKEROH on the $1.00 denomination in that specific order.  

#### Input:
```sql
--Return all currently active gamesets that are available on terminals with their game.name and denomination. 
SELECT  gameset.gameset_id,
	gameset.current_tickets_played,
	gameset.game_form_id,
	gameset.current_total_won,
	game_form.denomination,
	game.name
	
FROM	gameset
JOIN    game_form ON gameset.game_form_id=game_form.form_id
JOIN    game ON game_form.game_id=game.game_id
JOIN    store ON gameset.store_id=store.store_id
JOIN    pulltab_tickets ON gameset.gameset_id = pulltab_tickets.gameset_id

WHERE	gameset.is_active = 1 
AND     game_form.is_active = 1
	
GROUP BY	gameset.gameset_id,
  		gameset.current_tickets_played,
	  	gameset.game_form_id,
		gameset.current_total_won,
		game_form.denomination,
		game.name
```
#### Output:
![ActiveGamesetsOutput2](https://github.com/user-attachments/assets/8348aa62-939b-47f8-9c1a-d2048eeec0e6)
<br>
The database analyst would then manually search for the gameset_id that aligned with the denomination and name that they were looking for. This would be gameset_id 1482.
The next step would be to search for a win_amount containing 10x for $1.00 denomination which would be 10.00. The next query would be written to find this information.
<br>
#### Input:
```sql
SELECT	*
FROM	pulltab_tickets
WHERE	gameset_id = 1482
AND 	win_amount = 10.00
```
#### Output:
![WinAmountOutputpng](https://github.com/user-attachments/assets/36f637ea-68cc-42fa-b783-60b22c0e0152)
<br>
At this time, the QA team can't continue testing on the specific game and denomination that was requested. The database analyst has found a win_amount of 10.00 as needed and will take that and the win_data_json for the turn needed to copy over to the next upcoming ticket. But the pulltab_id still has to be found and the following query would be written to find the answer.
<br>
#### Input:
```sql
SELECT	* 
FROM	pulltab_tickets
WHERE	gameset_id = 1482
AND	has_been_played = 0
```
#### Output:
![HasBeenPlayedOutput](https://github.com/user-attachments/assets/8cd825bf-e14e-43df-b154-e23da951f8e7)
<br>
Now we have pulltab_id for the next ticket to be played and an UPDATE can be made for the first requested turn. We will re-run the SELECT statement alongside the UPDATE to view the changes to pulltab_id = 34774001
<br>
#### Input:
```sql
UPDATE 	pulltab_tickets
SET 	win_amount = 10,
	win_data_json = '[{"IconData":[1,12,12,12,2,12,7,4,4],"BonusData":[],"WinAmount":0.0,"BonusWinData":[]}]'
WHERE	pulltab_id = 34774001

SELECT 	*
FROM	pulltab_tickets
WHERE 	gameset_id = 1482
AND 	has_been_played = 0
```
#### Output:
![UpdatePulltabTicketOutput](https://github.com/user-attachments/assets/adbf3163-bf85-46b4-8cb5-d1ca788db34b)
<br>
This process would be repeated for each individual multiplier requested. If the same multiplier was requested repeatedly such as a 10x turn 5 times then it may have be written as: 
<br>
#### Input:
```sql
UPDATE 	pulltab_tickets
SET 	win_amount = 10,
	win_data_json = '[{"IconData":[1,12,12,12,2,12,7,4,4],"BonusData":[],"WinAmount":0.0,"BonusWinData":[]}]'
WHERE	pulltab_id BETWEEN 34774001 AND 34774005

SELECT 	*
FROM	pulltab_tickets
WHERE 	gameset_id = 1482
AND 	has_been_played = 0
```
This would UPDATE all 5 tickets BETWEEN pulltab_id 34774001 AND 34774005 but this would allow testing to continue while the UPDATE is being made due to the query not changing on the fly while running the UPDATE. I never wanted to impede on others testing and work. I didn't want to be the reason slowing them down anymore than necessary. 
<br>
#### Input:
```sql
--Change the TOP number if more than 5 tickets are needed for testing.
UPDATE TOP(5) pulltab_tickets
--Nested SELECT statements to directly copy cell data over to another record. 
SET 	win_amount	= (SELECT win_amount
			   FROM pulltab_tickets
			   WHERE pulltab_id = 34783533),
	win_data_json	= (SELECT win_data_json
			   FROM pulltab_tickets
			   WHERE pulltab_id = 34783533)
FROM	pulltab_tickets 
WHERE	gameset_id = 1482 
AND	has_been_played = 0 

--Return the recently UPDATED gameset to verify that the turns have been changed. 
SELECT	*
FROM	pulltab_tickets
WHERE	gameset_id = 1482
AND	has_been_played = 0
```
This query does same UPDATE as the previous except that it does a few processes better. It runs check for the has_been_played = 0 and starts the UPDATE at the first 0 value. If the next ticket to be played was 1,500 tickets later, it would know to start the UPDATE at 34775501 allowing QA to continue testing without a need to wait between requests. It also has a nested SELECT statement that pulls the win_amount and win_data_json over from the selected pulltab_id. This is safer since it eliminates user error that could occur from manually copying and pasting. Some win_data_json can get to a LEN of 1,000 or more depending on the turn. 
<br>
<br>
Figuring out a faster way to fulfill requests felt great but it wasn't enough. I wanted a way to put more power into the hands of QA when they don't fully understand how SQL and the database works. This would allow them to queue up their own requests with minimal difficulty. I started looking into Stored Procedures and Variables.
#### Input:
```sql
--Input gameset_id for the game, the win_amount(not multiplier), and the # of times you want the turn queued up. 
Create Procedure QueueTicketWinAmount
		@gamesetID	INT,
		@win_amount	DECIMAL(18,2),
		@numberOfTurns  INT
AS
UPDATE 	TOP(@numberOfTurns) pulltab_tickets
SET 	win_amount	=(SELECT	TOP(1)  pulltab_tickets.win_amount
					FROM	pulltab_tickets
					WHERE	gameset_id = @gamesetID and win_amount = @win_amount),	
	win_data_json	=(SELECT	TOP(1)  pulltab_tickets.win_data_json
					FROM	pulltab_tickets
					WHERE	gameset_id = @gamesetID and win_amount = @win_amount)
FROM pulltab_tickets
WHERE gameset_id = @gamesetID and has_been_played = 0
```
This was my first attempt at simplifying and expediting the UPDATE process. This would eliminate the need to look up a win_amount and copying over the pulltab_id from the desired multiplier. The user would only have to execute a stored procedure. 
#### Input:
```sql
-- Input variables for gameset_id, win_amount desired, and # of times the turn repeats. 
EXEC QueueTicketWinAmount 1482, 10.00, 5
```
Unfortunately, the following are still additional issues that aren't solved with this Stored Proc:
- It didn't eliminate the need to look up the gameset_id using the active gamesets query. 
- It doesn't expedite and solve the initial issue when requested multipliers are 10x, 1x, 5x, and 50x in that specific order. 
- It also doesn't run a check on errors if the win_amount doesn't exist within a gameset.
- Accidentally inserting the wrong gameset_id can put the turns on another gameset and cause a forced crash resulting in a loss of resources chasing a one time, non existent bug.
- Some turns can win on the same multiplier over 100,000 different ways. Win_data_json determines how the solving will play out but our previous attempts uses the same win_data_json for every turn.
<br>
With this starting foundation, I decided to dig deeper. In order to solve all of the issues listed above as several others, the following Stored Procedure was created:
<br>

#### Input:

`````sql
CREATE PROCEDURE QueueMultiplier
			@gameName	VARCHAR(50), 
			@denom		DECIMAL(18,2),
			@1stWin		DECIMAL(18,2),
			@2ndWin		DECIMAL(18,2) = NULL,
			@3rdWin		DECIMAL(18,2) = NULL,
			@4thWin		DECIMAL(18,2) = NULL,
			@5thWin		DECIMAL(18,2) = NULL,
			@6thWin		DECIMAL(18,2) = NULL,
			@7thWin		DECIMAL(18,2) = NULL,
			@8thWin		DECIMAL(18,2) = NULL,
			@9thWin		DECIMAL(18,2) = NULL,
			@10thWin	DECIMAL(18,2) = NULL,
			@11thWin	DECIMAL(18,2) = NULL,
			@12thWin	DECIMAL(18,2) = NULL

AS
BEGIN
-- DECLARE/SET VARIABLES AND SETTINGS
DECLARE		@counter		INT;
DECLARE		@MaxCounter		INT;
DECLARE		@gameset		INT;
DECLARE		@rollingWin		DECIMAL(18,2);
DECLARE		@nextGameset		INT;

SET			NOCOUNT		ON;
SET			@counter	= 1;
SET			@gameset	=(SELECT	top(1)gameset_id
				  	FROM		gameset
 					JOIN		game_form ON game_form.form_id = gameset.game_form_id
					JOIN		game ON game.game_id = game_form.game_id
					WHERE		gameset.is_active = 1
					AND		game.name = @gameName
					AND		game_form.denomination = @denom);
SET			@nextGameset 	=(SELECT	TOP(1)	gameset_id
							FROM	gameset
							WHERE	game_form_id = (SELECT	game_form_id
										FROM	gameset
										WHERE	gameset_id = @gameset)
							AND	is_active = 0
							AND	is_terminated = 0);
-- Check for #Tables and drop if it exist
DROP TABLE	
IF EXISTS #TEMPDB
DROP TABLE	
IF EXISTS #WinAmounts

--Create table with desired win_amounts to pull from
DECLARE @WinAmounts TABLE
(
    ID INT IDENTITY(1,1) PRIMARY KEY,
    AMOUNT DECIMAL(18,2)
);
--These are the input variables used for selecting multipliers. To further reduce user error, only the multiplier needs to be input. The $1.00 denomination is simple because an 18x multiplier is a 18.00 win_amount
--A $0.25 or $3.00 denomination is where user error would most likely come into play if math doesn't come out to win_amount = 4.50 or 54.00. 
INSERT INTO @WinAmounts (AMOUNT) VALUES ((@1stWin)*(@denom)),((@2ndwin)*(@denom)),((@3rdWin)*(@denom)),((@4thWin)*(@denom)),
					((@5thWin)*(@denom)),((@6thWin)*(@denom)),((@7thWin)*(@denom)),((@8thWin)*(@denom)),
					((@9thWin)*(@denom)),((@10thWin)*(@denom)),((@11thWin)*(@denom)),((@12thWin)*(@denom));

SET			@MaxCounter	= (SELECT MAX(ID) FROM @WinAmounts)

--Debug check for gameset_id being used. 
--This is done and printed in messages to mitigate risk and allow the user to see the gameset_id that was found for the UPDATE. 
PRINT 'Using Gameset ' + CAST(@gameset AS VARCHAR(10));

-- Turn off random tickets
-- This is a feature done in the field but is turned off to test on a live environment. 
UPDATE	global_vars
SET	variable_value = 0
WHERE	variable_key = 'use_commingling'

-- Create Temporary RowNumber Table
SELECT 
	ROW_NUMBER() OVER (ORDER BY pulltab_id) AS RowNumber,
	pulltab_tickets.pulltab_id,
	pulltab_tickets.win_amount,
	pulltab_tickets.win_data_json
INTO	#TEMPDB
FROM	pulltab_tickets
JOIN	gameset ON pulltab_tickets.gameset_id = gameset.gameset_id
JOIN	game_form ON game_form.form_id = gameset.game_form_id
JOIN	game ON game.game_id = game_form.game_id

WHERE	game.name = @gameName
AND	game_form.denomination = @denom
AND	has_been_played = 0
AND	game_form.is_active = 1
AND	gameset.is_active = 1

--Increment counter/RowNumber and update next turn to be played
WHILE	(@counter <= @MaxCounter)
	BEGIN
		SELECT @rollingWin = AMOUNT 
		FROM @WinAmounts 
		WHERE ID = @counter
--Check for win_amount EXISTS, RAISEERROR, find next gameset for activation and execution stops
--Display messages for debugging
		IF	
			EXISTS	(SELECT		win_data_json
				FROM		pulltab_tickets
				JOIN		gameset on pulltab_tickets.gameset_id = gameset.gameset_id
				WHERE		win_amount = @rollingWin
				AND		game_form_id = (SELECT	game_form_id
								FROM	gameset
								WHERE	gameset_id = @gameset))
				BEGIN
					DECLARE @NoErrorMessage NVARCHAR(4000) = 
					'Selected win amount of '+CAST(@rollingWin AS VARCHAR(10)) + ' was found';
					PRINT @NoErrorMessage;
				END
		ELSE 	
			IF 
				NOT EXISTS	(SELECT		win_data_json
						FROM		pulltab_tickets
						JOIN		gameset on pulltab_tickets.gameset_id = gameset.gameset_id
						WHERE		win_amount = @rollingWin
						AND		game_form_id = (SELECT	game_form_id
										FROM	gameset
										WHERE	gameset_id = @gameset))
				BEGIN
					DECLARE @ErrorMessage NVARCHAR(4000) = 
					'ERROR: No win amount found for $'+CAST(@rollingWin AS VARCHAR(10)) + ' in Gameset '+CAST(@gameset AS VARCHAR(10)) +' or related game_form. '
					+ CHAR(13) +'Win amount does not exist or win amounts have been overwritten.'
					+ CHAR(13) +'Next gameset_id for $'+CAST(@denom AS VARCHAR(10)) + ' ' + UPPER(@gameName) + ' is ' + CAST(@nextGameset AS VARCHAR(10));
	--Display updated wins that queued before ERROR
					SELECT	*
					FROM	pulltab_tickets
					WHERE	gameset_id = @gameset
					AND		has_been_played = 0;
					PRINT	@ErrorMessage;
					RETURN;
				END;
		
--Pull/Update win_data_json and win_amount from a random ticket with that win_amount and game_form_id
		UPDATE  pulltab_tickets
		SET		win_data_json	=(SELECT TOP(1) win_data_json
						FROM	pulltab_tickets
						JOIN	gameset on pulltab_tickets.gameset_id = gameset.gameset_id
						WHERE	win_amount = @rollingWin
						AND	game_form_id = (SELECT	TOP(1)game_form_id
									FROM	gameset
									WHERE	gameset_id = @gameset)
									ORDER BY NEWID()),
--NEWID() orders the results randomly to increase the variety of the turns that are being tested and the TOP(1) is selected.
				win_amount	=(SELECT TOP(1) win_amount
						FROM	pulltab_tickets
						JOIN	gameset on pulltab_tickets.gameset_id = gameset.gameset_id
						WHERE	win_amount = @rollingWin
						AND	game_form_id = (SELECT	TOP(1)game_form_id
									FROM	gameset
									WHERE	gameset_id = @gameset)
									ORDER BY NEWID())
		FROM	pulltab_tickets 
		WHERE	pulltab_id =(	SELECT pulltab_id
					FROM	#TEMPDB
					WHERE	RowNumber = @counter
					AND	has_been_played = 0)

	SET	@counter = @counter + 1
	END
--VIEW GAMESET AFTER UPDATES
SELECT	*
FROM	pulltab_tickets
WHERE	gameset_id = @gameset
AND	has_been_played = 0

--DROP TEMP TABLE TO REMOVE FUTURE ERRORS
DROP TABLE	#TEMPDB
DROP TABLE	#WinAmounts

END;
`````
<br>

#### Input:


`````sql

--Input the game.name, denomination, and up to 12 multipliers of choice. 
EXEC QueueMultiplier 'FLAMINJOKEROH', 1, 10, 1, 5, 50

`````
With this previous and finalized SQL script, all of our previous issues listed are solved.  
<br>
Targeted issues to solve:
- It didn't eliminate the need to look up the gameset_id using the active gamesets query. 
- It doesn't expedite and solve the initial issue when requested multipliers are 10x, 1x, 5x, and 50x in that specific order. 
- Accidentally inserting the wrong gameset_id can put the turns on another gameset and cause a forced crash resulting in a loss of resources chasing a one time, non existent bug.
- Some turns can win on the same multiplier over 100,000 different ways. Win_data_json determines how the solving will play out but our previous attempts uses the same win_data_json for every turn.

Amended issues solved post functional:
- It doesn't run a check on errors if the win_amount doesn't exist within a gameset.
- If the DB needs to activate a gameset, it tells the next gameset available to be activated. 
























