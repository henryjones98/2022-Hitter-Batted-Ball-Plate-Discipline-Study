# 2022-Hitter-Batted-Ball-Plate-Discipline-Study #

##### Under Fangraphs' "Leaders Tab" click "2022" next to "Batting". Click "Export Data" underneath "Statcast NEW!". #####
##### Rename columns "BB%" to "BBperc", "K%" to "Kperc", and "wRC+" to "wRCplus" columns in the Excel Spreadsheet #####
##### Delete the "player id column" from the Excel Spreadsheet #####
##### Upload to My SQL Schema using "Table Data Import Wizard". Title as "baseball.fangraphs leaderboard" #####

ALTER TABLE baseball.`fangraphs leaderboard`
RENAME COLUMN ï»¿"Name" TO Name;

##### Under Baseball Savant's "Leaderboards Tab" click "Exit Velocity and Barrels" underneath "Batting." Click "Download CSV". #####
##### In the CSV, using the "Replace" function in the last_name column, replace all names that say "RamÃ­rez" with "Ramirez", 
##### "GarcÃ­a" with "Garcia", "DÃ­az" with "Diaz", "PeÃ±a" with "Pena", "GimÃ©nez" with "Gimenez", "RodrÃ­guez" with "Rodriguez",
##### "SuÃ¡rez" with "Suarez", "HernÃ¡ndez" with "Herandez", "AcuÃ±a Jr." with "Acuna", "VÃ¡zquez" with "Vazquez", "UrÃ­as" with "Urias",
##### "GonzÃ¡lez" with "Gonzalez", and "SÃ¡nchez" with "Sanchez." #####
##### Next, using the "Replace function in the first_name_column, replace all names that say "JosÃ©" with "Jose", " CÃ©sar" with "Caesar",
##### " AndrÃ©s" with "Andres", " JesÃºs" with "Jesus", " RamÃ³n" with "Ramon", " YoÃ¡n" with "Yoan", " AvisaÃ­l" with "Avisail",
##### and " MartÃ­n" with "Martin". #####
##### Next, Insert a new column to the left of the "last_name" column and title it "Name." In cell A2, type "CONCATENATE(C2, " ", B2)"
##### to merge the first and last name columns. ######
##### Finally, delete the "last_name", "first_name", and "player id" columns from the Excel Spreadsheet ##### 
##### Upload to My SQL Schema using "Table Data Import Wizard". Title as "baseball.exit_velocity" #####

ALTER TABLE baseball.`exit_velocity`
RENAME COLUMN ï»¿Name TO Name;
    
CREATE TABLE FGEVdata(SELECT
	*,
    ROUND((EV.max_hit_speed - EV.avg_hit_speed), 2) AS Potential_Power_Gain, 
    ROUND((xwOBA - wOBA), 3) AS Missed_OBA
FROM baseball.`fangraphs leaderboard` AS FG
INNER JOIN baseball.`exit_velocity` AS EV
ON FG.Name = EV.Name
ORDER BY WAR DESC);

ALTER TABLE FGEVdata
	RENAME COLUMN avg_hit_angle TO Avg_Launch_Angle,
    RENAME COLUMN anglesweetspotpercent TO SweetSpot_Percentage,
    RENAME COLUMN fbld TO FBLD_Ratio,
    RENAME COLUMN gb TO GB,
    RENAME COLUMN ev95plus TO Hard_Hits,
    RENAME COLUMN ev95percent TO HardHit_Percentage,
    RENAME COLUMN barrels TO Barrels,
    RENAME COLUMN brl_percent TO BarrelsperBBE_Percentage,
    RENAME COLUMN brl_pa TO BarrelsperPA_Percentage;

##### Under Baseball Savant's "Leaderboards" Tab, select "Statcast Park Factors", highlight the whole table, and copy it. #####
##### Delete the first row and first column. Next, highlight the entire table to make the theme color background white, 
##### unbolden the text, and make the font Calibri and size 11. Higlight the "Venue" column and click "Remove Hyperlinks". #####
##### In the "Team" Column, rename "Rockies" to "COL", "Reds" to "CIN", "Red Sox" to "BOS", "Angels" to "LAA", 
##### "Phillies" to "PHI", "Royals" to "KCR", "White Sox" to "CHW", "Dodgers" to "LAD", "Orioles" to "BAL", "D-backs"
##### to "ARI", "Pirates" to "PIT", "Brewers" to "MIL", "Giants" to "SFG", "Braves" to "ATL", "Nationals" to "WSN", 
##### "Guardians" to "CLE", "Blue Jays" to "TOR", "Marlins" to "MIA", "Rangers" to "TEX", "Yankees" to "NYY", "Cubs"
##### to "CHC",  "Astros" to "HOU", "Twins" to "MIN", "Tigers" to "DET", "Rays" to "TBR", "Mets" to "NYM", "Cardinals"
##### to "STL", "Athletics" to "OAK", "Padres" to "SDP", and "Mariners" to "SEA". #####
##### Upload to My SQL Schema using "Table Data Import Wizard". Title as "baseball.park factors" #####

ALTER TABLE baseball.`park factors`
RENAME COLUMN ï»¿Name TO Name;

CREATE TABLE FG_EV_PFdata (SELECT *
FROM baseball.`FGEVdata` AS FGEV
INNER JOIN baseball.`park factors` AS PF
ON FGEV.Team = PF.Club);

##### Under Fangraphs' "Leaders Tab" click "2022" next to "Batting". Click the tab
##### that says "Plate Discipline". Click "Export Data" underneath "Statcast NEW!". #####
##### Delete the "player id" column. Next, click on the cell that says "O-Swing %" and hit control + shift + right arrow key, and find and replace
##### all "%"s with "_perc". #####
##### Next, in the excel spreadsheet, rename columns "F-Strike_perc" to "First_Pitch_Strike_perc", "CStr_perc" to "CalledStrike_perc",
##### and "CSW_perc" to "CalledSwingingStrike_perc". #####
##### Upload to My SQL Schema using "Table Data Import Wizard". Title as "baseball.plate discipline" #####

ALTER TABLE baseball.`plate discipline`
RENAME COLUMN ï»¿Name TO Name;

ALTER TABLE baseball.`plate discipline`
RENAME COLUMN Name TO Player,
RENAME COLUMN Team TO Org;

CREATE TABLE FG_EV_PF_PDdata (SELECT *
FROM baseball.`FG_EV_PFdata` AS FGEVPF
INNER JOIN baseball.`plate discipline` AS PD
ON FGEVPF.Name = PD.Player);

##### Export table from My SQL into Excel Spreadsheet. #####
##### Delete duplicate columns of player name and team name and move the "Venue column" 
##### to Column C. Delete all park factor columns, except for the column title "Park Factor". Rename that column to 
##### "Park_Factor".#####
##### Upload Excel Spreadsheet to R Studio. #####

##### Below is the code in R: #####

xwOBA_BB_PF_PDreg <- lm(xwOBA~Attempts+Avg_Launch_Angle+SweetSpot_Percentage+Max_Hit_Speed+Avg_Hit_Speed+Hard_Hits+HardHit_Percentage+Barrels+BarrelsperBBE_Percentage+Park_Factor+Bbperc+Kperc, data=FG_EV_Park_Data)
summary(xwOBA_BB_PFreg)

SweetSpotperc_xWOBA_plot<- plot(x = FG_EV_PF_PD_Data$SweetSpot_Percentage, y = FG_EV_PF_PD_Data$xwOBA,
     xlab = "Sweet Spot %",
     ylab = "xwOBA",
     xlim = c(20, 45),
     ylim = c(.260, .470),
     abline(lm(FG_EV_PF_PD_Data$xwOBA ~ FG_EV_PF_PD_Data$SweetSpot_Percentage), col = "Blue"))

Walk_perc_xWOBA_plot<- plot(x = FG_EV_PF_PD_Data$BB, y = FG_EV_PF_PD_Data$xwOBA, 
                               xlab = "BB %",
                               ylab = "xwOBA",
                               xlim = c(0, 20),
                               ylim = c(.260, .470),
                               abline(lm(FG_EV_PF_PD_Data$xwOBA ~ FG_EV_PF_PD_Data$BB), col = "Blue"))

BBperc_PDreg <- lm(BB~O_Swing_perc+Z_Swing_perc+Swing_perc+O_Contact_perc+Z_Contact_perc+Contact_perc+Zone_perc+Firstpitch_Strike_perc+SwStr_perc+CalledStrike_perc, data=FG_EV_PF_PD_Data)
summary(BBperc_PDreg)

BBperc_Swing_Discipline_reg <- lm(BB~O_Swing_perc+Z_Swing_perc+Swing_perc+O_Contact_perc+Z_Contact_perc+Contact_perc+SwStr_perc, data=FG_EV_PF_PD_Data)
summary(BBperc_Swing_Discipline_reg)

##### Back into SQL Code #####

SELECT 
    ROUND(AVG(xwOBA), 3) AS "Avg xwOBA",
    ROUND(AVG(wOBA), 3) AS "Avg wOBA",
	ROUND(AVG(SweetSpot_Percentage), 1) AS "Avg Sweet Spot %",
    ROUND(AVG(Z_Swing_perc), 1) AS "Avg In-Zone Swing %",
    ROUND(AVG(Swing_perc), 1) As "Avg Swing %",
    ROUND(AVG(CalledStrike_perc), 1) AS "Avg Called-Strike %"
FROM baseball.`fg_ev_pf_pd data`;

SELECT
	Name,
    Team,
    WAR,
    AVG,
    OBP,
    SLG,
    Avg_Launch_Angle AS "Avg Launch Angle",
    SweetSpot_Percentage AS "Sweet Spot %",
    Bbperc AS "BB %",
    Kperc AS "K %",
    wOBA,
    xwOBA,
    Missed_OBA AS "Missed OBA"
FROM baseball.`fg_ev_pf_pd data`
WHERE 
    SweetSpot_Percentage > 34.2 AND
    Missed_OBA > 0
ORDER BY Missed_OBA DESC
LIMIT 10; 

SELECT
	Name,
    Team,
    WAR,
    AVG,
    OBP,
    SLG,
    Avg_Launch_Angle AS "Avg Launch Angle",
    SweetSpot_Percentage AS "Sweet Spot %",
    Bbperc AS "BB %",
    Kperc AS "K %",
    wOBA,
    xwOBA,
    Missed_OBA AS "Missed OBA"
FROM baseball.`fg_ev_pf_pd data`
WHERE 
    SweetSpot_Percentage < 34.2 AND 
    Missed_OBA < 0 
ORDER BY Missed_OBA ASC
LIMIT 10; 

SELECT
	Name,
    Team,
    WAR,
    AVG,
    OBP,
    SLG,
    Bbperc AS "BB %",
    Kperc AS "K %",
    Swing_perc AS "Swing %",
    Z_Swing_perc AS "In-Zone Swing %",
    CalledStrike_perc AS "Called-Strike %",
    wOBA,
    xwOBA,
    Missed_OBA AS "Missed OBA"
FROM (
SELECT *
FROM baseball.`fg_ev_pf_pd data`
WHERE Missed_OBA > 0) AS Overlooked_Plate_Discipline_Hitters
WHERE 
    Z_Swing_perc > 69.5 OR
    CalledStrike_perc < 16.2 OR
    Swing_perc < 47.5 
ORDER BY Missed_OBA DESC
LIMIT 10; 

SELECT
	Name,
    Team,
    WAR,
    AVG,
    OBP,
    SLG,
    Bbperc AS "BB %",
    Kperc AS "K %",
    Swing_perc AS "Swing %",
    Z_Swing_perc AS "In-Zone Swing %",
    CalledStrike_perc AS "Called-Strike %",
    wOBA,
    xwOBA,
    Missed_OBA AS "Missed OBA"
FROM (
SELECT *
FROM baseball.`fg_ev_pf_pd data`
WHERE Missed_OBA < 0) AS Overperformed_Plate_Discipline_Hitters
WHERE 
    Z_Swing_perc < 69.5 OR
    CalledStrike_perc > 16.2 OR
    Swing_perc > 47.5 
ORDER BY Missed_OBA ASC
LIMIT 10;
