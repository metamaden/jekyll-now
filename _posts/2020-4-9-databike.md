---
title: 'Databike: coding an app about scooter upkeep'
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

<img src="https://raw.githubusercontent.com/metamaden/databike/master/appnotes/imgs/databike_ch150elite.jpg" align = "right" alt="elite" width="150"/>

I've owned my 1987 CH150 Honda elite (a.k.a. "the databike", shown at right) for almost 2 years, but I'm still a novice when it comes to scooter repair and upkeep. My hobby inspired me to write an app about scooter upkeep. In this post, I describe early development on [`databike`](https://github.com/metamaden/databike/), an app about scooter upkeep. In `databike`, the user makes decisions about maintenance (e.g. repair, maintain, or do nothing) and during obstacles encountered on rides (e.g. whether to quit or continue). These decisions impact the bike condition and total mileage or ride distance. The basic "user interface" consists of graphics (text/ascii art, printed in sequence), and user dialogues and notifications (pop-up windows with text and options buttons). The app code is available on GitHub ([link](https://github.com/metamaden/databike/)). This undertaking was novel for me, and I have loads of updates in mind.

# The application

With R installed, the app may be run from command line by navigating to the root directory (e.g. `cd /Documents/GitHub/databike`) and using:

```
R
source(app.R)
```

## Programming in R

<img src="https://raw.githubusercontent.com/metamaden/databike/master/appnotes/imgs/logo.jpg" align = "right" alt="logo" width="200"/>

Creating this app in R was a conscious design decision to test my programming knowledge. Rest assured, there are better languages and utilities more commonly used and supported for app development (e.g. developer kits, C++, Java, etc.). While the challenge of writing app code in R proved to be worthwhile, I also want to port the app to Python (see goals below). More generally, challenges of this kind already drive development of utilities that extend user capabilities to tackle real-world issues in data science and beyond.

## R scripts

As of posting, the main app is contained in 3 key scripts: `app.R`, `functions.R`, and `params.R`. These work together to define and update session variables while managing the user interface (see flowchart below).

<img src="https://raw.githubusercontent.com/metamaden/databike/master/appnotes/imgs/app_flowchart.jpg" align = "center" alt="logo" width="500"/>

# User interfacing and dependencies

I asked 2 key questions about app usability: Do looped graph prints create a pleasing animation or graphic? Are dialogue trees feasible? Exploring these questions was a challenge unto itself, and `databike` currently requires 2 dependencies, `svDialogs` and `grid`. 

## Animating printed characters

The game uses lines of characters (string vectors) printed at intervals to make the bike animations. Animation loops translate into distance or mileage, as shown in the example code below:

```
require(grid) # dependency
sleepint = 0.5 # pause between frame prints
# frame vector, or "slides" of animation
fv <- c(frame1, frame2, frame3)
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

The for loop iterates on c (unit distance) once per animation loop, where total distance is the maximum entry in ride.seq. The animation loop prints frames in the frame vector fv using `grid.newpage()` and `grid.text()`, and framerate is controlled by sleepint in `Sys.sleep(sleepint)`. 

## Obstacles during rides

Obstacles are randomly encountered on rides, triggering a continue/quit prompt. This is managed by `o.seq`, a vector of numbers matching entries in `ride.seq`. The num.obst total obstacles are randomized using `sample()`. In terms of the example code, this looks like:
 
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
for(c in ride.seq){
  if(c in o.seq){
    ride.obstacle()
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

Where `ride.obstacle()` handles continue/quit options for the encounter with `dlg_input()`.

# Goals/updates planned

Improvements that would greatly increase this app's usability include:

1. Scheduled error report logs and automated game tests (run app iterations and log outcomes or new errors)
2. Window management (e.g. manage multiple windows)
3. Python port
4. Save/load ride (use caching, use menu for load option)
5. Work on the real databike ;)



<img src="https://raw.githubusercontent.com/metamaden/databike/master/appnotes/imgs/drive.gif" align = "center" alt="drive" width="150"/>
