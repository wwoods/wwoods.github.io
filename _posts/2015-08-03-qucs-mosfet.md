---
layout: post
title: Qucs - Freely Combining Schematics and Simulation
tags: qucs electronics schematics
eye_catch: http://qucs.sourceforge.net/images/logos/qucs-banner-2.png
---

[Qucs](http://qucs.sourceforge.net) (also hosted on [GitHub](https://github.com/Qucs/qucs)) is a "(Q)uite (u)niversal (c)ircuit (s)imulator" that lets you both layout schematics and simulator the represented circuit.  Best of all, it's completely free.

I'm not aware of any other tool that matches up with its capabilities.  There are other tools, but they have downfalls as far as I am concerned.  I only spent 5-10 minutes investigating each piece of software; at the end, I decided Qucs was the best fit for my habits.  Your mileage may vary, so here's a quick list of the other tools I looked at:

<!--more-->

* Digikey has a tool, [PartSim](http://www.partsim.com), that lets you lay out circuits and simulate them.  It's pretty sleek too, but it's online only.  This means that if the user needs to optimize a parameter, they cannot simply write a script to automate the process.  This was something I needed.

* [EAGLE PCB](http://www.cadsoftusa.com/download-eagle/freeware/) has a freeware version, and seems to have accompanying simulator.  It's what most people at my university use.  Strictly speaking, the software is not free, as there are restrictions on the free version.  Additionally, it didn't seem to have the same parametrization capabilities that Qucs presents.  However, I only gave it a cursory look, so it's possible that I simply missed some better functionality that EAGLE possesses, but takes longer to learn.  In that regard, for the novice, Qucs shines.

* [gEDA](http://www.geda-project.org) is a free, "full GPL'd suite and toolkit of Electronic Design Automation tools."  The part that I was interested in was `gschem`, the schematic component of the suite.  I found the interface to be annoying and archaic.  Again, that very well might be a lack of time invested.  User interface design practices have matured a lot over the years though, and generally I want to get straight to utilizing my expertise rather than spending a lot of time learning a user interface.

In general, Qucs provides everything needed to prototype and document circuit designs: a reasonable UI experience, a fairly comprehensive components library, project management, excellent support for subcircuits, and high-quality vector graphics exports.  The way that Qucs integrates subcircuits into its project management tools is extremely convenient for larger circuits, and when scripting simulations it makes things a lot easier.

Despite having elected to use Qucs for my master's thesis work, there are a few pitfalls that I would like to mention:

* Qucs lacks SPICE-compatible parameterized MOSFETs.  The default MOSFET model in Qucs is Level 1, the Shichman-Hodges model.  Faster than the typical BSIM3/4 models found in SPICE, but unfortunately I have had a terrible time finding parameter sets for circuit design.  BSIM models, on the other hand, have many [available models with realistic parameters](http://ptm.asu.edu/latest.html) to put into your circuits.  Technically, Qucs supports BSIM4 as a Verilog-A extension component (built-in), but I have not had much luck using it in a circuit.  Also, some of the parameters in Qucs' model differs from those found in the SPICE parameter list.  Also also, filling out parameters for a Qucs model is very time consuming and obnoxious.  They're working on [direct SPICE integration](https://github.com/Qucs/qucs/issues/77), which will be a nice, scriptable workaround for this issue.

* Some of the mouse operations don't work quite as well as I'd like; better than the previously-mentioned applications, but not fully tuned.  For instance, on Mac laptops, a horizontal scroll is typically accomplished via the two-finger horizontal swipe gesture.  Other touchpads often support that as well, but Qucs doesn't support it.  It's a minor complaint, and one that, when/if I have more time, I might submit a patch for myself.

Other than that, everything works great, and I'd strongly recommend that anyone looking to make schematics (with or without needing to simulate them) should take a look at Qucs.

