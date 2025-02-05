## Improving the SELECT/UPDATE processes in the database to improve testing practices

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
In this example, a member of the QA team is requesting a specific order of turns for testing. They want to test the different multipliers and how they react when played in that specific order. The QA member asks for a 10x, 2x, 40x, 0x, and 100x for FLAMINJOKEROH on the $1.00 denomination in that specific order. 

#### Input:
```sql
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
#### Output
![ActiveGamesetsOutput2](https://github.com/user-attachments/assets/8348aa62-939b-47f8-9c1a-d2048eeec0e6)
<br>
The database analyst would then manually search for the gameset_id that aligned with the denomination and name that they were looking for. This would be gameset_id 1482.
The next step would be to search for a win_amount containing 10x for $1.00 denomination which would be 10.00. The next query would be written to find this information:
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

















