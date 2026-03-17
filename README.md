# **League of Legends - Game Analysis and Role Prediction**

**By: Victor Picorelli**  
**DSC 80** Final Project  
University of California, San Diego 

---


## Sections
- [Overview](#overview)
- [Introduction](#introduction)
- [Data Cleaning and Exploratory Data Analysis](#data-cleaning-and-exploratory-data-analysis)
- [Assessment of Missingness](#assessment-of-missingness)
- [Hypothesis Testing](#hypothesis-testing)
- [Framing a Prediction Problem](#framing-a-prediction-problem)
- [Baseline Model](#baseline-model)
- [Final Model](#final-model)
- [Fairness Analysis](#fairness-analysis)


---

## Overview

- This project explores professional League of Legends match data to better understand what actually drives a team's success.  
- We analyze how early-game advantage, macro decisions, and player performance impact win/loss outcomes.  
- Additionally, we build a model to predict a player's role based on in-game statistics.

---

## Introduction

### General Introduction

League of Legends(LOL) is a competitive multiplayer online battle arena (MOBA) game developed by Riot Games, where two teams of five players compete to destroy the enemy base. Each player takes on a specific role within the team, and success depends on a combination of strategy, coordination, and individual performance.

Games are typically divided into different phases:

- **Early Game**: The first minutes of the match, where players focus on gaining small advantages such as gold, experience, and lane pressure.  
- **Mid/Late Game**: Phases where team coordination becomes more important, especially around objectives like dragons, towers, and barons.  

Here are some key-concepts we are going to be utilizing:

- **Macro Game**: Refers to team-level decisions, such as objective control, map movements, and overall strategy.  
- **Micro Game**: Refers to individual player performance, including mechanics, positioning, and combat decisions.  

A common discussion in competitive and casual play is understanding what truly determines the outcome of a match. Is it early-game advantage? Strong macro decisions? Or individual player skill (Micro Game)?

This project is motivated by that question:

**Which factors most strongly impact the win or loss of a team: early-game advantage, macro objective control, or player performance (micro)?**

Beyond analyzing these factors, we also approach a predictive task: identifying a player's role based on their in-game statistics.

In League of Legends, each team has five roles:

- **Top**: Usually plays more independently and is characterized by higher durability and more isolated performance. Players in this role tend to have moderate gold and damage, with less early interaction compared to other roles.
- **Jungle (Jng)**: Moves across the map and is characterized by high activity and objective involvement. Jungle players often have more assists, less consistent farming, and strong participation in map control.
- **Mid**: Central role with high impact on the game, typically showing high damage output and strong resource generation. Mid players are frequently involved in fights and contribute to overall map pressure.
- **Bot (ADC)**: Main damage dealer, especially in later stages of the game. This role is characterized by high gold income, high CS (farming), and strong sustained damage in team fights.
- **Support (Sup)**: Focused on enabling the team rather than dealing damage. Supports usually have low gold and damage, but high vision-related metrics and assist counts, playing a key role in team coordination.

Each role has distinct patterns in terms of gold, damage, vision, and map interaction. Because of that, we expect it to be possible to predict a player’s role using statistical in-game features alone.

---

### The Dataset

The dataset used in this project contains professional League of Legends match data collected from Oracle's Elixir. Each row represents a player’s performance in a specific match.

The dataset includes information about:
- Match metadata (league, patch, date)
- Player role and team
- Champion selection and bans
- Combat statistics (kills, deaths, assists)
- Objective control (dragons, barons, towers)
- Gold, experience, and farming metrics
- Early-game performance at different timestamps (10, 15, 20, 25 minutes)

Below are some of the most relevant columns used in this analysis:

| Column | Description | Category |
|------|------------|----------|
| `position` | Player role (top, jng, mid, bot, sup) | Target / Role |
| `result` | Win (1) or Loss (0) | Outcome |
| `kills`, `deaths`, `assists` | Basic combat stats | Micro |
| `doublekills`, `triplekills`, `quadrakills`, `pentakills` | Multi-kill performance indicators | Micro |
| `damagetochampions`, `dpm` | Damage output during the game | Micro |
| `visionscore`, `wardsplaced` | Vision control and map awareness | Macro |
| `dragons`, `barons`, `towers` | Objectives secured by the team | Macro |
| `firstdragon`, `firstherald`, `firstbaron`, `firsttower` | First objective taken in the game | Early Game / Macro |
| `firstbloodkill`, `firstbloodassist`, `firstbloodvictim` | First kill involvement in the match | Early Game / Micro |
| `goldat10`, `golddiffat10` | Early gold and advantage at 10 minutes | Early Game |
| `xpat10`, `csat10` | Early experience and farming at 10 minutes| Early Game |
| `totalgold`, `earnedgold` | Overall economic performance | Macro / Micro |

These features allow us to break the game into three main components: early game, macro decisions, and individual player performance.

- **Early Game Indicators**  
  Early-game performance is captured through variables related to first objectives and early advantages. Metrics such as `firstdragon`, `firstherald`, `firsttower`, and early gold, experience, and CS differences (e.g., `golddiffat15`, `xpdiffat15`, `csdiffat15`) help measure how strong a team is in the first stages of the match.

- **Macro Indicators**  
  Macro play is reflected in how teams control the map and secure objectives. Variables such as `dragons`, `heralds`, `barons`, `towers`, and `inhibitors` represent objective control, while features like `side` and `firstPick` capture structural game advantages. Vision-related metrics like `wardsplaced` and `visionscore` also play an important role in understanding map control.

- **Micro Indicators**  
  Individual player performance is captured through combat and efficiency metrics. This includes basic stats like `kills`, `deaths`, and `assists`, as well as more advanced indicators such as damage output (`damagetochampions`, `dpm`), gold and resource efficiency(`totalgold`,`earnedgold`), and teamfight impact through multi-kills (`doublekill`, `triplekill`, `quadrakill`, and `pentakills`). Vision and map interaction at the player level also contribute to understanding individual impact.

By organizing the data in this way, we are able to connect in-game behavior with measurable statistics, making it possible to both analyze what drives match outcomes and build predictive models based on player performance.

By combining these different types of features, we are able to both:
1. Analyze what contributes most to winning games  
2. Build a predictive model that identifies player roles based on behavior and performance patterns 

### Sample of the Original Dataset

To provide an overview of the structure of the dataset before any preprocessing, we display a random sample of observations below.

| gameid             | datacompleteness   | url                                          | league   |   year | split              |   playoffs | date                |   game |   patch |   participantid | side   | position   | playername   | playerid                                  | teamname        | teamid                                  |   firstPick | champion   | ban1    | ban2    | ban3     | ban4         | ban5   | pick1   | pick2   | pick3    | pick4   | pick5      |   gamelength |   result |   kills |   deaths |   assists |   teamkills |   teamdeaths |   doublekills |   triplekills |   quadrakills |   pentakills |   firstblood |   firstbloodkill |   firstbloodassist |   firstbloodvictim |   team kpm |   ckpm |   firstdragon |   dragons |   opp_dragons |   elementaldrakes |   opp_elementaldrakes |   infernals |   mountains |   clouds |   oceans |   chemtechs |   hextechs |   dragons (type unknown) |   elders |   opp_elders |   firstherald |   heralds |   opp_heralds |   void_grubs |   opp_void_grubs |   firstbaron |   barons |   opp_barons |   atakhans |   opp_atakhans |   firsttower |   towers |   opp_towers |   firstmidtower |   firsttothreetowers |   turretplates |   opp_turretplates |   inhibitors |   opp_inhibitors |   damagetochampions |     dpm |   damageshare |   damagetakenperminute |   damagemitigatedperminute |   damagetotowers |   wardsplaced |   wpm |   wardskilled |   wcpm |   controlwardsbought |   visionscore |   vspm |   totalgold |   earnedgold |   earned gpm |   earnedgoldshare |   goldspent |   gspd |   gpr |   total cs |   minionkills |   monsterkills |   monsterkillsownjungle |   monsterkillsenemyjungle |   cspm |   goldat10 |   xpat10 |   csat10 |   opp_goldat10 |   opp_xpat10 |   opp_csat10 |   golddiffat10 |   xpdiffat10 |   csdiffat10 |   killsat10 |   assistsat10 |   deathsat10 |   opp_killsat10 |   opp_assistsat10 |   opp_deathsat10 |   goldat15 |   xpat15 |   csat15 |   opp_goldat15 |   opp_xpat15 |   opp_csat15 |   golddiffat15 |   xpdiffat15 |   csdiffat15 |   killsat15 |   assistsat15 |   deathsat15 |   opp_killsat15 |   opp_assistsat15 |   opp_deathsat15 |   goldat20 |   xpat20 |   csat20 |   opp_goldat20 |   opp_xpat20 |   opp_csat20 |   golddiffat20 |   xpdiffat20 |   csdiffat20 |   killsat20 |   assistsat20 |   deathsat20 |   opp_killsat20 |   opp_assistsat20 |   opp_deathsat20 |   goldat25 |   xpat25 |   csat25 |   opp_goldat25 |   opp_xpat25 |   opp_csat25 |   golddiffat25 |   xpdiffat25 |   csdiffat25 |   killsat25 |   assistsat25 |   deathsat25 |   opp_killsat25 |   opp_assistsat25 |   opp_deathsat25 |
|:-------------------|:-------------------|:---------------------------------------------|:---------|-------:|:-------------------|-----------:|:--------------------|-------:|--------:|----------------:|:-------|:-----------|:-------------|:------------------------------------------|:----------------|:----------------------------------------|------------:|:-----------|:--------|:--------|:---------|:-------------|:-------|:--------|:--------|:---------|:--------|:-----------|-------------:|---------:|--------:|---------:|----------:|------------:|-------------:|--------------:|--------------:|--------------:|-------------:|-------------:|-----------------:|-------------------:|-------------------:|-----------:|-------:|--------------:|----------:|--------------:|------------------:|----------------------:|------------:|------------:|---------:|---------:|------------:|-----------:|-------------------------:|---------:|-------------:|--------------:|----------:|--------------:|-------------:|-----------------:|-------------:|---------:|-------------:|-----------:|---------------:|-------------:|---------:|-------------:|----------------:|---------------------:|---------------:|-------------------:|-------------:|-----------------:|--------------------:|--------:|--------------:|-----------------------:|---------------------------:|-----------------:|--------------:|------:|--------------:|-------:|---------------------:|--------------:|-------:|------------:|-------------:|-------------:|------------------:|------------:|-------:|------:|-----------:|--------------:|---------------:|------------------------:|--------------------------:|-------:|-----------:|---------:|---------:|---------------:|-------------:|-------------:|---------------:|-------------:|-------------:|------------:|--------------:|-------------:|----------------:|------------------:|-----------------:|-----------:|---------:|---------:|---------------:|-------------:|-------------:|---------------:|-------------:|-------------:|------------:|--------------:|-------------:|----------------:|------------------:|-----------------:|-----------:|---------:|---------:|---------------:|-------------:|-------------:|---------------:|-------------:|-------------:|------------:|--------------:|-------------:|----------------:|------------------:|-----------------:|-----------:|---------:|---------:|---------------:|-------------:|-------------:|---------------:|-------------:|-------------:|------------:|--------------:|-------------:|----------------:|------------------:|-----------------:|
| LOLTMNT01_196798   | complete           | nan                                          | LJL      |   2025 | Forge              |          0 | 2025-01-31 11:07:01 |      1 |   15.02 |               2 | Blue   | jng        | Tatsu        | oe:player:c46dc30e7888a9b9eaee54bb9698d27 | We Can Win      | nan                                     |           1 | Xin Zhao   | K'Sante | Sejuani | Nocturne | Braum        | Sylas  | nan     | nan     | nan      | nan     | nan        |         2182 |        1 |       5 |        3 |        13 |          22 |           15 |             2 |             0 |             0 |            0 |            0 |                0 |                  0 |                  1 |       0.6  |   1.02 |           nan |       nan |           nan |               nan |                   nan |         nan |         nan |      nan |      nan |         nan |        nan |                      nan |      nan |          nan |           nan |       nan |           nan |          nan |              nan |          nan |        1 |            1 |        nan |            nan |          nan |      nan |          nan |             nan |                  nan |            nan |                nan |            1 |                0 |               25892 |  711.97 |          0.2  |                1738.35 |                    1186.91 |             1360 |             7 |  0.19 |             8 |   0.22 |                    5 |            45 |   1.24 |       14494 |         9767 |       268.57 |              0.22 |       11525 | nan    |   nan |        244 |            41 |            203 |                     nan |                       nan |   6.71 |       3259 |     3910 |       77 |           3097 |         3487 |           68 |            162 |          423 |            9 |           0 |             0 |            0 |               0 |                 0 |                0 |       5313 |     6088 |      105 |           5283 |         5613 |          105 |             30 |          475 |            0 |           2 |             0 |            1 |               1 |                 3 |                1 |       7115 |     8777 |      141 |           7133 |         7756 |          135 |            -18 |         1021 |            6 |           2 |             1 |            2 |               1 |                 5 |                2 |       9132 |    11515 |      169 |           8604 |         9953 |          168 |            528 |         1562 |            1 |           4 |             3 |            2 |               1 |                 6 |                3 |
| LOLTMNT02_239612   | complete           | nan                                          | LRS      |   2025 | Split 1            |          0 | 2025-03-30 19:28:09 |      1 |   15.06 |               4 | Blue   | bot        | Horus        | oe:player:0d972c4c17b8777458c98282a40c1ad | Malvinas Gaming | oe:team:d5772fdb1ae7fcc8046fa2e056a8715 |           1 | Kai'Sa     | Ezreal  | Rumble  | Jayce    | Renata Glasc | Poppy  | nan     | nan     | nan      | nan     | nan        |         2105 |        1 |       7 |        2 |        11 |          24 |           13 |             1 |             1 |             0 |            0 |            1 |                0 |                  1 |                  0 |       0.68 |   1.05 |           nan |       nan |           nan |               nan |                   nan |         nan |         nan |      nan |      nan |         nan |        nan |                      nan |      nan |          nan |           nan |       nan |           nan |          nan |              nan |          nan |        0 |            0 |        nan |            nan |          nan |      nan |          nan |             nan |                  nan |            nan |                nan |            0 |                0 |               28770 |  820.05 |          0.26 |                 425.44 |                     322.4  |             4444 |            14 |  0.4  |             6 |   0.17 |                    1 |            31 |   0.88 |       14650 |        10080 |       287.32 |              0.23 |       12575 | nan    |   nan |        277 |           271 |              6 |                     nan |                       nan |   7.9  |       3003 |     3532 |       63 |           3058 |         3699 |           73 |            -55 |         -167 |          -10 |           0 |             3 |            0 |               0 |                 1 |                0 |       4857 |     6146 |      109 |           4886 |         6051 |          127 |            -29 |           95 |          -18 |           1 |             3 |            0 |               0 |                 1 |                0 |       6849 |     8983 |      155 |           6658 |         8578 |          170 |            191 |          405 |          -15 |           1 |             7 |            0 |               0 |                 1 |                1 |       8917 |    12544 |      204 |           8672 |        11067 |          220 |            245 |         1477 |          -16 |           1 |            10 |            1 |               0 |                 5 |                2 |
| LOLTMNT05_109372   | complete           | nan                                          | NLC      |   2025 | Winter             |          1 | 2025-02-12 19:32:02 |      3 |   15.03 |              10 | Red    | sup        | Eski         | oe:player:e38b7e92f32edc7bd206592ee7963ca | Rich Gang       | oe:team:2b20d0fcdb72f7b490eb1dc468136cf |           0 | Lulu       | Skarner | K'Sante | Poppy    | Braum        | Pyke   | nan     | nan     | nan      | nan     | nan        |         2162 |        1 |       1 |        1 |        12 |          22 |           13 |             0 |             0 |             0 |            0 |            1 |                0 |                  1 |                  0 |       0.61 |   0.97 |           nan |       nan |           nan |               nan |                   nan |         nan |         nan |      nan |      nan |         nan |        nan |                      nan |      nan |          nan |           nan |       nan |           nan |          nan |              nan |          nan |        0 |            0 |        nan |            nan |          nan |      nan |          nan |             nan |                  nan |            nan |                nan |            0 |                0 |                6423 |  178.25 |          0.06 |                 264.2  |                     147.28 |              635 |            47 |  1.3  |             8 |   0.22 |                    9 |            89 |   2.47 |        9195 |         4509 |       125.13 |              0.09 |        7575 | nan    |   nan |         40 |            40 |              0 |                     nan |                       nan |   1.11 |       2256 |     3147 |       19 |           2291 |         2896 |           11 |            -35 |          251 |            8 |           0 |             1 |            0 |               0 |                 2 |                0 |       3341 |     4528 |       29 |           3453 |         4806 |           19 |           -112 |         -278 |           10 |           0 |             1 |            0 |               0 |                 3 |                0 |       4428 |     6123 |       31 |           4365 |         6621 |           22 |             63 |         -498 |            9 |           0 |             3 |            1 |               0 |                 3 |                1 |       5772 |     7956 |       33 |           5282 |         8134 |           23 |            490 |         -178 |           10 |           1 |             6 |            1 |               0 |                 4 |                2 |
| 11921-11921_game_3 | partial            | https://lpl.qq.com/es/stats.shtml?bmid=11921 | LPL      |   2025 | Split 2 Placements |          0 | 2025-03-31 12:56:51 |      3 |   15.06 |               7 | Red    | jng        | Monki        | oe:player:a23b45071ea855f85e43db4b5a1683d | Team WE         | oe:team:62c1cd9465dc63824593ee5046f5aa8 |           0 | Olaf       | Naafiri | Yone    | Akali    | Gnar         | Jax    | nan     | nan     | nan      | nan     | nan        |         1929 |        1 |       1 |        3 |        12 |          18 |           13 |           nan |           nan |           nan |          nan |          nan |                0 |                nan |                nan |       0.56 |   0.96 |           nan |       nan |           nan |               nan |                   nan |         nan |         nan |      nan |      nan |         nan |        nan |                      nan |      nan |          nan |           nan |       nan |           nan |          nan |              nan |          nan |      nan |          nan |        nan |            nan |          nan |      nan |          nan |             nan |                  nan |            nan |                nan |          nan |              nan |               17391 |  540.93 |          0.21 |                1215.83 |                     nan    |              nan |             8 |  0.25 |             5 |   0.16 |                    6 |            38 |   1.18 |       12981 |         8770 |       272.78 |              0.2  |       11475 | nan    |   nan |        271 |            50 |            221 |                     151 |                        20 |   8.43 |        nan |      nan |      nan |            nan |          nan |          nan |            nan |          nan |          nan |         nan |           nan |          nan |             nan |               nan |              nan |        nan |      nan |      nan |            nan |          nan |          nan |            nan |          nan |          nan |         nan |           nan |          nan |             nan |               nan |              nan |        nan |      nan |      nan |            nan |          nan |          nan |            nan |          nan |          nan |         nan |           nan |          nan |             nan |               nan |              nan |        nan |      nan |      nan |            nan |          nan |          nan |            nan |          nan |          nan |         nan |           nan |          nan |             nan |               nan |              nan |
| 12439-12439_game_4 | partial            | https://lpl.qq.com/es/stats.shtml?bmid=12439 | LPL      |   2025 | Split 2            |          1 | 2025-06-13 12:08:10 |      4 |   15.11 |             200 | Red    | team       | nan          | nan                                       | Bilibili Gaming | oe:team:d356a144644879dabb5f34cd99c886d |           0 | nan        | Taliyah | Trundle | Orianna  | Miss Fortune | Akali  | Jayce   | Varus   | Xin Zhao | Karma   | Cassiopeia |         1708 |        1 |      29 |       10 |        75 |          29 |           10 |           nan |           nan |           nan |          nan |            1 |              nan |                nan |                nan |       1.02 |   1.37 |             1 |         4 |             0 |               nan |                   nan |         nan |         nan |      nan |      nan |         nan |        nan |                        4 |      nan |          nan |           nan |         1 |             0 |            0 |                3 |          nan |        1 |            0 |        nan |            nan |            1 |        9 |            2 |             nan |                  nan |            nan |                nan |            2 |                0 |              101177 | 3554.23 |        nan    |                4258.52 |                     nan    |              nan |           118 |  4.15 |            40 |   1.41 |                   47 |           259 |   9.1  |       60294 |        41494 |      1457.63 |            nan    |       51493 |   0.11 |   nan |        nan |           nan |            211 |                     135 |                        20 | nan    |        nan |      nan |      nan |            nan |          nan |          nan |            nan |          nan |          nan |         nan |           nan |          nan |             nan |               nan |              nan |        nan |      nan |      nan |            nan |          nan |          nan |            nan |          nan |          nan |         nan |           nan |          nan |             nan |               nan |              nan |        nan |      nan |      nan |            nan |          nan |          nan |            nan |          nan |          nan |         nan |           nan |          nan |             nan |               nan |              nan |        nan |      nan |      nan |            nan |          nan |          nan |            nan |          nan |          nan |         nan |           nan |          nan |             nan |               nan |              nan |

---


## Data Cleaning and Exploratory Data Analysis

texto aqui

## Assessment of Missingness

texto aqui

## Hypothesis Testing

texto aqui

## Framing a Prediction Problem

texto aqui

## Baseline Model

texto aqui

## Final Model

texto aqui

## Fairness Analysis

texto aqui