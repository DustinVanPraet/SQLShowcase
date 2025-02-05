## Improving the SELECT/UPDATE processes in the database to improve testing practices

### This will show the growth and changes that were made and how the position has become more efficient

Within the gambling industry for casino slot machine games, all states have different regualtions and requirements. Some of these states require a record of every turn played by every machine and this is recorded in a SQL server database for each individual location which may have many different machines that can play many different games. 
<br>
<br>
In the Quality Assurance(QA) department, all posible multipliers and seeds need to be tested across each individual game to experience and test different features that may not occur but 1 time out of 50,000 turns played. To circumvent this issue while also testing in a Live environment, we will write UPDATE statements to manipulate records to force low chance turns into playing when the terminal reaches out to the database for a turn. 
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
A member of the QA team is requesting a specific order of turns for testing. They want to test the different multipliers and how they react when played in that specific order. The QA member asks for a 10x, 2x, 40x, 0x, and 100x for the game on the $1.00 denomination in that specific order. 

### Input:
```sql
SELECT  gameset.gameset_id,
		    gameset.current_tickets_played,
		    gameset.game_form_id,
		    gameset.current_total_won,
		    game_form.denomination,
		    game.name
	
FROM		gameset
JOIN    game_form ON gameset.game_form_id=game_form.form_id
JOIN    game ON game_form.game_id=game.game_id
JOIN    store ON gameset.store_id=store.store_id
JOIN    pulltab_tickets ON gameset.gameset_id = pulltab_tickets.gameset_id

WHERE		gameset.is_active = 1 
AND     game_form.is_active = 1
	
GROUP BY	gameset.gameset_id,
  				gameset.current_tickets_played,
	  			gameset.game_form_id,
		  		gameset.current_total_won,
			  	game_form.denomination,
				  game.name
```
### Output
![ActiveGamesetsOutput2](https://github.com/user-attachments/assets/8348aa62-939b-47f8-9c1a-d2048eeec0e6)
