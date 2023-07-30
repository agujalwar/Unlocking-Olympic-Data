Queries Solutions

1. How many olympics games have been held?

	select count(DISTINCT Games) from olympics_history

2. List down all Olympics games held so far. Order the result by year.
	
	select DISTINCT Year, Season, City
	from olympics_history
	order by Year

3. Mention the total number of nations who participated in each olympics game?. Order the results by games.
 
(Hint: You can group the data by games and region, and then count the number of unique regions in each game to determine the total number of nations participating. Finally, sort the results by games.)

	SELECT oh.Games, count(distinct ohnr.region)
	FROM olympics_history oh
	JOIN olympics_history_noc_regions ohnr
	on oh.NOC=ohnr.NOC
	GROUP by oh.Games
	order by oh.Games

4. Which nation has participated in all of the olympic games? and order the output by first column which is fetched
 
(Hint: To find the nation that has participated in all Olympic games, you can calculate the total number of distinct games in the dataset and count the number of games each country has participated in. Then, join these two subqueries based on the total number of games. Finally, sort the output by the first column fetched.)

	SELECT olympics_history_noc_regions.region, COUNT(DISTINCT olympics_history.games) AS total_games_participated
	FROM olympics_history
	JOIN olympics_history_noc_regions ON olympics_history.noc = olympics_history_noc_regions.noc
	GROUP BY olympics_history_noc_regions.region
	HAVING total_games_participated = (SELECT COUNT(DISTINCT games) FROM olympics_history)
	ORDER BY olympics_history_noc_regions.region;

5. How many unique athletes have won a gold medal in the Olympics?

	SELECT COUNT(DISTINCT Name) 
	FROM olympics_history
	WHERE Medal='Gold'

6. Which Sports were just played only once in the olympics? and Order the output by Sports. output should include number of games.
 
(Hint: To find the sports that were played only once in the Olympics, you can start by selecting distinct combinations of games and sports. Then, in a separate subquery, count the number of games each sport was played. Join these two subqueries based on the sport column and filter for sports with a count of 1. Finally, sort the output by the sport column.)

	SELECT olympics_history.sport, COUNT(DISTINCT olympics_history.games) AS num_games, GROUP_CONCAT(DISTINCT olympics_history.games ORDER BY 	olympics_history.games) AS games
	FROM olympics_history
	GROUP BY olympics_history.sport
	HAVING num_games = 1
	ORDER BY olympics_history.sport;

7. Fetch the total number of sports played in each olympic games. Order by no of sports by descending.
 
(Hint: To determine the total number of sports played in each Olympic game, you can start by selecting distinct combinations of games and sports. Then, in a separate subquery, count the number of sports for each game. Finally, sort the output by the number of sports in descending order.)

	SELECT Games,COUNT(DISTINCT Sport)
	FROM olympics_history
	GROUP by Games
	ORDER by COUNT(Sport) DESC

8. Fetch oldest athlete to win a gold medal

	SELECT name, sex, age, team, games, city, sport, event, medal, rnk
	FROM (
    	SELECT name, sex, age, team, games, city, sport, event, medal, RANK() OVER (ORDER BY age DESC) AS rnk
    	FROM olympics_history
    	WHERE medal = 'Gold'
	) AS ranked_athletes
	WHERE rnk = 1;

9. Top 5 athletes who have won the most gold medals. Order the results by gold medals in descending.
 
(Hint: Create a temporary table counting the total number of gold medals for each athlete. Rank the athletes using dense_rank(). Select the top 5 athletes based on their rank, ordering them by the total gold medals in descending order.)

	WITH athlete_gold_count AS (
  	SELECT
    	name,
    	team,
    	COUNT(*) AS total_gold_medals
  	FROM
    	olympics_history
  	WHERE
    	medal = 'Gold'
  	GROUP BY
    	name, team
	)
	SELECT
  	name,
  	team,
  	total_gold_medals
	FROM
  	(
    	SELECT
      	name,
      	team,
      	total_gold_medals,
      	DENSE_RANK() OVER (ORDER BY total_gold_medals DESC) AS ranking
    	FROM
      	athlete_gold_count
  	) AS ranked_athletes
	WHERE
  	ranking <= 5
	ORDER BY
  	total_gold_medals DESC
  	LIMIT 5;

10. Top 5 athletes who have won the most medals (gold/silver/bronze). Order the results by medals in descending.
 
(Hint: Create a temporary table counting the total number of medals for each athlete. Rank the athletes using dense_rank(). Select the top 5 athletes based on their rank, ordering them by the total number of medals in descending order.)
	WITH athlete_gold_count AS (
  	SELECT
    	name,
    	team,
    	COUNT(*) AS total_gold_medals
  	FROM
    	olympics_history
  	WHERE
    	medal != 'Medal-less'
  	GROUP BY
    	name, team
	)
	SELECT
  	name,
  	team,
  	total_gold_medals
	FROM
  	(
    	SELECT
      	name,
      	team,
      	total_gold_medals,
      	DENSE_RANK() OVER (ORDER BY total_gold_medals DESC) AS ranking
    	FROM
      	athlete_gold_count
  	) AS ranked_athletes
	WHERE
  	ranking <= 5
	ORDER BY
  	total_gold_medals DESC
  	LIMIT 5;
	--------------------------------

	WITH athlete_count AS (
	SELECT
	name,
	team,
	COUNT(*) AS total_medals
	FROM
	olympics_history
	WHERE
	medal != 'Medal-less'
	GROUP BY
	name, team
	)
	SELECT
	name,
	team,
	total_medals
	FROM
	(
	SELECT
	name,
	team,
	total_medals,
	DENSE_RANK() OVER (ORDER BY total_medals DESC) AS ranking
	FROM
	athlete_count
	) AS ranked_athletes
	WHERE
	ranking <= 5
	ORDER BY
	total_medals DESC
	LIMIT 5;
	
11. Top 5 most successful countries in olympics. Success is defined by no of medals won.
 
(Hint: Create a temporary table counting the total number of medals for each country. Rank the countries using dense_rank(). Select the top 5 countries based on their rank, ordering them by the total number of medals in descending order.)


	WITH athlete_count AS (
  	SELECT
    	ohnc.region as country,
    	COUNT(*) AS total_medals
 	FROM
    	olympics_history as oh 
    	JOIN olympics_history_noc_regions ohnc 
    	on oh.NOC=ohnc.NOC
  	WHERE
    	medal != 'Medal-less'
  	GROUP BY
    	ohnc.region
	)

	SELECT
  	country,
  	total_medals,
  	ranking
	FROM
  	(
    	SELECT
      	country,
      	total_medals,
      	DENSE_RANK() OVER (ORDER BY total_medals DESC) AS ranking
   	  FROM
      	athlete_count
  	) 	AS ranked_athletes
	WHERE
  	ranking <= 5
	ORDER BY
  	total_medals DESC

12. In which Sport/event, India has won highest medals.
 
(Hint: Create a temporary table counting the total number of medals for each sport/event where India has won. Rank the sport/events using rank(). Select the sport/event with the highest rank (1) from the temporary table.)

	 WITH medal_count AS
 	(
	SELECT 
     	team,
     	Sport,
     	COUNT(Medal) as total_medals
	FROM olympics_history 
	WHERE Medal != "Medal-less" and Team = 'India' 
	GROUP by Team,Sport
     	)
     
     	SELECT 
     	sport,
     	total_medals
     	FROM
     	(
         SELECT
         sport,
         total_medals,
         RANK() OVER (ORDER BY total_medals DESC) as ranking
         
         FROM medal_count
     	) AS ranked_athletes
     
     	where ranking = 1
     	ORDER by total_medals


13. Break down all olympic games where india won medal for Hockey and how many medals in each olympic games and order the result by no of medals in descending.

	SELECT Team,Sport,Games,COUNT(Medal) as total_medal
	FROM olympics_history 
	WHERE Medal != "Medal-less" and Team = 'India' AND Sport='Hockey' 
	GROUP by Team,Sport,Games
	ORDER by COUNT(Medal) DESC
               
               
               
               


































































