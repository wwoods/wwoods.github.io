---
layout: post
title: Qucs - Not Quite Universal Enough for CMOS Simulations
tags: qucs electronics schematics mosfet cmos ngspice
# eye_catch: http://qucs.sourceforge.net/images/logos/qucs-banner-2.png
---

[Qucs](http://qucs.sourceforge.net), which I wrote about in my [last post]({% post_url 2015-08-03-qucs-mosfet %}), is still my tool of choice for generating nice-looking schematics.  However, I have found the simulator to be lacking in a couple of ways for CMOS circuit simulation:

<!--more-->

1. CMOS MOSFET parameters, such as those attainable from Arizona State University's wonderful [Predictive Technology Model](http://ptm.asu.edu/) page, are very difficult to input into Qucs.  Specifically, each parameter has to be carefully transcribed into a Qucs MOSFET element from the SPICE file generated through the PTM webpage.  Qucs' input mechanism for these types of parameters (of which there are more than 100) is very slow and painful.

2. Qucs' [default MOSFET model](http://qucs.sourceforge.net/tech/node71.html) is based on equations by Harold Shichman and David A. Hodges, and is roughly equivalent to SPICE's MOSFET level 1.  This model, while faster, produces "severe inaccuracies" (quoted from [here](http://web.engr.oregonstate.edu/~moon/ece323/hspice98/files/chapter_16.pdf)) in certain situations, and more importantly for my work does not support the wide variety of parameters provided through ASU's PTM.

3. The dominant model downloadable on PTM, in SPICE represented as a level 54 MOSFET, is a BSIM4.0 model.  Qucs supports BSIM4.0 through its support for Verilog-A devices within schematics.  However, the provided Verilog-A model for BSIM4.0, found under "verilog-a devices" -> "bsim4v30..." in Qucs' Components list, also does not seem to expose all of the parameters provided by the PTM webpage.

It is possible that these shortcomings might be overcome through a dedicated effort with Qucs, however the fact remains that with Qucs as it is now, simulating CMOS schematics using nanoscale transistors would require a lot of work.  Therefore, for my work, I am going back to [Ngspice](http://ngspice.sourceforge.net/), which is rather stable and effective for these sorts of circuits.  I would much prefer to work with schematics than netlists, but given the aforementioned limitations of Qucs' default environment, a lot of time is saved by working with netlists rather than trying to bend the MOSFET models within Qucs to a purpose for which they don't seem to be intended.
