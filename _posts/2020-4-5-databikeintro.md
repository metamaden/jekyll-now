---
layout: post
title: Databike: coding an app about scooter upkeep, in R
categories:[simulation; app; game; reproducible_code]
tags:[R; Rstats; grid; jpeg; svDialogs; ascii]
---

# Overview

Here, I describe the process of designing `databike`, an interactive application about scooter upkeep. The app code is available at the [databike repository](https://raw.githubusercontent.com/metamaden/databike/). This was a novel undertaking, and there's plenty of room for improving this application. I will periodically share lessons of general interest for learning programming.

(gif of logo, with color inversion)

<img src="https://raw.githubusercontent.com/metamaden/databike/vignettes/imgs/logo.jpg" align = "right" alt="drawing" width="500"/>
**Figure 1.** <`databike` logo>

# App design

## Goals

There were several goals for the initial app. The goal was to create a command line-runnable application the includes user interactivity and some form of hook to allow continuous progression. These goals would ideally be met while adhering to principles of reproducible code and app development.

## Early app progress

The initial driving notion was to design a compact application with few dependencies. I experimented with different methods to print text rapidly for a kind of animation interface. I arrived at looping on printed character strings in loops. In other words, ascii art!

(gif of ride normal animation here.)

Next, I explored options for user dialogue windows, settling on `svDialogue::dvg_dialogue()` owing to its flexibility and out-of-the-box utility.

## Tasks outline and workflows

I spent considerable time trying to simply get the ascii art and dialogues to play well togther. These still need considerable fine-tuning, but I could have saved considerable time had I thought to literally sketch out the app design itself. Ultimately, I arrived at the following design, which I can revisit if I need to return to "bigger picture" questions for next steps.

<img src="https://raw.githubusercontent.com/metamaden/databike/vignettes/imgs/app_flowchart.jpg" align = "center" alt="drawing" width="1800"/>
**Figure 2.** <App flowchart>

Even if an early workflow diagram doesn't accurately reflect the ultimate product, reviewing such a diagram provides a vital opportunity to pause and reflect on the bigger picture and individual steps to take next.

## Repository and package outline

(show tree diagram of app dir, main branch at posting)

## Takeaways
Takeways from this section include: 
* bulleted list
  + keep a log for goals, ideas, bugs, etc. and meticulously organize this. It can be helpful to simply have this in a text file or physical journal.
  + draw workflow diagrams and review these frequently.
  + schedule breaks and take them. Return to "bigger picture" questions frequently.

# R code examples

This section walks through some of the interface decisions for the early app. Examples are runnable from an active R session.

## Ascii art and "animations"

```
# generic code chunk
```

<img src="https://raw.githubusercontent.com/metamaden/databike/appnotes/imgs/drive.gif" align = "center" alt="drawing" width="200"/>
**Figure 3.** <drive animation>

## User dialogue trees

```
# generic code chunk
```

# Conclusions

