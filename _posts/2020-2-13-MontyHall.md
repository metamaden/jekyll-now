---
layout: post
title: Cracking the Monty Hall problem with brute force simulation
tags: monty_hall; simulation; R; Rstats; ggplot2
---

On a game show stage before you wait 3 closed doors, behind which have been deposited 2 goats and 1 prize, respectively. You are called on to pick a door to be opened to reveal either a goat or a prize. The host, Monty Hall, then reveals a goat behind one of the two remaining unpicked doors. You are then given the option to switch your door selection to the final unpicked door before the big reveal. What should you do?

This is the [Monty Hall problem](https://en.wikipedia.org/wiki/Monty_Hall_problem), a kind of logic puzzle involving conditional probability. Given that you value prizes over goats and lack prior knowledge about which door the prize is behind, it can be readily shown that switching doors *always* increases your win probability. If you stick with your first choice, your success frequency never exceeds 1 of 3 games, while switching increases this to 2 of 3, a pretty substantial improvement!

It's telling that the Monty Hall problem, based on an [actual game show](https://en.wikipedia.org/wiki/Let%27s_Make_a_Deal) from the 1960s, still serves as a good brain teaser to this day. Given its simple rules and decision parameters, it's a problem that lends itself to programmatic simulation. In this post, I'll show how I wrote a simulation function that captures the basic (or "classical") rules of the Monty Hall problem while allowing for exploration of how modifications to the underlying rules and conditions can chanage game outcomes. Hopefully I can inspire you to think generally about opportunities to conceptualize problems in simulations with useful code.

<img src="https://raw.githubusercontent.com/metamaden/montyhall/master/plots/montyhall.png" url="https://github.com/metamaden/montyhall" alt="drawing" width="200" align = "right"/>

I've deployed the simulation code with a strictly reproducible vignette in the [`montyhall`](https://github.com/metamaden/montyhall) R package. Deploying work as an R package can be extremely worthwhile in production level data science projects. In writing the code for this package, I've knowingly omitting a few best practices for package authorship, in service to expediency and what I consider more clearly written code. Note there are [many](https://cran.r-project.org/submit.html) [great](https://www.bioconductor.org/developers/package-submission/) [places](https://www.bioconductor.org/developers/package-guidelines/) you can and should refer to for learning R package standards and why they matter. Three key functions, `mhgame()`, `mhsim()`, and `getfw()`, manage game simulations and return win frequencies across sets of simulated games. These functions only make use of base R without added dependencies. I've also added several visualization utilities that make use of some stellar R packages for visualizations, including [`ggplot2`](https://cran.r-project.org/web/packages/ggplot2/index.html) and ['gridExtra`](https://cran.r-project.org/web/packages/gridExtra/index.html). Below, I'll walk through the simulation code and show its use in scripts to investigate the Monty Hall Porblem in greater depth.

# Formulating the game

Here's a formulation of the steps in the original Monty Hall problem, as described above:

1. Three doors total, behind which 1 has a prize, and the remaining 2 have goats.
2. The player picks a door (player decision 1).
3. Monty reveals one of the two remaining doors to be a goat.
4. The player decides whether to stick with their initial choice or switch to the last unpicked door (player decision 2).
5. Game outcome is determined by whether the final player-selected door reveals either a prize (win) or a goat (loss).

We should note some key characteristics of this formulation. First, an initial naive formulation might simply have that the player simply picks a door twice, with Monty doing something or other in between. Instead, I've stated the player picks a door (step 2/decision 1) then decides whether to switch doors (step 4/decision 2). This distinction is vital because in the second decision we have new information from Monty's reveal in step 3 that can help our win chances if we know to heed it. Also note there's implied randomness in the first 3 steps. That is, the prize door is set at random (step 1), the player picks their initial door at random (step 2/player decision 1), and Monty will occasionally reveal a goat at random (step 3). To clarify, randommization occurs in step 3 just a third of the time, specifically whenever the player first picked the prize door and thus leaves Monty to decide between one of two goats to reveal.

Before we dive into the simulation code, I'll next show a [pseudocode](https://en.wikipedia.org/wiki/Pseudocode) outline of tasks the code needs to accomplish to simulate the game with classic parameters. Pseudocode is simply a way of abstracting coding tasks that has the convenience of being programming language-agnostic. For the Monty Hall simulation, the initial pseudocode might be something like:

* run Monty_Hall_Game:
  + get door_indices from 1:ndoors
  + assign prize_door
  + randomize player_door_index1
  + get remaining_doors
  + get monty_door_indices up to ndoors - 2
  + get player_door_index2 as remaining_door_index
  + if player_door_index2 == prize_door, return "win", else "lose"

* run Monty_Hall_Simulation:
  + do Monty_Hall_Game up to num_iterations
  
I've outlined pseudocode for two functions which loosely correspond to the `mhgame()` and `mhsim()` functions in the `montyhall` package. In programming, it's often preferred to break a large problem into discrete smaller sub-problems, wherein each sub-problem solution can be more easily fine-tuned. This reductionist approach can make debugging and [unit testing](https://en.wikipedia.org/wiki/Unit_testing) *a lot* easier, especially for large and multifaceted projects. With these conceptual formulations of the problem and code tasks, let's look how I tackled these in the simnulation code below.

# The simulation code

I've written 3 functions that allow us to rapidly complete Monty Hall problem simulations while allowing for modification of various game conditions. First, the `mhgame()` function runs a single game or "game iteration." Second, `mhsim()` executes a series of game iterations that defines a simulation run, up to N = `niter` total game iterations, where every such series uses a single seed for reproducible randomization (more on that below). Finally, `getfw()` takes the output from `mhsim()`, a list of game outcome vectors (either "win" or "loss"), and returns a single vector of win fractions.

Importantly, `mhsim()` [vectorizes](https://en.wikipedia.org/wiki/Automatic_vectorization) game simulations with `lapply()`. Vectorization is a great way to speed up repetitive tasks in coding, and can be crucial for especially memory-taxing or large problems. The `lapply()` function is a member of the `apply` [family](https://www.rdocumentation.org/packages/base/versions/3.6.2/topics/lapply) of R functions, which have been specialized for different varieties of vectorization tasks. Some other useful ways of speeding up your code can include parallelization of tasks with [multithreading](https://en.wikipedia.org/wiki/Thread_(computing)#Multithreading). However, note that some parallelization solutions aren't strictly replicable and often require one or several additional dependencies. It's important to tailor the complexity a solution to that of its problem, and thus avoid premature or excessive optimization. For our purposes, running tens of thousands of Monty Hall simulations isn't memory intensive, and our operations below will all complete in about a minute or less.

The arguments `niter` and `seed` in `mhsim()` should be considered for each run. The `niter` argument specifies the number of game iterations to simulate, while setting `seed` specifies the value passed to `set.seed()`. Setting the seed allows for *exact replication* of run results with the same seed, an important component for operations implementing randomness. As already mentioned, the simulation assumes random player selection in stage 2 and occasional random door selection by Monty in stage 3. I've also allowed for the player decision to switch to be modified from "always" (100% of games) to some frequency between 0 and 1. I implemented randomization with the `sample()` R function.

To show how `mhgame()` delivers on the pseudocode tasks above, I'll describe how it breaks the game into discrete component steps. First, the index of the prize door is specified.

```
which.prize <- sample(doorseq, nprize)
```

Then the player's first decision is simulated.

```
dec1select <- sample(doorseq, ndec1)
```

Next, Monty reveals a goat. This entails either selecting 1 of 2 doors at random (e.g. if the player already picked the prize door, an unlikely event), or simply picking the last remaining door (e.g. if the player already picked a goat door, a likely event).

```
mdooroptions <- doorremain1[!doorremain1 %in% which.prize]
if(length(mdooroptions) < 2){
  mselect <- mdooroptions
} else{
  mselect <- sample(mdooroptions, nr)
}
```

We check that the number of remaining "non-revealed" doors specified by `nrevealdif` is valid for the problem, then proceed to either specify a random set ofdoor indices to reveal or the only remaining valid door.

Next, the player either switches or stays on their initial door selection. Note how switch frequency from `doorswitch` impacts the likelihood of passing `switch` or `stay` to `ssvar`.

```
if(ssvar == "switch"){
  if(length(doorremain2) > 1){
    dec2select <- sample(doorremain2, ndec2)
  } else{
    dec2select <- doorremain2
  }
}
```

The function then returns a vector of game outcomes (either `win` or `loss`) of length equal to `niter`. I've also included a `verbose.results` option that stores the granual game details for each iteration alongside outcome. I mainly included this for bug squashing.

# Simulating the canonical/classic problem

Let's study the impact of varying the number of simulations and iterations per simulation on the distribution of win frequencies across simulations. I started small with just 5 simulations of 2 games (10 total games), and increased to 100 (10,000 games) and 1,000 (1,000,000 games) simulations and iterations, respectively. To execute the simulations, I iterated over 3 parameter sets and time it with `Sys.time()`. I used a `for` loop to iterate over the indices of the 3-value parameter vectors where indices are used to retrieve parameters for each run.

```
# parameter sets
simv <- c(5, 100, 1000)
iterv <- c(2, 100, 1000)
lr <- list()
t1 <- Sys.time()
for(s in 1:length(simv)){
  runname <- paste0(simv[s], ";", iterv[s])
  lr[[runname]] <- getfw(nsimulations = simv[s], niterations = iterv[s])
}
tdif <- Sys.time() - t1
```

The 3 runs completed in about 27 seconds. With so few iterations and simulations in the first run, there's huge variance in the win fraction (standard deviation of 0.45). Increasing iterations and simulations each to 100 already shows the distribution converging on the expected win frequency of 0.66. Further increase to 1,000 simulations and iterations results in a more clearly normal distribution with much tighter standard deviation of 0.01.

Let's now show the composite plot of win frequency distributions across the 3 runs. Note I've stored the run info (number of simulations and iterations per run) in the list names, and we can unpack these with regular exressions using `gsub()` for the respective plot titles. We'll use `par` to manage the plot output and formatting, where `nrow = c(1, 3)` specifies the plot output conforms to a matrix of 1 row and 3 columns, and `oma = c(3, 3, 3, 1)` adds outer margin whitespace for axis labels. We'll remove redundant axis labels for each plot and add these back with `mtext()`.

```
pdf("mh_3runs.pdf", 10, 4)
# format image output
par(mfrow = c(1, 3), oma = c(3, 3, 3, 1))
for(r in 1:length(lr)){
  rdat <- lr[[r]]
  # get plot title info
  rname <- names(lr)[r]
  simr <- gsub(";.*", "", rname)
  iterr <- gsub(".*;", "", rname)
  pmain <- paste0(simr," simulations\n", iterr, " iterations")
  # add run histogram to image output
  hist(rdat, main = pmain, xlab = "", ylab = "")
}
# add outer axis labels
mtext("Win Frequency", side = 1, outer = T)
mtext("Number of Simulations", side = 2, outer = T)
dev.off()
```

<img src="https://raw.githubusercontent.com/metamaden/montyhall/master/plots/mh_3runs.png" align = "center" alt="drawing" width="1500"/>

If you prefer to be more precise about the increase in normalcy, we can show greater distribution normalcy by high confidence from the 
[Shapiro-Wilk Normality test](https://en.wikipedia.org/wiki/Shapiro%E2%80%93Wilk_test)
with `shapiro.test`, where we test the null hypothesis that data were drawn from a normal distribution.

```
# run normalcy tests
st1 <- shapiro.test(lr[["5;2"]])$p.value
st2 <- shapiro.test(lr[["100;100"]])$p.value
st3 <- shapiro.test(lr[["1000;1000"]])$p.value
```

With increased simulations and iterations, our p-value increase from 0.05 in the first and smallest run to 0.58 in the third and largest run. Practically, this means confidence to reject the alternative hypothesis of non-normality decreases as the underlying distributions converge to approximate normality.

# Bending the rules

I've written `mhsim()` and its `getfw()` wrapper to allow us to modify the game rules in a few ways. Exploring how bending the rules impacts simulated win frequencies can lead us to better understand the roles of key game conditions. Showing quantitatively how win frequencies change in light of different conditions can help us to better characterize the problem and inform the notion that it's *always* better to switch doors under the classical game rules.

The first condition we'll explore is the number of doors. I've allowed for the door quantity to be changed with the `ndoors` argument. Practically, this just changes the game setup for an interation by defining a vector of sequential door indices of length `ndoors`. Thus increasing `ndoors` from 3 still preserves other parameters for the classical game be default. 

I've also allowed for changing the frequency with which the player switches doors with `doorswitch`. The default value of 1 means the player switches 100% of the time, and setting this a lower value between 0 and 1 means decreasing the switch frequency. I did this by implementing `sample()` to randomly select from what's essentiallt a weighted binomial distribution (e.g. possible outcomes are binary but each outcome has a distinct weight). If `doorswitch = 0.2`, we parse player decision by sampling from a distribution where 20% of options are "switch" and (100 - 20 = ) 80% of options are "stay".

# Many doors as an extrapolation mnemonic

One of the more useful [mnemonic devices](https://en.wikipedia.org/wiki/Mnemonic) I've encountered for intuiting the answer to the Monty Hall problem is to increase the number of doors. Maybe we're unsure if switching doors will increase our odds when there are just 3 doors. But if there are 100 doors, and Monty reveals goats behind 98 of them, it's much clearer that switching will increase our chances of winning. We can quantitatively visualize this intuitively useful device with the simulation code. 

Let's now generate and time the results from running 100 simulations of 100 iterations each, varying `ndoors` from 3 to 103 by 10, with otherwise classical rule parameters.

```
# get win frequencies from varying ndoors
simi = 100; iteri = 100
ndoorl <- seq(3, 103, 10)
seedl <- seq(1, 100, 1)
lnd <- list()
t1 <- Sys.time()
for(nd in ndoorl){
  fw <- getfw(simi, iteri, nd)
  lnd[[paste0(nd)]] <- fw
}
tdif <- Sys.time() - t1
# store the reference plot
pref <- getlineplot(lnd, ptitle = "Canonical rules, varying doors")
```

All runs completed in about 5 seconds. Let's visualize results in a few different ways. First, we'll generate [violin plots](https://en.wikipedia.org/wiki/Violin_plot), a powerful way of visualizing data in a relatively distribution-agnostic manner (and thus typically better than boxplots). Next, we'll use overlapping density plots, variously called "ridge plots" or "joyplots" after their use in the iconic visualization of CP 1919 pulsar's radio waves on the cover of Joy Division's Unknown Pleasures record ([awesome!](https://en.wikipedia.org/wiki/Unknown_Pleasures#Artwork_and_packaging)). Finally, we'll show line plots of run means with confidence boundaries. While the first two plots show the exact data distributions, we'll focus on the third line plots for their economy of space and meaningful reflection of simulation distribution properties.

To generate these visualizations, I've wrapped the code inot the several plotting utilities functions `getggdat()` (format data for violin and ridge plots), `getlinedat` (format data for line plots), `getlineplot()` (generates line plot), `getprettyplots()` (generate composite of 3 ggplot2 plot types). I won't exhaustively describe these, but will note the code may be generally useful if you're looking for a generalizable way of plotting data for your own data science project. This code makes use of the supremely awesome R packages [`ggplot2`](https://cran.r-project.org/web/packages/ggplot2/index.html) (powerful plotting functions and meta syntax), [`gridExtra`](https://cran.r-project.org/web/packages/gridExtra/index.html) (managing plot outputs and composite plotting),
[`ggridges`](https://cran.r-project.org/web/packages/ggridges/index.html) (ridge plot options).

```
pdf("mh_ndoors_3plots.pdf", 10, 4)
getprettyplots(lnd, "Varying door count")
dev.off()
```

<img src="https://raw.githubusercontent.com/metamaden/montyhall/master/plots/mh_ndoors_3plots.png" align = "center" alt="mh_ndoors_3plots" width="1500"/>

This quantitatively shows the magnitude of win likelihood increase with `ndoors` increase, reinforcing our intuition about the mnemonic device. It's also interesting to note how the standard deviation converges after the means in runs with higher door counts as the win frequency increase becomes both higher and more certain.

We'll focus on the line plot visualizations below, but I've allowed for two plot types with the `ribbontype` argument. This defines the gray-colored confidence visualization to be either the standard deviation of run distribution (if `sd`, the default), or the minimum and maximum win frequencies observed (if `minmax`). Let's show these side-by-side to illustrate the difference.

```
pclassic1 <- getlineplot(lnd, ptitle = "Std. dev. overlay", ribbontype = "sd")
pclassic2 <- getlineplot(lnd, ptitle = "Min. max. overlay", ribbontype = "minmax")

pdf("mh_2lineplots.pdf", 5, 3)
grid.arrange(pclassic1, pclassic2, top = "Ribbon overlay comparison", ncol = 2)
dev.off()
```

<img src="https://raw.githubusercontent.com/metamaden/montyhall/master/plots/mh_2lineplots.png" align = "center" alt="mh_2lineplots" width="1500"/>

We'll lean on these line plot representations using distribution standard deviations to calculate the overlaid ribbons.

# What if the player doesn't always switch?

Let's now observe the impact of player switch frequency, or how often the player switches from their initial door selection. As mentioned, this is set by passing the decimal switch frequency to the `doorswitch` argument, which then parses player choice for each iteration from a weighted binomial distribution.

Let's run 10 simulations varying the switch frequency from 0% to 100% in increments of 10%. I'll store the results data in `ldat` and plots in `plist`, then show the results in a composite plot. For the plot, note that I've set the x- and y-axis ranges in `getlineplot` to be identical for the 10 results plots to make visual comparison somewhat easier.

```
# get fwin dist across ndoors
plist <- list()
sfreq <- seq(0, 1, 0.1)
t1 <- Sys.time()
ldat <- list()
for(s in sfreq){
  simi = 100; iteri = 100
  ndoorl <- seq(3, 103, 10)
  seedl <- seq(1, 100, 1)
  lnd <- list()
  for(nd in ndoorl){
    fw <- getfw(simi, iteri, nd, doorswitch = s)
    lnd[[paste0(nd)]] <- fw
  }
  plist[[paste0(s)]] <- getlineplot(lnd, ptitle = paste0("S.F. = ", s),
                                    xlim = c(0, 100), ylim = c(0, 1),
                                    xlab = "", ylab = "")
  ldat[[paste0(s)]] <- lnd
  # message(s)
}
tdif <- Sys.time() - t1
```

All runs completed in about 1 minute. The composite plot is then generated from the `plist` plots list as follows.

```
pdf("mh_switchfreq.pdf", 10, 6)
grid.arrange(plist[[1]], plist[[2]], plist[[3]],
             plist[[4]], plist[[5]], plist[[6]],
             plist[[7]], plist[[8]], plist[[9]],
             plist[[10]],
             ncol = 5, 
             bottom = "Number of Doors", left = "Win Fraction")
dev.off()
```

<img src="https://raw.githubusercontent.com/metamaden/montyhall/master/plots/mh_switchfreq.png" align = "center" alt="mh_switchfreq" width="1500"/>

Across run sets of each door switch frequency, there's a clear transition from an approximate negative power function (e.g. x ^ -1, top leftmost plot), to something approaching a fractional power function (e.g. x ^ 1/2, bottom rightmost plot). 

Increasing run switch frequency incrementally under classical rules should show progressive increase in win fraction distributions. Let's generate and visualize the simulation results for this. Note, I'll appropriate my `getlineplot()` function for this as written, but in some applications it can be better to add code that explicitly handles specific axis variables like `ndoors` and `doorswitch`. The resulting plot shows a clear linear win fraction increase with switch frequency, maxing out at the now-familiar 2/3rds fraction.

```
sfreq <- seq(0, 1, 0.1)
lnd <- list()
for(s in sfreq){
  simi = 100; iteri = 100
  seedl <- seq(1, 100, 1)
  fw <- getfw(simi, iteri, doorswitch = s)
  lnd[[paste0(s)]] <- fw
}
pdf("mh_switchfreq_classicrules.pdf", 4, 4)
getlineplot(lnd, ptitle = "Win Freq. by Switch Freq.", 
            xlim = c(0, 1), ylim = c(0, 1), 
            xlab = "Switch frequency")
dev.off()
```

<img src="https://raw.githubusercontent.com/metamaden/montyhall/master/plots/mh_switchfreq_classicrules.png" align = "center" alt="mh_switchfreq_classicrules" width="900"/>

# Conclusions and analysis extensions

We've explored simulations of the Monty Hall problem using a brute force approach. By exploring changes in win frequency across varying problem conditions, we've proven that always switching doors will increase player win frequency. We've also quantitatively shown how improved win frequency converges as the number of doors is increased. Finally, we explored the role of certain conditions to the problem itself. Unsuprisingly, as player switch frequency increases, so too does win frequency. In so doing, we showed how switch frequency increase leads to different win frequency improvements across doors. 

This brute force simulation approach is one of many possible ways of sloving and exploring the Monty Hall problem, and alternate approaches implementing Bayesian models could lead to further interesting insights. There are several other game conditions that could also be explored. These include changing the total number of doors with prizes, for games of at least 4 doors. Ultimately, I hope this investigation provided some useful code that equips you with a framework for investigating new problems through simulation.

# Bonus plot animations

In data science, more tools in our toolkit means more options for tackling future problems. In this section I'll show how some results from sequential experiments testing some parameter range (e.g. increasing `ndoors` or `doorswitch`, etc.) lend themselves to nice animations. For this task, I've written the function `getprettygifs()`, which uses the R packages `gganimate` and `magick` and leans heavily on the helpful code provided [here](https://github.com/thomasp85/gganimate/wiki/Animation-Composition) for the composite gif. Passing options to `plottype` results in either a composite plot of the `ndoors` data (violin and line plots), or a line plot showing how `doorswitch` increase impacts win fraction across games varying `ndoors`.

Let's use the plot gif function to generate the gif files. Note all the data is contained in the `ldat` object, where the final item shows win fraction in relation to door count when the player always switches.

```
getprettygif(ldat[[11]], plottype = "composite_ndoors", gifname = "mh_ndoors.gif")
getprettygif(ldat, plottype = "lineplots_doorswitch", gifname = "mh_switchfreq.gif")
```
![https://raw.githubusercontent.com/metamaden/montyhall/master/plots/mh_ndoors.gif](https://raw.githubusercontent.com/metamaden/montyhall/master/plots/mh_ndoors.gif)

![https://raw.githubusercontent.com/metamaden/montyhall/master/plots/mh_switchfreq.gif](https://raw.githubusercontent.com/metamaden/montyhall/master/plots/mh_switchfreq.gif)