---
title: 'Databike: coding an app about scooter upkeep, in R'
author: "Sean Maden"
date: "4/9/2020"
layout: post
tags:
- R
- Rstats
- grid
- jpeg
- svDialogs
- ascii
categories:
- simulation
- app
- game
- reproducible_code
---

# Introduction and motivation

I've owned my 1987 CH150 Honda elite (a.k.a. "the databike") for almost 2 years, but I'm still a novice when it comes to scooter repair and upkeep. My hobby inspired me to write an app about scooter upkeep. In this post, I describe early development on [`databike`](https://github.com/metamaden/databike/), an app about scooter upkeep. In `databike`, the user makes decisions about maintenance (e.g. whether to repair or maintain) and during obstacles encountered on rides (e.g. whether to quit or continue). These decisions impact the bike condition, and total "mileage" (distance traveled) is tracked. The basic "user interface" consists of graphics (text/ascii art, printed in sequence), and user dialogues and notifications (pop-up windows with text and options buttons). The app code is available on GitHub ([link](https://github.com/metamaden/databike/)). This undertaking was novel for me, and I have loads of updates in mind.

# The application

## Programming in R

Creating this app in R was a conscious design decision to test my programming knowledge. Rest assured, there are better languages and utilities more commonly used and supported for app development (e.g. developer kits, C++, Java, etc.). The challenge of writing app code in R proved to be worthwhile, and lead me to real insights and about the language's potential. More generally, challenges of this kind already drive continued development and improvement of code to tackle real-world issues. Modern innovations in programming usability already seek to address issues of this kind. I eventually would like to port `databike` to Python (see Goals, below).

## R scripts

Currently, the main app leans on 3 key scripts: `app.R`, `functions.R`, and `params.R`. These work together to define and update session variables, and manage user interfacing. The app runs from the root directory using:

```
R
Rscript ./app.R
```

## User interfacing and dependencies

To include a sort of proto user interface and animations, I addressed 2 questions: 1. I create the illusion of motion with looped graph prints? 2. Could I manage dialogue trees and sequence them effectively? Exploring these questions was a challenge unto itself, and my current solution requires 2 dependencies, [`svDialogs`](https://cran.r-project.org/web/packages/svDialogs/index.html) and [`grid`](https://bookdown.org/rdpeng/RProgDA/the-grid-package.html). 

## Ascii art and "animations"

<img src="https://raw.githubusercontent.com/metamaden/databike/master/appnotes/imgs/drive.gif" align = "right" alt="drawing" width="200"/>
**Figure 2.** Normal ride animation

The game "graphics" are simply characters printed to a new R viewport (using `grid.newpage(); grid.text()`) with some delay (using `Sys.sleep(sleepint)`). Note that "mileage", or total distance (`tdist` variable), is updated as frames progress during a ride.

The following example script illustrates the concept. The for loop takes c `c` (unit distance) for every animation loop of frames vector `fv`.

```
require(grid) # dependency
sleepint = 0.5 # pause between frame prints
# frame vector, or "slides" of animation
fv <- c(bike.frame1, bike.frame2, bike.frame3)
ride.seq <- seq(1, 20, 1)
tdist <- 0
for(c in ride.seq){
  for(f in fv){
    grid.newpage()
    grid.text(f) # prints frame text
    Sys.sleep(sleepint)
  }
  tdist <- tdist + 1
}
```

# Obstacles during rides

Obstacles are randomly encountered on rides, triggering a user prompt with options to 
"continue" or "quit ride". This is managed by `o.seq`, which is a vector of `ride.seq` indices corresponding to obstacles. In terms of the example script, this adds the following if/else condition as follows
 
```
require(grid) # dependency
sleepint = 0.5 # pause between frame prints
# frame vector, or "slides" of animation
fv <- c(bike.frame1, bike.frame2, bike.frame3)
ride.seq <- seq(1, 20, 1)
tdist <- 0
# obstacles data
num.obst <- sample(10, 1) # obstacle quantity
o.seq <- sample(ride.seq, num.obst) # obstacle indices
for(c in ride.dist){
  if(c in o.seq){
    # handles obstacle encounter
  } else{
    for(f in fv){
    grid.newpage()
    grid.text(f) # prints frame text
    Sys.sleep(sleepint)
  }
  tdist <- tdist + 1
  }
}
```

Where `#handles obstacle encounter` manages options for the obstacle encounter, with a chance to update bike condition (`bcond` variable) or stop the ride sequence altogether.

## Goals for app functionality

1. Scheduled error report logs and automated "game testing" (e.g. run app iterations and log outcomes or new errors)
2. Run convenience (e.g. runs from command line, executable file, etc.)
3. Window management (e.g. manage multiple windows)
4. Build (e.g. pass `R CMD CHECK` and `R CMD BUILD`)
5. Python port

# R code examples

This section walks through some of the interface decisions for the early app. Examples are runnable from an active R session.



## User dialogue trees

```
# generic code chunk
```

# Conclusions
