# NBA_attendance

# NBA Attendance Analytics & Self-Updating Dashboard (Tableau + Azure SQL)

Proof-of-concept (POC) BI solution for NBA team business ops & ticketing: centralizes historical game-level attendance data in the cloud, lets staff add new games through a form, and surfaces insights via a Tableau dashboard + storyboard.

## What this does
For any selected NBA team, the dashboard helps answer:
- How average **home** and **away** attendance changes by **season**
- Which **opponents** drive above/below-average attendance (as **% change vs. that team’s baseline**)
- How attendance patterns relate to **season wins**
- How newly entered games compare once they flow into the database 

---

##Dataset:
https://www.kaggle.com/datasets/eoinamoore/historical-nba-data-and-player-box-scores?select=PlayerStatistics.csv

## Architecture (end-to-end)
This project is intentionally “self-updating”:

1) **Azure SQL (db30)** stores data in a normalized structure:
   - `Games`
   - `TeamHistories`
   - `FormInput` 

2) **Microsoft Forms + Power Automate**
   - https://forms.office.com/r/m10eyAi2cN
   - User enters: home team, away team, date, attendance, and winner
   - Flow writes the row to `FormInput` and derives a consistent `WinnerTeamName` 

3) **Tableau Desktop**
   - https://public.tableau.com/views/nbaproject_17651568642560/NBAAttendanceDashboard?:language=en-US&publish=yes&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link 
   - Uses **Custom SQL** to union historical games + form-entered games into a single “Games+” source
   - Historical winner IDs are translated to `WinnerTeamName` via `TeamHistories`; form rows already store `WinnerTeamName`
  
4) **Tableau Storyboard**
   - https://public.tableau.com/app/profile/jonah.jutzi/viz/nbaproject_17651568642560/NBAAttendanceStoryboard
   - Guides users through the process of selecting KPIs and creating the final dashboard

---

## Data scope
- **Regular-season NBA vs. NBA games only** (excludes preseason/playoffs and non-NBA opponents)
- **Dates:** '03 Season - Current
- **Missing attendance** games excluded
- Seasons labeled like `'03-04' ... '24-25'` and defined as **Oct–Apr**

---

## Key calculations (Tableau)
- **Home Avg:** mean home attendance for the selected team (baseline)
- **% Change in Attendance:**  
  `(Avg attendance vs opponent – Home Avg) / Home Avg` 
- **Season Wins:** count of games where `WinnerTeamName = Team Selector`

---

## What you’ll see in Tableau
Built in Tableau Desktop:
- Season-level **home/away attendance** histogram
- Opponent tables with **avg attendance**, **home average**, and **% change vs baseline**
- A **Season Wins** heatmap as performance context

---

## How to use (viewer workflow)
### Open and refresh the dashboard
1. Open the packaged Tableau workbook (`.twbx`)
2. Connect/login to the Azure SQL database (`db30`)
      #Note: requires UW login info
4. In Tableau: **Data → Refresh All Extracts** (or Refresh if live) 

### Add a new game (self-updating loop)
1. Open the Microsoft Form
2. Enter: home team, away team, date, attendance, winner
3. Submit → Power Automate writes into `dbo.FormInput`
4. Refresh Tableau → the new game appears in the dashboard
   # OR alternatively, pull data from Kaggle again


