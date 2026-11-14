# Batting-Data-Analysis

# ============================================================
# PROJECT BASED LEARNING
# DATA ANALYSIS - MATCH RESULTS DATASET
# GOOGLE COLAB SOURCE CODE
# ============================================================

import pandas as pd
import numpy as np
from google.colab import files
from IPython.display import display

# ------------------------------------------------------------
# 1. UPLOAD CSV FILE
# ------------------------------------------------------------

print("Please upload your Batting.csv file")

import io
import pandas as pd
from google.colab import files

# 1. Trigger the upload button and select Batting.csv
uploaded = files.upload()

# 2. Read the file directly using its exact name
df = pd.read_csv(io.BytesIO(uploaded['Batting.csv']))

# 3. Preview the first 5 rows to confirm it loaded
df.head()


# Clean column names
df.columns = df.columns.str.strip()

# Check required columns
required = [
    'Date', 'Team', 'Vs', 'Player',
    'R', 'B', '4s', '6s', 'SR', 'Chase', 'Win'
]

missing = [col for col in required if col not in df.columns]

if missing:
    raise ValueError(f"Missing columns: {missing}")

# Convert numerical columns
numeric_cols = ['R', 'B', '4s', '6s', 'SR', 'Chase', 'Win']

for col in numeric_cols:
    df[col] = pd.to_numeric(df[col], errors='coerce').fillna(0)

print("\nDataset Loaded Successfully!")
print("Total Records:", len(df))
print("Total Players:", df['Player'].nunique())
print("\nDataset Preview:")
display(df.head())


# Dictionary to store all answers
results = {}


# ------------------------------------------------------------
# QUESTION 1: TOP 10 BATSMEN BY TOTAL RUNS
# ------------------------------------------------------------

print("\n========== QUESTION 1 ==========")

players = df.groupby('Player').agg(
    Total_Runs=('R', 'sum'),
    Total_Balls=('B', 'sum'),
    Total_Fours=('4s', 'sum'),
    Total_Sixes=('6s', 'sum')
).reset_index()

players['Strike_Rate'] = np.where(
    players['Total_Balls'] > 0,
    players['Total_Runs'] / players['Total_Balls'] * 100,
    0
)

q1 = players.sort_values(
    'Total_Runs', ascending=False
).head(10)

results['Q1_Top_Batsmen'] = q1

print("Top 10 Batsmen:")
display(q1)


# ------------------------------------------------------------
# QUESTION 2: TOP 10 STRIKE RATES (MINIMUM 50 BALLS)
# ------------------------------------------------------------

print("\n========== QUESTION 2 ==========")

qualified = players[players['Total_Balls'] >= 50]

q2 = qualified.sort_values(
    'Strike_Rate', ascending=False
).head(10)

results['Q2_Top_Strike_Rates'] = q2

print("Top 10 Strike Rates:")
display(q2)


# ------------------------------------------------------------
# QUESTION 3: MOST FOURS AND SIXES
# ------------------------------------------------------------

print("\n========== QUESTION 3 ==========")

most_fours = players.loc[
    players['Total_Fours'].idxmax()
]

most_sixes = players.loc[
    players['Total_Sixes'].idxmax()
]

print("Player with Most Fours:")
print(most_fours[['Player', 'Total_Fours']])

print("\nPlayer with Most Sixes:")
print(most_sixes[['Player', 'Total_Sixes']])

results['Q3_Boundary_Statistics'] = players.sort_values(
    'Total_Fours', ascending=False
)


# ------------------------------------------------------------
# QUESTION 4: TEAM WITH HIGHEST NUMBER OF WINS
# ------------------------------------------------------------

print("\n========== QUESTION 4 ==========")

team_matches = df.groupby(
    ['Date', 'Team', 'Vs'], as_index=False
).agg(
    Win=('Win', 'max')
)

q4 = team_matches.groupby('Team').agg(
    Total_Wins=('Win', 'sum'),
    Total_Matches=('Win', 'count')
).reset_index()

q4 = q4.sort_values(
    'Total_Wins', ascending=False
)

results['Q4_Team_Wins'] = q4

print("Team-wise Wins:")
display(q4)

print("\nHighest Winning Team:")
display(q4.head(1))


# ------------------------------------------------------------
# QUESTION 5: HIGHEST RUN SCORER IN EACH MATCH
# ------------------------------------------------------------

print("\n========== QUESTION 5 ==========")

# Create a common fixture identifier for both teams
match_data = df.copy()

match_data['Team_1'] = match_data[
    ['Team', 'Vs']
].apply(
    lambda row: min(row.astype(str)), axis=1
)

match_data['Team_2'] = match_data[
    ['Team', 'Vs']
].apply(
    lambda row: max(row.astype(str)), axis=1
)

# Find the highest individual score in each fixture
idx = match_data.groupby(
    ['Date', 'Team_1', 'Team_2']
)['R'].idxmax()

q5 = match_data.loc[
    idx,
    ['Date', 'Team_1', 'Team_2', 'Player', 'Team', 'R']
].copy()

q5 = q5.rename(columns={
    'Player': 'Highest_Scorer',
    'Team': 'Scorer_Team',
    'R': 'Runs_Scored'
})

q5 = q5.sort_values(['Date', 'Team_1', 'Team_2'])

results['Q5_Match_Highest_Scorer'] = q5

print("Highest Scorer in Each Match:")
display(q5)


# ------------------------------------------------------------
# QUESTION 6: BATTING FIRST VS CHASING
# ------------------------------------------------------------

print("\n========== QUESTION 6 ==========")

team_innings = df.groupby(
    ['Date', 'Team', 'Vs', 'Chase'],
    as_index=False
).agg(
    Team_Runs=('R', 'sum'),
    Win=('Win', 'max')
)

q6 = team_innings.groupby('Chase').agg(
    Total_Innings=('Team_Runs', 'count'),
    Average_Runs=('Team_Runs', 'mean'),
    Total_Wins=('Win', 'sum'),
    Win_Rate=('Win', 'mean')
).reset_index()

q6['Situation'] = q6['Chase'].map({
    0: 'Batting First',
    1: 'Chasing'
})

q6['Win_Rate_Percentage'] = q6['Win_Rate'] * 100

results['Q6_Batting_First_vs_Chasing'] = q6

print("Performance Comparison:")
display(q6)


# ------------------------------------------------------------
# QUESTION 7: TOP 10 BOWLERS
# ------------------------------------------------------------

print("\n========== QUESTION 7 ==========")

print("""
Bowling data is not available in Batting.csv.

Required columns:
1. Bowler
2. Overs
3. Runs Conceded
4. Wickets

Economy Rate = Runs Conceded / Overs Bowled

Upload a bowling dataset to calculate this question.
""")


# ------------------------------------------------------------
# QUESTION 8: MOST RUNS THROUGH BOUNDARIES
# ------------------------------------------------------------

print("\n========== QUESTION 8 ==========")

q8 = players.copy()

q8['Boundary_Runs'] = (
    q8['Total_Fours'] * 4 +
    q8['Total_Sixes'] * 6
)

q8 = q8.sort_values(
    'Boundary_Runs', ascending=False
).head(10)

results['Q8_Boundary_Runs'] = q8

print("Top 10 Boundary Scorers:")
display(q8)


# ------------------------------------------------------------
# QUESTION 9: PLAYER PERFORMANCE SCORE
# ------------------------------------------------------------

print("\n========== QUESTION 9 ==========")

score_data = players.copy()

score_data['Boundary_Runs'] = (
    score_data['Total_Fours'] * 4 +
    score_data['Total_Sixes'] * 6
)

# Min-Max Normalization
features = [
    'Total_Runs',
    'Strike_Rate',
    'Boundary_Runs',
    'Total_Balls'
]

for col in features:

    minimum = score_data[col].min()
    maximum = score_data[col].max()

    if maximum == minimum:
        score_data[col + '_Normalized'] = 0
    else:
        score_data[col + '_Normalized'] = (
            (score_data[col] - minimum) /
            (maximum - minimum)
        ) * 100

# Weighted Performance Score
score_data['Performance_Score'] = (
    0.35 * score_data['Total_Runs_Normalized'] +
    0.25 * score_data['Strike_Rate_Normalized'] +
    0.25 * score_data['Boundary_Runs_Normalized'] +
    0.15 * score_data['Total_Balls_Normalized']
)

q9 = score_data.sort_values(
    'Performance_Score', ascending=False
).head(20)

results['Q9_Top_20_Performance'] = q9

print("Top 20 Players by Performance Score:")
display(q9)


# ------------------------------------------------------------
# QUESTION 10: TOP 10 ALL-ROUND PERFORMERS
# ------------------------------------------------------------

print("\n========== QUESTION 10 ==========")

print("""
A complete all-rounder analysis requires bowling statistics.

Required information:
1. Total Runs
2. Strike Rate
3. Total Wickets
4. Bowling Economy
5. Boundary Runs
6. Match Results

The current batting dataset does not contain bowling statistics.
Therefore, an accurate all-rounder score cannot be calculated.
""")


# ------------------------------------------------------------
# SAVE ALL ANSWERS IN ONE EXCEL WORKBOOK
# ------------------------------------------------------------

print("\n========== EXPORTING RESULTS ==========")

output_file = 'Match_Results_Analysis.xlsx'

with pd.ExcelWriter(output_file, engine='openpyxl') as writer:

    for sheet_name, result_df in results.items():

        result_df.to_excel(
            writer,
            sheet_name=sheet_name[:31],
            index=False
        )

print("Excel Workbook Created Successfully!")

# Download Excel workbook
files.download(output_file)

print("\n========== ANALYSIS COMPLETED ==========")
print("All available answers have been calculated.")
