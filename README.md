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

This project explores professional League of Legends match data to better understand what actually drives a team's success.  
We analyze how early-game advantage, macro decisions, and player performance impact win/loss outcomes.  
Additionally, we build a model to predict a player's role based on in-game statistics.

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