# CS:GO/CS2 Esports Splunk Dashboard — Query Reference

**Index:** `counterstrike_index`  
**Sourcetypes:** `team_rankings`, `player_stats`, `game_economics`, `match_viewership`, `tournament_results`

---

## Row 1: KPI Tiles

### Panel 1: Total Tournaments
```spl
index=counterstrike_index sourcetype=tournament_results | stats count
```
**Visualization:** Single Value  
**Description:** Count of all tournaments in dataset.

---

### Panel 2: Total Prize Money
```spl
index=counterstrike_index sourcetype=tournament_results | stats sum(prize_pool_usd) as total_prize
```
**Visualization:** Single Value  
**Custom format:** `$` (USD)

---

### Panel 3: Latest Concurrent Players
```spl
index=counterstrike_index sourcetype=game_economics | sort - year_month | head 1 | table year_month avg_concurrent_players
```
**Visualization:** Single Value  
**Description:** Avg concurrent players in most recent month on record.

---

### Panel 4: Current #1 Team
```spl
index=counterstrike_index sourcetype=team_rankings hltv_rank<50 | sort - year_month | dedup team_name | sort hltv_rank | head 1 | table team_name country_iso3 hltv_rank hltv_points
```
**Visualization:** Table  
**Description:** Highest-ranked team (lowest rank number) in curated tier.

---

## Row 2: Player Base & Economy Trends

### Panel 5: Concurrent Players Over Time
```spl
index=counterstrike_index sourcetype=game_economics | timechart span=1mon max(avg_concurrent_players) as "Avg Players" max(peak_concurrent_players) as "Peak Players"
```
**Visualization:** Line Chart  
**Legend:** Two lines, avg vs peak.

---

### Panel 6: Skins Market Value Trend
```spl
index=counterstrike_index sourcetype=game_economics | timechart span=1mon max(estimated_skins_market_usd_m) as "Market (M USD)"
```
**Visualization:** Line Chart  
**Y-Axis Label:** Market Value (Millions USD)

---

### Panel 7: Operation Impact on Player Count
```spl
index=counterstrike_index sourcetype=game_economics | stats avg(avg_concurrent_players) as "Avg Players" by has_active_operation | rename has_active_operation as "Active Operation"
```
**Visualization:** Bar Chart  
**Description:** Compare avg players when operation=1 vs 0.

---

### Panel 8: CS2 Era Transition
```spl
index=counterstrike_index sourcetype=game_economics | timechart span=1mon max(avg_concurrent_players) as avg_players by is_cs2_era | rename is_cs2_era as "CS2 Era"
```
**Visualization:** Line Chart (dual lines)  
**Description:** Two lines: pre-CS2 (0) vs CS2 era (1).

---

## Row 3: Team Rankings (Curated Tier)

### Panel 9: Top 10 Teams (Current)
```spl
index=counterstrike_index sourcetype=team_rankings hltv_rank<30 | sort - year_month | dedup team_name | sort hltv_rank | table hltv_rank team_name country_iso3 hltv_points | head 10
```
**Visualization:** Table  
**Description:** Latest snapshot of top 10 ranked teams.

---

### Panel 10: Rank Trend (Top Teams Over Time)
```spl
index=counterstrike_index sourcetype=team_rankings hltv_rank<10 | timechart span=1mon min(hltv_rank) as rank by team_name limit=20
```
**Visualization:** Line Chart  
**Description:** Rank lines for top 20 teams (lower=better).

---

### Panel 11: Teams by Country (Curated)
```spl
index=counterstrike_index sourcetype=team_rankings hltv_rank<50 | dedup team_name | stats count by country_iso3 | sort - count | head 15
```
**Visualization:** Bar Chart or Geo Map  
**Description:** How many of the top 22 curated teams per country.

---

### Panel 12: Global Team Population (All Tiers)
```spl
index=counterstrike_index sourcetype=team_rankings | timechart span=1mon dc(team_name) as "Active Teams" limit=1
```
**Visualization:** Line Chart  
**Description:** Distinct count of all teams tracked (curated + bulk amateur) per month.

---

## Row 4: Player Stats

### Panel 13: Top 10 Players by Peak Rating
```spl
index=counterstrike_index sourcetype=player_stats | sort - peak_hltv_rating | table player_handle real_name country_iso3 primary_role peak_hltv_rating major_tournament_wins | head 10
```
**Visualization:** Table  
**Description:** Curated pros naturally appear at top (higher rating = better).

---

### Panel 14: Major Tournament Winners Leaderboard
```spl
index=counterstrike_index sourcetype=player_stats major_tournament_wins>0 | sort - major_tournament_wins peak_hltv_rating | table player_handle major_tournament_wins estimated_earnings_usd | head 15
```
**Visualization:** Table  
**Description:** Players with major titles, sorted by count then rating.

---

### Panel 15: Role Distribution
```spl
index=counterstrike_index sourcetype=player_stats | top limit=10 primary_role
```
**Visualization:** Pie Chart or Bar Chart  
**Description:** Count of players per role (Rifler, AWPer, IGL, Entry, Support).

---

### Panel 16: Earnings vs Peak Rating (Elite Players)
```spl
index=counterstrike_index sourcetype=player_stats (major_tournament_wins>0 OR estimated_earnings_usd>100000) | table player_handle peak_hltv_rating estimated_earnings_usd
```
**Visualization:** Scatter Chart  
**X-Axis:** peak_hltv_rating  
**Y-Axis:** estimated_earnings_usd  
**Description:** Correlation: do top-rated players earn more?

---

### Panel 17: Player Nationality Distribution
```spl
index=counterstrike_index sourcetype=player_stats | top limit=20 country_iso3 | rename country_iso3 as Country
```
**Visualization:** Bar Chart  
**Description:** Top 20 countries by player count (includes bulk amateurs).

---

## Row 5: Tournaments & Viewership

### Panel 18: Prize Pool Growth Over Time
```spl
index=counterstrike_index sourcetype=tournament_results | timechart span=1y sum(prize_pool_usd) as "Total Prize Pool"
```
**Visualization:** Line/Area Chart  
**Y-Axis Label:** Prize Pool (USD)

---

### Panel 19: Major vs Tier1 Events per Year
```spl
index=counterstrike_index sourcetype=tournament_results | timechart span=1y count by tier | rename Tier as tier
```
**Visualization:** Stacked Bar or Line Chart  
**Description:** Major (=1) vs Tier1 events over time.

---

### Panel 20: Most Successful Teams (Tournament Wins)
```spl
index=counterstrike_index sourcetype=tournament_results | top limit=15 winner_team | rename winner_team as "Team" count as "Wins"
```
**Visualization:** Bar Chart  
**Description:** Teams with most tournament victories.

---

### Panel 21: MVP Frequency
```spl
index=counterstrike_index sourcetype=tournament_results | top limit=15 mvp_player | rename mvp_player as "Player" count as "MVP Awards"
```
**Visualization:** Bar Chart  
**Description:** Players with most MVP awards across tournaments.

---

### Panel 22: Peak Viewers by Tournament
```spl
index=counterstrike_index sourcetype=tournament_results | sort - peak_viewers_k | table tournament_name year peak_viewers_k winner_team | head 15
```
**Visualization:** Table  
**Description:** Top 15 most-watched tournaments.

---

### Panel 23: Viewership by Broadcast Language
```spl
index=counterstrike_index sourcetype=match_viewership | stats sum(hours_watched_m) as "Total Hours Watched" by language | sort - "Total Hours Watched" | head 10
```
**Visualization:** Bar Chart  
**Description:** Which languages/regions watch most (EN, RU, PT, CN, etc.).

---

### Panel 24: Viewership Trend (Grand Finals)
```spl
index=counterstrike_index sourcetype=match_viewership stage="Grand Final" | timechart span=1y max(peak_viewers_k) as "Peak Viewers"
```
**Visualization:** Line Chart  
**Description:** Peak viewers in Major grand finals per year.

---

## Row 6: Cross-Sourcetype Correlation

### Panel 25: Tournament Winner's Current Rank
```spl
index=counterstrike_index sourcetype=tournament_results is_major=1 
| stats values(winner_team) as winner_team by tournament_id 
| rename winner_team as team_name 
| join type=left team_name [ search index=counterstrike_index sourcetype=team_rankings hltv_rank<50 | sort - year_month | dedup team_name | table team_name hltv_rank hltv_points ] 
| table tournament_id team_name hltv_rank hltv_points 
| sort tournament_id
```
**Visualization:** Table  
**Description:** Major tournament winners + their current HLTV rank (if in top 50).

---

### Panel 26: MVP Player's Career Stats
```spl
index=counterstrike_index sourcetype=tournament_results 
| stats count as "MVP Count" by mvp_player 
| rename mvp_player as player_handle 
| join type=left player_handle [ search index=counterstrike_index sourcetype=player_stats | table player_handle peak_hltv_rating estimated_earnings_usd major_tournament_wins ] 
| table player_handle "MVP Count" peak_hltv_rating estimated_earnings_usd major_tournament_wins 
| sort - "MVP Count"
```
**Visualization:** Table  
**Description:** Players who've won MVPs + their career stats (rating, earnings, major titles).

---

## Quick Tips

- **Filter Curated Data:** Use `hltv_rank<50` (team_rankings) or `major_tournament_wins>0` / `peak_hltv_rating>1.15` (player_stats) to surface meaningful/pro-level entities.
- **Date Extraction:** Most queries use built-in `year_month` or `date` fields; no custom extraction needed.
- **Performance:** Large sourcetypes (team_rankings: 7.8M, player_stats: 1.7M) benefit from `dedup`, `limit`, and `head` to reduce output size.
- **Visualization Types:**
  - Single Value: KPIs
  - Table: Ranked lists, detailed records
  - Line/Area: Trends over time
  - Bar/Pie: Distributions, comparisons
  - Scatter: Correlation
  - Geo Map: Geographic distribution

---

## Dashboard Assembly Notes

Assemble these 26 panels into a Splunk dashboard (classic or Studio). Recommended layout:
- **Row 1 (KPIs):** 4 single-value tiles, side-by-side.
- **Row 2 (Economy):** 4 time-series charts.
- **Row 3 (Teams):** 4 panels, mixed table + trend.
- **Row 4 (Players):** 5 panels, mixed table + scatter + pie.
- **Row 5 (Tournaments):** 7 panels, mixed trend + bar + table.
- **Row 6 (Cross-join):** 2 complex join panels.

Total: 26 panels across 6 rows, all linked to `counterstrike_index`.
