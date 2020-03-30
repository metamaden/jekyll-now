---
layout: post
title: `scootsim`
tags: ascii; animation; R; Rstats; app
---

(hexsticker goes here)

# Game sequence

The score is an imaginary points system, which is defined here as:

Dt - (Nc + 0.5 * Ns)

Where 

1. Dt is total "distance" traveled (just an integer counter in this case), 
2. Nc is the number of "crashes" (rides where bike stops), and 
3. Ns is the number of "stops" (rides where player stops riding before end of ride).

# App data

I've designed this "app" (e.g. simplistic simulation) with modularity in mind. In other words, I've taken measures to ensure weights on score conditions are tolerant to large magnitude modifications.

# Game guts

Using R presents some challenges, and it took some planning to organize simulation elements.

I settled on a simplistic "graphical interface" that appropriates viewports to generate a passable animation, and a basic user input system. The hard work behind the scenes is being done by `grid::grid.text()` and `svDialogs::dlgInput()`.


If you need help with scooters, consult an expert!


Thie this simulation of scooters could potentially be used as a reminder for scheduled maintenance

The notion is maintenance of aging systems, in this case a rad scooter.

Ongoing monitoring can be crucial to prevent accidents and prolong the lifetime of your bike



A. "inspect bike"
A1. "maintain"
A2. ""
B. "drive"
B1. "obstacle encounter"

