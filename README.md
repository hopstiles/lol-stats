# LoL ranked dashboards (Google Colab)

A notebook that pulls your League of Legends ranked games and shows two interactive dashboards right in Colab:

- **Ranked review**: record and rank over time, win rate by deaths, first objectives, game length, role, champion and time of day; gold and deaths minute by minute; objective presence and deaths before objectives; and a short list of highlights worked out from your own games.
- **Champions & matchups**: click any champion you played (or faced) to see your stats, every lane matchup (in bot, the enemy ADC, the enemy support and your lane partner separately), and a gold-vs-opponent timeline for each game with your kills, deaths, assists and every objective marked.

Both are also saved as standalone `.html` files (`ranked_review.html`, `champion_matchups.html`) you can download and open in any browser.

## Run it

1. Open `lol_dashboards.ipynb` in Colab (on GitHub, open the file and use **Open in Colab**, or go to colab.research.google.com → GitHub tab and paste the repo URL).
2. Get a development API key at [developer.riotgames.com](https://developer.riotgames.com). It expires every 24 hours.
3. In Colab, open **Secrets** (key icon on the left), add `RIOT_API_KEY` with your key, and turn on **Notebook access**.
4. In the first code cell, set `GAME_NAME`, `TAG_LINE`, `REGION` and `TIMEZONE`.
5. **Runtime → Run all.**

The Riot pull takes roughly 2 minutes per 50 games because of the development key's rate limit (100 requests per 2 minutes, 2 requests per game).

## What each step does

| Step | Needs | Writes |
|---|---|---|
| 1. Riot pull | API key | `match_players_summary.csv`, `all_players_minute_frames.csv`, `all_matches_events_log.csv`, `complete_raw_match_dumps.json.gz` |
| 2. OP.GG pull (optional) | nothing | `all_matches_full_details.json`, `<name>_season_matches.csv` |
| 3. Dashboard code | — | `lol_process.py`, `lol_render.py`, two page templates |
| 4. Build + show | Step 1 and/or 2 | `dashboard_data/*.csv`, the two `.html` dashboards |

Without the OP.GG export, the rank, OP Score and session sections are hidden. Without the Riot export, only those OP.GG sections show (and the matchups dashboard is skipped).

## Matchup stars

`matchup_ratings.csv` (included here) marks favored matchups with ★ (2+ point win-rate edge) or ★★ (4+). It covers every champion in each role (Top, Jungle, Mid, ADC, Support), about 9,500 matchups from lolalytics.com's counters pages, last 30 days, Emerald+, 200+ games. Put it in the data folder (`/content` by default) next to your exports and the stars show up for anyone's games.

```
role,champion,enemy,edge,games,patch,read_from
Top,Mordekaiser,Illaoi,5.52,5776,30d,Mordekaiser counters page
Support,Zilean,Swain,3.42,2004,30d,Zilean counters page
```

`edge` is the champion's opponent-adjusted win-rate edge in points (positive = favored). Bot ratings compare same roles only (ADC vs ADC, support vs support). To refresh it, re-read the counters pages and keep the same columns.


## Notes

- Ranked solo/duo only by default (`RANKED_SOLO_ONLY = True`); remakes and games under 5 minutes are left out.
- "Near the objective" uses Riot's once-a-minute positions, so it means where you were 0–60 seconds before the kill.
- OP.GG data comes from the unofficial `opgg.py` library and can break if OP.GG changes its site.
