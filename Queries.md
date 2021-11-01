# SQL Queries for Project 1 - GameCompany


## Number of players joined, retained, and fractional retention

```sql
SELECT 
  joined, 
  COUNT(player_id) as numPlayersJoined, 
  SUM(players_retained) as numPlayersRetained, 
  ROUND(SUM(players_retained) / COUNT(player_id), 2) as fractional_retention 
FROM 
  (
    SELECT 
      ma_info.player_id, 
      joined, 
      MAX(day) as last_day_played, 
      CASE 
        WHEN (MAX(day)- joined) > 30 THEN 1 
        ELSE 0 
      END as players_retained, 
    FROM 
      `class-5-bigquery.gamecompanydata.matches_info` AS ma_info 
      JOIN `class-5-bigquery.gamecompanydata.player_info` AS pl_info 
      ON pl_info.player_id = ma_info.player_id 
    GROUP BY ma_info.player_id, joined
  ) 
WHERE joined <= 334 
GROUP BY joined
```

## Average sales data for retained and unretained players

```sql
SELECT 
  ROUND(AVG(sum_item_prices),2) as average_sales, 
  players_retained, 
FROM 
  (
    SELECT
      ROUND(SUM(it_info.price), 2) as sum_item_prices, 
      CASE 
        WHEN (MAX(ma_info.day)- joined) > 30 THEN 1 
        ELSE 0 
      END as players_retained 
    FROM 
      `class-5-bigquery.gamecompanydata.matches_info` AS ma_info 
      JOIN `class-5-bigquery.gamecompanydata.player_info` AS pl_info 
      ON pl_info.player_id = ma_info.player_id 
      JOIN `class-5-bigquery.gamecompanydata.purchase_info` AS pu_info 
      ON pu_info.player_id = pl_info.player_id 
      JOIN `class-5-bigquery.gamecompanydata.item_info` AS it_info 
      ON it_info.item_id = pu_info.item_id 
    GROUP BY 
      pl_info.player_id, 
      joined 
    ORDER BY 
      joined
  ) 
GROUP BY 
  players_retained
```

## System played for unretained and retained players

```sql
SELECT
    system,
    players_retained,
    COUNT(player_id) as numPlayersPerSystem
FROM(
SELECT
    system,
    pl_info.player_id,
    CASE 
        WHEN (MAX(ma_info.day)-joined) > 30 THEN 1
        ELSE 0
    END as players_retained
FROM `class-5-bigquery.gamecompanydata.matches_info` AS ma_info
JOIN `class-5-bigquery.gamecompanydata.player_info` AS pl_info
ON pl_info.player_id = ma_info.player_id
JOIN `class-5-bigquery.gamecompanydata.purchase_info` AS pu_info
ON pu_info.player_id = pl_info.player_id
JOIN `class-5-bigquery.gamecompanydata.item_info` AS it_info
ON it_info.item_id = pu_info.item_id
GROUP BY system, pl_info.player_id, joined
)
GROUP BY system, players_retained
ORDER BY system, numPlayersPerSystem
```
