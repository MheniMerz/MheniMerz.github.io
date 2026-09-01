---
layout: post
section-type: post
title: "Eight users on one chip : the source side of a flex-grid quantum network"
category: 'networking'
tags: [ 'networking', 'fiber', 'quantum' ]
---

[the last post]({% post_url 2026-04-04-Flex-grid-for-entanglement %}) was about the switch. one wavelength selective switch in the middle of a quantum network, replacing a tree of passive filters, so the operator can decide who gets how much entanglement by pushing a config instead of driving to the site.

that post skipped a question. the switch divides up a spectrum. where does the spectrum come from, and how much of it is there?

this post is about the other 2021 paper, the one that answered the source half. [Appas et al., "Flexible entanglement-distribution network with an AlGaAs chip for secure communications," *npj Quantum Information* 7, 118 (2021)](https://doi.org/10.1038/s41534-021-00454-7). a Paris group with Nokia Bell Labs on the author list, a semiconductor chip emitting entangled pairs across 60 nm of the telecom band, the same kind of switch in the middle, and, unlike the Purdue/Oak Ridge line, an actual key.

the two papers came out four months apart, and this one cites the other. they are the same architecture approached from opposite ends. the Optica paper had a good switch and a narrow source, and spent its effort on allocation policy. this one has a wide source and spends its effort on proving the links carry secret key.

below: how much spectrum a network actually needs, what the chip provides, what the key rates look like when you count honestly, and then what i think is missing.

## what the switch is spending

quick recap, and [the previous post]({% post_url 2026-04-04-Flex-grid-for-entanglement %}) has the long version. a source in the middle makes photon pairs, and the two photons of a pair always add up in energy to the pump:

```
    ω_signal  +  ω_idler  =  ω_pump
```

so if Alice gets a slice of spectrum sitting a little above the center of the band, her photons' partners are all sitting the same distance below it. give that lower slice to Bob and Alice and Bob share entanglement. give it to Carol and Alice and Bob share nothing. you connect two users by handing them the two halves of the same energy budget, which is why this is a spectrum allocation problem rather than a routing problem.

now count. a fully connected network of N users has N(N−1)/2 two-user links. every link needs its own conjugate pair of slices, one on each side of degeneracy. so if the grid is Δ wide, the spectrum you need on each side of center is:

```
    B_half  =  N(N-1)/2  ×  Δ
```

that is the whole scaling story in one line. **the number of users you can serve is set by the source bandwidth divided by the channel width, and it goes as the square root, because the link count is quadratic.** doubling your source gets you 40% more users, not twice as many.

now the constraint. the paper's switch is a Finisar WaveShaper 4000S, and it is a C band part. the paper puts its upper cutoff at 1565 nm. the chip's degeneracy sits at 1556.55 nm, and channels have to be symmetric about degeneracy, so the reachable window is bounded by whichever band edge is closer: about 11 nm on each side of center, call it 1400 GHz.

that number is a back-calculation, not something the paper states. but it explains every configuration in it.

<div class="qw" id="qw-fit" data-qw><div class="qw-hd"><span class="qw-t">the same window, carved finer</span><span class="qw-s">a fully connected network needs N(N&minus;1)/2 conjugate channel pairs. pick a grid and the user count follows.</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">the paper&rsquo;s three configurations</span><span class="qw-b" data-grp="ps"><button type="button" data-v="200">4 users, 200 GHz</button><button type="button" data-v="100" class="on" aria-pressed="true">5 users, 100 GHz</button><button type="button" data-v="50">8 users, 50 GHz</button><button type="button" data-v="12.5">the 12.5 GHz floor</button></span></div></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">spectrum you can actually reach</span><span class="qw-b" data-grp="bd"><button type="button" data-v="1400" class="on" aria-pressed="true">C band through the WSS, 1400 GHz</button><button type="button" data-v="3600">the whole 60 nm chip, 3600 GHz</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">channel width <span class="qw-gk">&Delta;</span> <b id="qwf-wv">100 GHz</b></span><input type="range" id="qwf-w" min="0" max="100" step="0.5" value="60"></label></div><div class="qw-two"><div class="qw-pane"><span class="qw-pl">the half band, one slice per link</span><svg class="qw-svg" viewBox="0 0 320 150" preserveAspectRatio="xMidYMid meet" role="img" aria-label="The usable half band divided into one slice per two-user link"><g id="qwf-spec"></g></svg></div><div class="qw-pane"><span class="qw-pl">the network that buys you</span><svg class="qw-svg" viewBox="0 0 320 150" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Complete graph of the resulting network"><g id="qwf-graph"></g></svg></div></div><div class="qw-out"><div class="qw-oi"><span class="qw-ok">users</span><span class="qw-ov" id="qwf-n">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">two-user links</span><span class="qw-ov" id="qwf-e">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">key rate per link, 0 km</span><span class="qw-ov" id="qwf-r">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">finite-key reach</span><span class="qw-ov" id="qwf-d">&mdash;</span></div></div><p class="qw-n" id="qwf-note"></p><noscript><p class="qw-n">With the 1400 GHz reachable on each side of degeneracy through a C-band WSS: 200 GHz channels support 4 users, 100 GHz supports 5, and 50 GHz supports 8, which are exactly the three networks the paper built. Going to the switch&rsquo;s 12.5 GHz floor would support 15. Using the chip&rsquo;s whole 60 nm instead would support 24.</p></noscript></div>

200 GHz gives four users, 100 GHz gives five, 50 GHz gives eight. those are the paper's three demonstrated networks, and all three spend the same window. the 8-user case uses it exactly: 28 links times 50 GHz is 1400 GHz, with nothing left over.

so the three configurations are not three experiments. they are one window, cut three ways.

## the chip

the source is an AlGaAs Bragg reflection waveguide. 4 mm long, 5 μm wide ridge, wet etched, pumped by a tunable diode laser at 778 nm through a microscope objective, held at 19.3 °C by a Peltier and a PID loop to pin degeneracy at 1556.55 nm.

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Property</th><th>Value</th><th>Why it matters</th></tr>
</thead>
<tbody>
<tr><td>Pair rate at the chip</td><td>~10<sup>7</sup> s<sup>&minus;1</sup></td><td>the number that gets quoted</td></tr>
<tr><td>Brightness</td><td>3.6 &times; 10<sup>5</sup> pairs/s/mW</td><td>the number that gets compared</td></tr>
<tr><td>Heralding efficiency</td><td>9.4%</td><td>the number that decides the key rate</td></tr>
<tr><td>Biphoton FWHM</td><td>60 nm, ~72 ITU 100 GHz channels</td><td>36 conjugate pairs, in principle</td></tr>
<tr><td>Degeneracy</td><td>1556.55 nm (ITU CH26)</td><td>near the top of the C band, which is a problem</td></tr>
<tr><td>Raw visibility</td><td>98.0% (Z), 97.7% (X)</td><td>sets the QBER floor at about 1%</td></tr>
<tr><td>Walk-off compensation</td><td>none needed</td><td>this is the actual selling point</td></tr>
</tbody>
</table>
</div>

that last row is the one to pay attention to. type-II downconversion produces the two photons in orthogonal polarizations, and in most materials those two polarizations travel at different speeds, so they come out of the crystal separated in time. that timing difference is a label. it tells you which photon is which, and a photon you can label is a photon that is not entangled with anything.

the usual fix is to undo the walk-off off-chip. the Optica paper did it with a 90 degree splice of polarization maintaining fiber, so that the second half of the fiber delays whichever polarization got ahead in the first half. it works, but it is another component, it has to be built for a particular source, and it is one more thing that drifts.

AlGaAs has small enough group velocity mismatch between the two guided modes that the walk-off never gets big enough to matter over 4 mm. the pairs come out of the facet already in a |Ψ⁺⟩ Bell state. you couple to fiber and you are done.

that sounds like a detail and it is not. the entire source is one chip, one pump diode and a temperature controller. the paper points out that this material also supports [electrical injection](https://doi.org/10.1103/PhysRevLett.112.183901), which this group had already demonstrated, so the pump laser could eventually be on the same die.

### where the bandwidth goes

60 nm is the FWHM of the pair spectrum. it is not 60 nm of usable entanglement, and the paper is careful about the difference.

fidelity to |Ψ⁺⟩ stays above 95% over the middle 26 nm, which is 13 conjugate channel pairs on a 100 GHz grid. it stays above 85% right out to the edges of the 60 nm band, but with visibly larger error bars and only three data points in the far wing, because the L-band measurement had to be done with a coarse WDM unit and a tunable filter rather than the switch.

two things cause the roll-off. the waveguide facets are uncoated, so the chip is a weak Fabry-Perot cavity and the joint spectrum picks up ripple. and there is residual birefringence between the H and V modes, which grows with detuning and gradually turns the Bell state into something less symmetric. the paper models both, gets very good agreement, and says plainly that an anti-reflection coating fixes the first one.

then the harder limit, which is the one from the top of the post. **because channels have to sit symmetrically about degeneracy, the reachable window is set by whichever band edge is closer**, and here that is the C-band ceiling 11 nm above. the QKD measurements got 13 conjugate pairs out of the 36 the chip emits.

the paper says this outright: the range "is limited by the upper cutoff wavelength (1565 nm) of the WSS ... and not by the spectral bandwidth of the generated biphoton state." it also names the fix, which is to retune degeneracy to the middle of the C band. it is a temperature and pump wavelength adjustment. nobody has to invent anything.

but the 60 nm number is what gets cited, and the 11 nm window is what got built.

## running BBM92 on it

the benchmark application is [BBM92](https://doi.org/10.1103/PhysRevLett.68.557), which is the entanglement version of BB84. both users measure their photon in one of two randomly chosen bases. where they happened to pick the same basis, their results are correlated and become key. where they picked differently, they throw it away, which is where the factor of 1/2 comes from. the useful property is that no one, including whoever runs the source in the middle, needs to be trusted: the correlations either pass the test or they do not.

two numbers come out. the **QBER** is the fraction of the sifted bits that disagree, and it is your eavesdropper budget: above about 11% there is no key at all. and the **key rate** is what survives error correction and privacy amplification.

across the 13 reachable channel pairs, right at the demux stage, the QBER stays below 2% and the asymptotic key rate sits between **28 and 39 bits per second**. flat across the whole range, which is the point being made. flatness comes from the source spectrum being flat there and from the switch having wavelength-independent loss.

### asymptotic is not the number you want

that 28 to 39 bits/s is an asymptotic rate. it assumes an infinitely long key, which lets you ignore the fact that estimating an error rate from a finite sample has error bars, and that you have to be conservative about those error bars or your security proof does not hold.

with real blocks you pay for that. the paper redoes the analysis properly, with 10 minute blocks and correctness and secrecy parameters of 10⁻¹⁰ each, following [Tomamichel et al.](https://doi.org/10.1038/ncomms1631). the penalty scales roughly as 1/√n in the block size, so it is mild when you are collecting thousands of bits and brutal when you are collecting hundreds.

this is where the paper's honesty shows. the asymptotic key rate stays positive out to 250 km of equivalent fiber. the finite-key rate dies at **75 km**.

<div class="qw" id="qw-dist" data-qw><div class="qw-hd"><span class="qw-t">where the key rate actually dies</span><span class="qw-s">a model calibrated to the paper&rsquo;s four stated landmarks: 28&ndash;39 bits/s at 0 km, QBER under 2% to 50 km, asymptotic key to 250 km symmetric and 215 km asymmetric, finite key to 75 km.</span><span class="qw-lg"><i style="background:#32c29e"></i>asymptotic<i style="background:#f2a03d"></i>finite key, 10 min blocks<i style="background:#6ba3f0"></i>QBER</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">link geometry</span><span class="qw-b" data-grp="sy"><button type="button" data-v="1" class="on" aria-pressed="true">symmetric</button><button type="button" data-v="0">all the fiber on one side</button></span></div><div class="qw-g"><span class="qw-l">network size sets the grid</span><span class="qw-b" data-grp="ps"><button type="button" data-v="200">4 users, 200 GHz</button><button type="button" data-v="100" class="on" aria-pressed="true">5 users, 100 GHz</button><button type="button" data-v="50">8 users, 50 GHz</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">distance between the two users <b id="qwd-lv">50 km</b></span><input type="range" id="qwd-l" min="0" max="280" step="1" value="50"></label><label class="qw-sr"><span class="qw-l">block size <b id="qwd-tv">10 min</b></span><input type="range" id="qwd-t" min="1" max="120" step="1" value="10"></label></div><div class="qw-pane"><span class="qw-pl">secret key rate and QBER vs distance</span><svg class="qw-svg" viewBox="0 0 640 260" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Secret key rate and quantum bit error rate versus fiber distance"><g id="qwd-grid"></g><g id="qwd-curves"></g><g id="qwd-mark"></g></svg></div><div class="qw-out"><div class="qw-oi"><span class="qw-ok qw-bs">asymptotic</span><span class="qw-ov" id="qwd-ra">&mdash;</span></div><div class="qw-oi"><span class="qw-ok qw-fs">finite key</span><span class="qw-ov" id="qwd-rf">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">QBER</span><span class="qw-ov" id="qwd-q">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">secret bits per block</span><span class="qw-ov" id="qwd-b">&mdash;</span></div><div class="qw-oi qw-grow"><span class="qw-ok">verdict</span><span class="qw-ov qw-txt" id="qwd-v">&mdash;</span></div></div><p class="qw-n" id="qwd-note"></p><noscript><p class="qw-n">On 100 GHz channels the asymptotic key rate stays positive to about 250 km symmetric and 215 km asymmetric, while the finite-key rate with 10 minute blocks dies at 75 km in both cases. Halving the channel width to fit 8 users instead of 5 pulls that 75 km back to about 63 km; doubling it to 200 GHz for a 4-user network pushes it out to about 91 km.</p></noscript></div>

there is a 175 km gap between those two numbers and the gap is not a rounding error. it is the difference between "a key exists in principle" and "a key exists before the customer gives up waiting". **at 75 km the link is producing a few dozen secret bits per ten minute block.** at 50 km it is a few hundred. you could re-key something on a slow schedule with that. you could not run a VPN over it.

### symmetric versus asymmetric

the paper also asks whether it matters where you put the fiber. two users each 25 km from the source, or one user next door and the other 50 km away?

for the asymptotic rate it matters: positive key to 250 km symmetric, 215 km asymmetric. the mechanism is worth understanding because it is not about the signal.

true coincidences depend only on the product of the two transmissions, so the total attenuation is all that counts and both geometries look identical. accidental coincidences do not work that way. an accidental is two unrelated detections landing in the same window, so its rate goes as the product of the two users' *singles* rates. in the symmetric case both singles rates fall together, so accidentals fall as the square root of the loss twice over. in the asymmetric case the near user keeps clicking at full rate no matter how bad the far link gets, and the accidental floor is set by that near user's singles times the far user's dark counts. the far user's noise gets multiplied by a large number instead of a small one.

so asymmetry costs you about 35 km of asymptotic reach. and then the finite-key analysis makes the whole distinction moot, because **both geometries die at 75 km for the same reason, which is that neither is producing enough bits per block**. you run out of statistics before you run out of signal-to-noise.

for a deployed network that is good news, and the paper says so. it means a source sitting in an exchange with some users nearby and some far away is not a pathological case. it is just a case.

## four, five and eight users out of the same window

the multi-user demonstration is three configurations of the same hardware: 4 users on 200 GHz channels, 5 on 100 GHz, 8 on 50 GHz. for each one they measured the time correlation histogram on every two-user link.

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Users</th><th>Links</th><th>Channel width</th><th>Spectrum used per side</th><th>Spread across links</th></tr>
</thead>
<tbody>
<tr><td>4</td><td>6</td><td>200 GHz</td><td>1200 GHz</td><td>7% relative std dev</td></tr>
<tr><td>5</td><td>10</td><td>100 GHz</td><td>1000 GHz</td><td>9%</td></tr>
<tr><td>8</td><td>28</td><td>50 GHz</td><td>1400 GHz</td><td>11%</td></tr>
</tbody>
</table>
</div>

that last column is the result. every link on an 8-user network carries within about 11% of the same rate as every other link, and the reconfiguration between these three networks was a profile loaded into the switch.

compare that to a passive filter tree, where channel transmission varies with which filter port a photon happened to come out of, and where the [Optica paper's fixed grid](https://doi.org/10.1364/OPTICA.413657) measured a factor of 4200 between its best and worst link. most of that came from the detectors rather than the optics, but the structural point holds: a passive tree has a different loss on every path and an active switch has the same loss on all of them.

they also check that narrowing channels does not hurt entanglement. fidelity is flat against channel width, QBER is flat, and the key rate scales linearly. which is what you would hope, and worth verifying, because it means the grid is a free parameter rather than a physics tradeoff.

the floor is set somewhere below 12.5 GHz by an argument i like: a narrower slice is a longer photon, and a longer photon smears the coincidence peak. at 12.5 GHz the coincidence histogram is about 100 ps wide, which is still narrower than the 200 ps timing jitter of the detectors. so the filtering is free until the photon gets longer than your detector's uncertainty about when it arrived, and only then do you start paying in noise.

## leveling an unbalanced network

the last experiment is the one that makes this a networking paper.

take the 5-user network. move one user, B, 25 km down a fiber spool and leave the other four next to the source. B is on four of the ten links, and all four of them are now about 5 times worse than the six links that never leave the building.

<div class="qw" id="qw-lvl" data-qw><div class="qw-hd"><span class="qw-t">flatten it, and see what it costs</span><span class="qw-s">five users, ten links, one of them (B) sitting 25 km down a fiber. 1400 GHz of reachable spectrum to divide. rates are calibrated to the paper&rsquo;s Fig. 6c.</span><span class="qw-lg"><i style="background:#32c29e"></i>links between local users<i style="background:#e2603f"></i>links to the distant user</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">allocations worth trying</span><span class="qw-b" data-grp="ps"><button type="button" data-v="a" class="on" aria-pressed="true">the paper, before</button><button type="button" data-v="b">the paper, after</button><button type="button" data-v="c">spend the idle band on B</button><button type="button" data-v="d">throughput first</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">width per link to B <b id="qwv-bv">100 GHz</b></span><input type="range" id="qwv-b" min="10" max="330" step="2.5" value="100"></label><label class="qw-sr"><span class="qw-l">width per local link <b id="qwv-ov">100 GHz</b></span><input type="range" id="qwv-o" min="5" max="230" step="2.5" value="100"></label></div><div class="qw-pane"><span class="qw-pl">spectrum budget, 1400 GHz per side</span><svg class="qw-svg" viewBox="0 0 640 54" preserveAspectRatio="xMidYMid meet" role="img" aria-label="How the spectrum budget is divided"><g id="qwv-bar"></g></svg></div><div class="qw-pane"><span class="qw-pl">coincidence counts per link, 30 s</span><svg class="qw-svg" viewBox="0 0 640 190" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Coincidence counts on each of the ten two-user links"><g id="qwv-bars"></g></svg></div><div class="qw-out"><div class="qw-oi"><span class="qw-ok">worst link</span><span class="qw-ov" id="qwv-min">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">imbalance</span><span class="qw-ov" id="qwv-r">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">network total</span><span class="qw-ov" id="qwv-tot">&mdash;</span></div><div class="qw-oi qw-grow"><span class="qw-ok">verdict</span><span class="qw-ov qw-txt" id="qwv-v">&mdash;</span></div></div><p class="qw-n" id="qwv-note"></p><noscript><p class="qw-n">At a fixed 100 GHz everywhere the six local links run about 3350 coincidences per 30 s and the four links to the distant user about 700, a factor of 4.8. Giving B&rsquo;s links 267 GHz each and the local links 56 GHz each brings all ten to about 1870, but the network total falls by roughly 40% compared with spending the same spectrum on equal channels.</p></noscript></div>

they reallocate on a 12.5 GHz granularity, give B's links more bandwidth and the local links less, and bring all ten to within about 10% of each other. that is the result, and it is a good one. the same knob would work just as well to deliberately favor a link instead of leveling them.

two things worth noticing that the paper does not dwell on.

**part of the fix was free.** at a fixed 100 GHz the ten links used 1000 GHz of a roughly 1400 GHz window, so about 400 GHz was sitting unallocated. giving that idle spectrum to B's four links alone takes the imbalance from 4.8 down to about 2.4 without touching anyone else's channels. the rest of the way to flat is what actually costs the local users something.

**the last part of the fix is expensive.** leveling this network throws away roughly 40% of its total coincidence budget compared with spending the same spectrum on equal channels. that is not a criticism of the result, it is the arithmetic of fairness: you are moving spectrum from links where a GHz is worth full value into links where a GHz is worth a fifth of that, because it has to survive 25 km first. **on a lossy network you pay for fairness in aggregate throughput, and the exchange rate is the loss ratio between the links you are trying to equalize.**

that is a number an operator would want on the invoice, and it is missing.

## what's missing

the paper is good and it is honest about most of this. some of what follows is a limitation it names and does not solve, which is fair for a proof of principle. some of it is a gap between what got measured and what the title claims.

**the two headline results were never composed.** the 8-user network was measured at 0 km, on a bench, with one time-to-digital converter. the QKD-over-distance results were two users on 100 GHz channels. nobody ran an 8-user network over metro fiber with finite-key accounting, and the model above says why it would have been awkward: halving the channel width to fit 8 users pulls the finite-key reach from about 77 km back to about 63 km, because the finite-size penalty is superlinear in how few bits you collected. **the reach of this network is a function of how many people are on it**, which is an interesting and unstated result sitting inside the paper's own data.

**it is a characterization, not a QKD session.** the basis choice was not random. the waveplates were set by hand for each of the 8 projective measurements. the links were measured one at a time because there were not enough detectors to run them in parallel, and the polarization drift was compensated manually at each fiber length. all of that is reasonable for a first demonstration and all of it is stated. but it means there is no measurement of the thing that actually matters operationally, which is what happens over the following six hours as the spool warms up.

**9.4% is the number, not 10⁷.** the chip makes ten million pairs a second and the user gets thirty secret bits a second. most of the gap is heralding efficiency, and heralding efficiency enters a two-user link *squared*, because both photons have to survive. a source with half the brightness and twice the heralding efficiency would beat this one by a factor of two on every link in the network. brightness is the number that gets compared between papers and it is close to the wrong one.

**two thirds of the advertised band is out of reach.** 36 conjugate pairs emitted, 13 usable, because degeneracy sits near the top edge of a C-band switch. the paper names this and names the fix. the citation record mostly quotes the 60 nm.

**the reallocation is "a simple algorithm".** that is the entire description. what it optimizes, under what constraint, against what baseline: none of that is in the paper. meanwhile the [other 2021 paper's line of work](https://arxiv.org/abs/2404.08744) turned into a formal routing and spectrum assignment problem, proved max-min fair allocation NP-hard, and produced [solvers that run on carrier topologies](https://arxiv.org/abs/2607.15465). this paper had the better source and stopped at the demo.

**the propagation time problem is named and left.** in a real session each user's N−1 channels land on one detector, and each of those channels has a different delay through the switch and the fiber. so either you open one wide coincidence window and eat the extra noise, or you equalize the delays per channel. the paper points at [Joshi et al.'s supplement](https://doi.org/10.1126/sciadv.aba0959) for the fix and does not implement it. with 28 links at 50 GHz that means 28 delays to measure and 28 to compensate, per user, and it has to hold while the fiber moves.

**there is no clock, because there did not need to be.** every detector was in the same lab on the same TDC. the synchronization problem that [the coexistence post]({% post_url 2026-03-23-coexistence-running-a-quantum-channel-and-its-clock-down-the-same-fiber %}) is about does not appear anywhere in this paper, and neither does classical traffic in the same fiber. both of those turn up as soon as the spools are replaced by real glass.

## what i'd build next

**1. run the composed experiment.** eight users, 50 GHz channels, deployed metro fiber, all links live at once, finite-key accounting. that single measurement is what separates a network from a demonstration, and i suspect the result would be the useful kind of disappointing: positive key on the short links, nothing on the long one, and an operator who now needs a policy for that. five years on i still cannot find it published for a flex-grid source.

**2. AR coat the facets and move degeneracy to the middle of the C band.** the paper asks for both. the coating removes the cavity ripple that limits fidelity far from center; the retuning roughly doubles how much of the band a C-band switch can reach. together they take the reachable pair count from 13 toward the mid thirties, which on the arithmetic in the first widget is the difference between five users and nine. it is by a distance the cheapest item on this list, and it needs no new physics.

**3. put the finite-key block size inside the allocator.** this is the one i would actually write.

every flex-grid allocator in the literature optimizes coincidence rate, or ebits per second, or fidelity. all three are smooth functions of allocated bandwidth. **secret key after finite-size correction is not.** below a threshold rate a link produces exactly zero secret key at any block size you are willing to wait for, and just above it the return on an extra GHz is enormous. that discontinuity changes what an optimal allocation looks like: there are allocations that maximize total ebits and produce no key on any link, and allocations that produce less entanglement and more key.

it also turns admission control from a policy preference into a hard constraint, which is the thing i said was missing in [the flex-grid post]({% post_url 2026-04-04-Flex-grid-for-entanglement %}). a solver that knows about block sizes can tell a user at request time that their link is infeasible, and can offer them a longer block instead of nothing.

**4. publish η² Δ per link, not brightness.** the honest figure of merit for a distribution network is the product of the two endpoints' heralding efficiencies times the bandwidth allocated to their link. it is measurable at the endpoints, and unlike brightness it composes, so two networks built on entirely different source technologies can be compared on it.

**5. price the fairness.** the leveling experiment should come with the number i computed above, or the authors' own version of it. how many coincidences per second did the network give up to close the gap, and what is the exchange rate. once that number exists you can ask the question that follows from it. should the operator have leveled at all, or sold the distant user a lower service class and spent the spectrum where it was worth more?

**6. two knobs, not one.** the switch decides which slices go where. the pump decides where the spectrum sits and how wide it is, and here the pump is a tunable diode and the chip is temperature tuned, so both are already software. a group in Beijing showed you can [reconfigure an entanglement network with pump management alone](https://doi.org/10.1126/sciadv.ado9822), no switch involved. nobody has an allocator that uses both, and they are interestingly different: retuning the switch moves one link, retuning the pump moves everyone at once. that is a slow outer loop and a fast inner loop, which is a structure every control engineer would recognize and no quantum network has yet.

**7. decide what the L band is for.** the chip emits across C and L. the tooling is C only. either somebody builds a C+L flex-grid switch for this market, which exists classically and would cost real money, or the L band gets deliberately assigned to something else in the same fiber: the White Rabbit clock, the classical control plane, the polarization pilot tone that the [now-solved](https://doi.org/10.1103/PRXQuantum.5.030330) stabilization schemes need. right now the source's best feature is stranded, and the second option is more interesting than the first because it makes the source's bandwidth do work even when the quantum channel cannot use it.

## what happened since

the pieces this paper was missing mostly exist now, scattered across other groups.

polarization drift is handled now. [automated distribution of polarization-entangled photons over deployed New York City fiber](https://doi.org/10.1103/PRXQuantum.5.030330) (PRX Quantum 2024) and, more recently, [entanglement distribution over a polarization-stabilized aerial fiber](https://arxiv.org/abs/2601.11753) mean nobody has to touch waveplates by hand any more. there is also [continuous stabilization from a dim coexisting reference signal](https://arxiv.org/abs/2411.15135), which is the one that costs you spectrum and therefore belongs in the allocator.

and the fully connected topology scaled, though not with a switch. [Liu et al. built a 40-user network](https://doi.org/10.1186/s43074-022-00048-2) (PhotoniX 2022) as five interconnected 8-user subnets, using a passive AWG rather than a WSS, getting about 51 bits/s within a subnet and 22 between. that is the non-fully-connected topology this paper proposed in its own discussion, built by someone else with a different source, and their stated limit is detector count rate rather than bandwidth.

meanwhile the chip kept going. the same Paris group used a multiplexed AlGaAs source to demonstrate [triangle-network nonlocality on fiber](https://journals.aps.org/prxquantum/abstract/10.1103/PRXQuantum.6.020313) (PRX Quantum 2025), which is a genuinely different application of the same energy-matching trick: three frequency pairs, three parties, no inputs. and a group in China built a [five-user quantum VLAN](https://doi.org/10.1007/s11433-024-2545-5) on an AlGaAs source, running encrypted voice and images over a campus network, borrowing port-based VLAN grouping from classical networking.

for scale on the key rates: an entanglement-based QKD system was recently [deployed in financial infrastructure](https://arxiv.org/abs/2607.11252) between two data centers, 22 km, 8 dB, and held **63.8 kb/s averaged over four continuous months** at 93.7% uptime. that is three orders of magnitude above this paper's 30 bits/s. it is also two users, one dedicated point-to-point link, and a source optimized for exactly that. the gap between "a link" and "a link that is one of 28 sharing a source" is most of what this whole architecture is trying to close.

## the short version

- a fully connected N-user entanglement network needs N(N−1)/2 conjugate channel pairs, so **users go as the square root of source bandwidth over channel width**. doubling your source buys 40% more users.
- the reachable window here is about 1400 GHz on each side of degeneracy, and that one number explains all three of the paper's networks: 6 links × 200 GHz, 10 × 100, 28 × 50.
- the source is an AlGaAs Bragg reflection waveguide whose real advantage is not brightness. it has low enough group velocity mismatch that **the Bell state needs no off-chip walk-off compensation**.
- 60 nm of entanglement bandwidth, 36 conjugate pairs, 13 of them reachable, because the switch is C-band and degeneracy sits near its top edge. the fix is a temperature knob.
- QBER under 2% and 28 to 39 asymptotic bits/s, flat across every reachable channel. flatness is the selling point against a passive filter tree.
- **asymptotic key survives to 250 km. finite key, with 10 minute blocks, dies at 75.** the gap is where most of the honest engineering lives.
- link asymmetry costs 35 km of asymptotic reach, via accidentals rather than signal, and costs nothing at all in the finite-key regime because block size kills you first.
- 4, 5 and 8 user networks with 7, 9 and 11% spread across links, reconfigured by loading a profile.
- **the reach of the network depends on how many users are on it**: halving channel width to go from 5 users to 8 pulls the finite-key distance from about 77 km to 63.
- leveling an unbalanced network works, and about half of the fix came from spending idle spectrum. the other half cost roughly 40% of the network's total coincidence budget, which nobody prices.
- what's missing: the composed 8-user-over-fiber-with-finite-key experiment, a real allocator instead of "a simple algorithm", propagation time equalization, a clock, and a figure of merit that is η² rather than brightness.

what i keep coming back to is that this paper and [the Optica one]({% post_url 2026-04-04-Flex-grid-for-entanglement %}) each had the half the other lacked, four months apart, with this one citing that one and neither borrowing much. one had a switch worth programming and a source too narrow to program it with. this one had 60 nm of entanglement and stopped at "a simple algorithm". the interesting work of the following five years has been assembling one paper out of the two, and it is mostly done except for the part where somebody turns all 28 links on at once and reports what happens.

{% raw %}<style>
.qw{text-align:left;background:#191919;border:1px solid #333;border-radius:6px;padding:18px 18px 14px;margin:28px 0;font-family:"Open Sans","Helvetica Neue",Helvetica,Arial,sans-serif;font-size:13px;line-height:1.5;color:#e6e6e6;touch-action:manipulation;-webkit-tap-highlight-color:transparent}
.qw *{box-sizing:border-box}
.qw-hd{margin-bottom:14px}
.qw-t{display:block;font-size:14px;font-weight:600;color:#fff;letter-spacing:.01em}
.qw-s{display:block;font-size:12px;color:#8f8f8f;margin-top:3px}
.qw-lg{display:flex;flex-wrap:wrap;gap:6px 16px;margin-top:8px;font-size:11.5px;color:#b4b4b4;align-items:center}
.qw-lg i{display:inline-block;width:18px;height:3px;border-radius:2px;margin-right:6px;vertical-align:middle}
.qw-ctl{display:flex;flex-wrap:wrap;gap:18px;margin-bottom:12px}
.qw-g{display:flex;flex-direction:column;gap:5px}
.qw-l{font-size:11px;text-transform:uppercase;letter-spacing:.08em;color:#8f8f8f}
.qw-gk{text-transform:none;font-size:12px}
.qw-l b{text-transform:none;color:#32c29e;font-weight:600;font-variant-numeric:tabular-nums}
.qw-b{display:inline-flex;flex-wrap:wrap;gap:4px}
.qw-b button{font:inherit;font-size:12px;line-height:1;padding:7px 11px;background:#242424;color:#c8c8c8;border:1px solid #3a3a3a;border-radius:4px;cursor:pointer;transition:background .12s,color .12s,border-color .12s}
.qw-b button:hover{background:#2e2e2e;color:#fff}
.qw-b button.on{background:#32c29e;border-color:#32c29e;color:#10231f;font-weight:600}
.qw-b button:focus-visible{outline:2px solid #32c29e;outline-offset:2px}
.qw-sl{display:flex;flex-wrap:wrap;gap:16px;margin-bottom:12px}
.qw-sr{flex:1 1 220px;display:flex;flex-direction:column;gap:6px;cursor:pointer}
.qw-sr input[type=range]{-webkit-appearance:none;appearance:none;width:100%;height:4px;background:#3a3a3a;border-radius:2px;outline:none;margin:4px 0}
.qw-sr input[type=range]::-webkit-slider-thumb{-webkit-appearance:none;appearance:none;width:15px;height:15px;border-radius:50%;background:#32c29e;cursor:grab;border:0}
.qw-sr input[type=range]::-moz-range-thumb{width:15px;height:15px;border-radius:50%;background:#32c29e;cursor:grab;border:0}
.qw-sr input[type=range]:focus-visible{outline:2px solid #32c29e;outline-offset:4px}
.qw-svg{display:block;width:100%;height:auto;overflow:visible}
.qw-two{display:flex;flex-wrap:wrap;gap:14px}
.qw-pane{flex:1 1 260px;min-width:0;margin-bottom:6px}
.qw-pl{display:block;font-size:11px;text-transform:uppercase;letter-spacing:.08em;color:#8f8f8f;margin-bottom:4px}
.qw-out{display:flex;flex-wrap:wrap;gap:8px;margin-top:12px}
.qw-oi{flex:0 1 auto;min-width:120px;background:#212121;border:1px solid #303030;border-radius:4px;padding:7px 10px}
.qw-grow{flex:1 1 200px}
.qw-ok{display:block;font-size:10px;text-transform:uppercase;letter-spacing:.08em;color:#8f8f8f}
.qw-ok.qw-bs{color:#32c29e}
.qw-ok.qw-fs{color:#f2a03d}
.qw-ov{display:block;font-family:ui-monospace,SFMono-Regular,Menlo,Consolas,monospace;font-size:14px;color:#fff;margin-top:2px;font-variant-numeric:tabular-nums}
.qw-ov sup{font-size:9px}
.qw-ov.qw-txt{font-family:inherit;font-size:13px;font-weight:600}
.qw-n{font-size:12px;color:#9a9a9a;margin:11px 0 0;line-height:1.55}
.qw-n b{color:#cfcfcf;font-weight:600}
@media(max-width:600px){.qw{padding:14px 12px 12px}.qw-ctl{gap:12px}}
</style>
<script>
(function(){
  var NS='http://www.w3.org/2000/svg';
  function el(t,a,txt){
    var n=document.createElementNS(NS,t);
    for(var k in a) n.setAttribute(k,a[k]);
    if(txt!==undefined) n.appendChild(document.createTextNode(txt));
    return n;
  }
  function clear(g){ while(g.firstChild) g.removeChild(g.firstChild); }
  function btnGroup(root,grp,cb){
    var bs=root.querySelectorAll('[data-grp="'+grp+'"] button');
    Array.prototype.forEach.call(bs,function(b){
      b.addEventListener('click',function(){
        Array.prototype.forEach.call(bs,function(o){ o.classList.remove('on'); o.removeAttribute('aria-pressed'); });
        b.classList.add('on'); b.setAttribute('aria-pressed','true');
        cb(b.getAttribute('data-v'));
      });
    });
  }
  function offGroup(root,grp){
    Array.prototype.forEach.call(root.querySelectorAll('[data-grp="'+grp+'"] button'),function(b){
      b.classList.remove('on'); b.removeAttribute('aria-pressed');
    });
  }
  function fmtR(r){
    if(r<=0) return 'no key';
    if(r>=10) return r.toFixed(0)+' b/s';
    if(r>=1) return r.toFixed(1)+' b/s';
    if(r>=0.01) return r.toFixed(2)+' b/s';
    return r.toExponential(1)+' b/s';
  }

  /* ---------- shared BBM92 model ----------
     Calibrated to the four landmarks the paper states: 28-39 bits/s asymptotic
     at 0 km on a 100 GHz pair, QBER under 2% out to 50 km, asymptotic key to
     250 km symmetric and 215 km asymmetric, finite key (10 min blocks,
     eps = 1e-10) to 75 km.                                                 */
  var ALPHA=0.22, ACOEF=2.74e-5, RDARK=0.17, EOPT=0.010, FEC=1.16, R0=70, EPSL=23.72, OVER=99;
  function log2(x){ return Math.log(x)/Math.LN2; }
  function H2(x){
    if(x<=0) return 0; if(x>=1) return 1;
    return -x*log2(x)-(1-x)*log2(1-x);
  }
  function trans(L){ return Math.pow(10,-ALPHA*L/10); }
  function accid(L,sym){
    var t=trans(L);
    return sym ? ACOEF*Math.pow(Math.sqrt(t)+RDARK,2)/t
               : ACOEF*(1+RDARK)*(t+RDARK)/t;
  }
  function qber(L,sym){ var a=accid(L,sym); return (EOPT+0.5*a)/(1+a); }
  function rawRate(L,w,sym){ return R0*(w/100)*trans(L)*(1+accid(L,sym)); }
  function keyAsym(L,w,sym){
    var e=qber(L,sym);
    return Math.max(0, rawRate(L,w,sym)*0.5*(1-FEC*H2(e)-H2(e)));
  }
  function keyFinite(L,w,sym,tsec){
    var n=rawRate(L,w,sym)*tsec*0.5;
    if(n<=1) return 0;
    var e=qber(L,sym), mu=Math.sqrt(EPSL/(2*n));
    return Math.max(0,(n*(1-H2(Math.min(0.5,e+mu))-FEC*H2(e))-OVER)/tsec);
  }
  function reach(w,sym,tsec){
    if(keyFinite(0,w,sym,tsec)<=0) return 0;
    var lo=0, hi=400;
    for(var i=0;i<50;i++){
      var m=(lo+hi)/2;
      if(keyFinite(m,w,sym,tsec)>0) lo=m; else hi=m;
    }
    return lo;
  }

  /* ---------- 1. how many users fit in the window ---------- */
  (function(){
    var H=document.getElementById('qw-fit');
    if(!H) return;
    var WMIN=12.5, WMAX=400, band=1400;
    var sw=H.querySelector('#qwf-w'), gS=H.querySelector('#qwf-spec'), gG=H.querySelector('#qwf-graph');
    function p2w(p){ return WMIN*Math.pow(WMAX/WMIN,p/100); }
    function w2p(w){ return 100*Math.log(w/WMIN)/Math.log(WMAX/WMIN); }
    function users(w,B){
      var n=2;
      while((n+1)*n/2*w<=B && n<60) n++;
      return n;
    }
    function spec(w,B,links){
      clear(gS);
      var X0=10,X1=310,Y0=16,Y1=118;
      var used=Math.min(links*w,B), frac=used/B;
      gS.appendChild(el('rect',{x:X0,y:Y0,width:X1-X0,height:Y1-Y0,fill:'#232323',stroke:'#3a3a3a','stroke-width':1,rx:3}));
      var wpx=(X1-X0)*(w/B);
      if(links<=160){
        for(var i=0;i<links;i++){
          var x=X0+i*wpx;
          if(x+wpx>X1+0.5) break;
          gS.appendChild(el('rect',{x:x+(wpx>2.5?0.6:0.15),y:Y0+1,width:Math.max(wpx-(wpx>2.5?1.2:0.3),0.5),height:Y1-Y0-2,
            fill:'#32c29e','fill-opacity':0.62,stroke:'#32c29e','stroke-width':wpx>3?0.8:0,rx:wpx>4?1.5:0}));
        }
      } else {
        gS.appendChild(el('rect',{x:X0+1,y:Y0+1,width:(X1-X0-2)*frac,height:Y1-Y0-2,fill:'#32c29e','fill-opacity':0.5}));
        gS.appendChild(el('text',{x:X0+(X1-X0)*frac/2,y:(Y0+Y1)/2+4,'text-anchor':'middle',fill:'#0e2620','font-size':11,'font-weight':600},links+' slices'));
      }
      if(frac<0.995){
        var xi=X0+(X1-X0)*frac;
        gS.appendChild(el('text',{x:Math.min(xi+6,X1-4),y:(Y0+Y1)/2+4,fill:'#7d7d7d','font-size':10},'idle'));
      }
      gS.appendChild(el('text',{x:X0,y:Y1+16,fill:'#8f8f8f','font-size':10},'degeneracy'));
      gS.appendChild(el('text',{x:X1,y:Y1+16,'text-anchor':'end',fill:'#8f8f8f','font-size':10},(B/1000).toFixed(1)+' THz out'));
      gS.appendChild(el('text',{x:(X0+X1)/2,y:Y1+32,'text-anchor':'middle',fill:'#8f8f8f','font-size':10},
        links+' × '+(w<100?w.toFixed(1):w.toFixed(0))+' GHz = '+Math.round(links*w)+' GHz'));
    }
    function graph(n){
      clear(gG);
      var cx=160, cy=66, R=52;
      var pts=[];
      for(var i=0;i<n;i++){
        var a=-Math.PI/2+2*Math.PI*i/n;
        pts.push([cx+R*Math.cos(a),cy+R*Math.sin(a)]);
      }
      var op=n<=6?0.7:(n<=10?0.45:(n<=16?0.26:0.14));
      var sw2=n<=8?1.3:(n<=16?0.9:0.6);
      for(var i=0;i<n;i++) for(var j=i+1;j<n;j++)
        gG.appendChild(el('line',{x1:pts[i][0],y1:pts[i][1],x2:pts[j][0],y2:pts[j][1],
          stroke:'#32c29e','stroke-opacity':op,'stroke-width':sw2}));
      var r=n<=8?5:(n<=16?3.6:2.6);
      for(var i=0;i<n;i++)
        gG.appendChild(el('circle',{cx:pts[i][0],cy:pts[i][1],r:r,fill:'#191919',stroke:'#fff','stroke-width':n<=16?1.6:1.1}));
      gG.appendChild(el('text',{x:cx,y:cy+4,'text-anchor':'middle',fill:'#8f8f8f','font-size':13,'font-weight':600},n+' users'));
      gG.appendChild(el('text',{x:cx,y:138,'text-anchor':'middle',fill:'#8f8f8f','font-size':10},
        n*(n-1)/2+' two-user links, every pair connected'));
    }
    function upd(){
      var w=p2w(+sw.value), n=users(w,band), links=n*(n-1)/2;
      var r0=keyAsym(0,w,true), d=reach(w,true,600);
      H.querySelector('#qwf-wv').textContent=(w<100?w.toFixed(1):w.toFixed(0))+' GHz';
      H.querySelector('#qwf-n').textContent=n;
      H.querySelector('#qwf-e').textContent=links;
      H.querySelector('#qwf-r').textContent=fmtR(r0);
      H.querySelector('#qwf-d').textContent=d.toFixed(0)+' km';
      spec(w,band,links); graph(n);
      var note;
      if(Math.abs(w-200)<3&&band===1400) note='This is the paper&rsquo;s <b>4-user network</b>. Six links, 200 GHz each, 1200 of the 1400 GHz spent.';
      else if(Math.abs(w-100)<2&&band===1400) note='This is the paper&rsquo;s <b>5-user network</b>. Ten links at 100 GHz, and 400 GHz left idle, which turns out to matter in the leveling experiment further down.';
      else if(Math.abs(w-50)<1.5&&band===1400) note='This is the paper&rsquo;s <b>8-user network</b>, and it fits exactly: 28 links × 50 GHz = 1400 GHz, nothing left over.';
      else note='At <b>'+(w<100?w.toFixed(1):w.toFixed(0))+' GHz</b> the window supports <b>'+n+' users</b> and '+links+' links.';
      note+=' Each link gets <b>'+fmtR(r0)+'</b> of asymptotic key at zero distance, and its finite-key reach with 10 minute blocks is <b>'+d.toFixed(0)+' km</b>. '
        +(band===1400
          ? 'Only 1400 GHz on each side of degeneracy is reachable, because the switch is a C-band part and degeneracy sits near its upper edge.'
          : 'This assumes the whole 60 nm the chip emits is reachable, which needs a C+L switch and degeneracy moved to the middle of the band.');
      H.querySelector('#qwf-note').innerHTML=note;
    }
    btnGroup(H,'ps',function(v){ sw.value=w2p(+v); upd(); });
    btnGroup(H,'bd',function(v){ band=+v; upd(); });
    sw.addEventListener('input',function(){ offGroup(H,'ps'); upd(); });
    upd();
  })();

  /* ---------- 2. where the key rate dies ---------- */
  (function(){
    var H=document.getElementById('qw-dist');
    if(!H) return;
    var sym=true, w=100;
    var sl=H.querySelector('#qwd-l'), st=H.querySelector('#qwd-t');
    var gG=H.querySelector('#qwd-grid'), gC=H.querySelector('#qwd-curves'), gM=H.querySelector('#qwd-mark');
    var X0=52,X1=590,Y0=14,Y1=210,LMAX=280,DEC=5;
    function px(L){ return X0+L/LMAX*(X1-X0); }
    function py(r){ return Y1-Math.max(0,Math.min(1,(log2(r)/log2(10)+3)/DEC))*(Y1-Y0); }
    function qy(q){ return Y1-Math.min(q/0.15,1)*(Y1-Y0); }
    function grid(){
      clear(gG);
      [100,10,1,0.1,0.01,0.001].forEach(function(v){
        gG.appendChild(el('line',{x1:X0,y1:py(v),x2:X1,y2:py(v),stroke:'#2b2b2b','stroke-width':1}));
        gG.appendChild(el('text',{x:X0-8,y:py(v)+4,'text-anchor':'end',fill:'#7d7d7d','font-size':10},
          v>=1?String(v):(v===0.1?'0.1':(v===0.01?'0.01':'0.001'))));
      });
      [0,5,10,15].forEach(function(q){
        gG.appendChild(el('text',{x:X1+8,y:qy(q/100)+4,fill:'#6b8fc4','font-size':10},q+'%'));
      });
      [0,50,100,150,200,250].forEach(function(L){
        gG.appendChild(el('text',{x:px(L),y:Y1+18,'text-anchor':'middle',fill:'#7d7d7d','font-size':10},L));
      });
      gG.appendChild(el('text',{x:(X0+X1)/2,y:Y1+38,'text-anchor':'middle',fill:'#8f8f8f','font-size':11},'fiber between the two users, km'));
      gG.appendChild(el('text',{x:X0-10,y:Y0-6,'text-anchor':'end',fill:'#8f8f8f','font-size':10},'bits/s'));
      gG.appendChild(el('text',{x:X1+8,y:Y0-6,fill:'#6b8fc4','font-size':10},'QBER'));
      gG.appendChild(el('line',{x1:X0,y1:qy(0.11),x2:X1,y2:qy(0.11),stroke:'#4a3a2a','stroke-width':1,'stroke-dasharray':'4 4'}));
      gG.appendChild(el('text',{x:X0+290,y:qy(0.11)-7,fill:'#8a7a5a','font-size':10},'11% QBER, no key above this'));
    }
    function curves(){
      clear(gC);
      var t=+st.value*60, da='', df='', dq='', started=false;
      for(var L=0;L<=LMAX;L+=1){
        dq+=(L?'L':'M')+px(L).toFixed(1)+' '+qy(qber(L,sym)).toFixed(1);
        var ra=keyAsym(L,w,sym);
        if(ra>0.001) da+=(started?'L':'M')+px(L).toFixed(1)+' '+py(ra).toFixed(1), started=true;
      }
      var st2=false;
      for(var L=0;L<=LMAX;L+=0.5){
        var rf=keyFinite(L,w,sym,t);
        if(rf>0.001) df+=(st2?'L':'M')+px(L).toFixed(1)+' '+py(rf).toFixed(1), st2=true;
        else if(st2) break;
      }
      gC.appendChild(el('path',{d:dq,fill:'none',stroke:'#6ba3f0','stroke-width':1.6,'stroke-opacity':0.75}));
      gC.appendChild(el('path',{d:da,fill:'none',stroke:'#32c29e','stroke-width':2.3,'stroke-linecap':'round'}));
      gC.appendChild(el('path',{d:df,fill:'none',stroke:'#f2a03d','stroke-width':2.3,'stroke-linecap':'round'}));
      [[reach(w,sym,t),'#f2a03d','finite key ends'],[reachA(),'#32c29e','asymptotic ends']].forEach(function(c,i){
        if(c[0]<=0||c[0]>LMAX) return;
        gC.appendChild(el('line',{x1:px(c[0]),y1:Y0+(i?26:6),x2:px(c[0]),y2:Y1,stroke:c[1],'stroke-width':1,'stroke-opacity':0.5,'stroke-dasharray':'3 3'}));
        gC.appendChild(el('text',{x:px(c[0])+(px(c[0])>X1-120?-5:5),y:Y0+(i?24:4)+8,'text-anchor':px(c[0])>X1-120?'end':'start',fill:c[1],'font-size':9.5,'fill-opacity':0.85},c[2]+', '+c[0].toFixed(0)+' km'));
      });
    }
    function upd(){
      var L=+sl.value, t=+st.value*60;
      var ra=keyAsym(L,w,sym), rf=keyFinite(L,w,sym,t), q=qber(L,sym);
      H.querySelector('#qwd-lv').textContent=L+' km';
      H.querySelector('#qwd-tv').textContent=st.value+' min';
      H.querySelector('#qwd-ra').textContent=fmtR(ra);
      H.querySelector('#qwd-rf').textContent=fmtR(rf);
      H.querySelector('#qwd-q').textContent=(q*100).toFixed(2)+'%';
      H.querySelector('#qwd-b').textContent=rf>0?Math.round(rf*t).toLocaleString():'0';
      curves();
      clear(gM);
      gM.appendChild(el('line',{x1:px(L),y1:Y0,x2:px(L),y2:Y1,stroke:'#5a5a5a','stroke-width':1}));
      if(ra>0.001) gM.appendChild(el('circle',{cx:px(L),cy:py(ra),r:4.2,fill:'#fff',stroke:'#32c29e','stroke-width':2}));
      if(rf>0.001) gM.appendChild(el('circle',{cx:px(L),cy:py(rf),r:4.2,fill:'#fff',stroke:'#f2a03d','stroke-width':2}));
      var vd, vc;
      if(rf<=0&&ra<=0){ vd='no key at all, at any block size'; vc='#e2603f'; }
      else if(rf<=0){ vd='a key exists in theory and not in practice'; vc='#e2603f'; }
      else if(rf*t<1000){ vd='a trickle, not a channel'; vc='#f2a03d'; }
      else if(rf*t<20000){ vd='enough to re-key something on a schedule'; vc='#d8c257'; }
      else { vd='a working link'; vc='#32c29e'; }
      var ve=H.querySelector('#qwd-v'); ve.textContent=vd; ve.style.color=vc;
      var d=reach(w,sym,t);
      H.querySelector('#qwd-note').innerHTML=
        'On <b>'+w+' GHz</b> channels '+(sym?'split evenly between the two users':'with all the fiber on one side')
        +', the asymptotic rate stays positive to <b>'+reachA().toFixed(0)+' km</b> but the finite-key rate with <b>'
        +st.value+' minute</b> blocks dies at <b>'+d.toFixed(0)+' km</b>. '
        +'At the marker you get <b>'+(rf>0?Math.round(rf*t).toLocaleString()+' secret bits':'nothing')+'</b> per block. '
        +'Longer blocks always help, because the finite-size penalty falls as 1/&radic;n, but they also mean waiting that long before you have a usable key.';
    }
    function reachA(){
      if(keyAsym(0,w,sym)<=0) return 0;
      var lo=0,hi=400;
      for(var i=0;i<50;i++){ var m=(lo+hi)/2; if(keyAsym(m,w,sym)>0) lo=m; else hi=m; }
      return lo;
    }
    btnGroup(H,'sy',function(v){ sym=(v==='1'); upd(); });
    btnGroup(H,'ps',function(v){ w=+v; upd(); });
    sl.addEventListener('input',upd);
    st.addEventListener('input',upd);
    grid(); upd();
  })();

  /* ---------- 3. leveling an unbalanced network ---------- */
  (function(){
    var H=document.getElementById('qw-lvl');
    if(!H) return;
    var KG=33.5, TB=0.209, BUD=1400;
    var LOCAL=['AC','AD','AE','CD','CE','DE'], FAR=['AB','BC','BD','BE'];
    var sb=H.querySelector('#qwv-b'), so=H.querySelector('#qwv-o');
    var gB=H.querySelector('#qwv-bar'), gR=H.querySelector('#qwv-bars');
    var PRE={ a:[100,100], b:[267.5,55], c:[200,100], d:[25,215] };
    function budgetBar(wb,wo){
      clear(gB);
      var X0=6,X1=634,Y=10,Hh=22;
      var far=4*wb, loc=6*wo, tot=far+loc;
      var sc=(X1-X0)/Math.max(BUD,tot);
      gB.appendChild(el('rect',{x:X0,y:Y,width:X1-X0,height:Hh,fill:'#232323',stroke:'#3a3a3a','stroke-width':1,rx:3}));
      gB.appendChild(el('rect',{x:X0,y:Y,width:far*sc,height:Hh,fill:'#e2603f','fill-opacity':0.7,rx:3}));
      gB.appendChild(el('rect',{x:X0+far*sc,y:Y,width:loc*sc,height:Hh,fill:'#32c29e','fill-opacity':0.7}));
      if(tot>BUD+1){
        gB.appendChild(el('rect',{x:X0+BUD*sc,y:Y-3,width:(tot-BUD)*sc,height:Hh+6,fill:'none',stroke:'#e2603f','stroke-width':1.5,'stroke-dasharray':'3 3'}));
        gB.appendChild(el('text',{x:X0+BUD*sc+4,y:Y+Hh+16,fill:'#e2603f','font-size':10},'over budget by '+Math.round(tot-BUD)+' GHz'));
      } else if(tot<BUD-1){
        gB.appendChild(el('text',{x:X0+tot*sc+6,y:Y+15,fill:'#7d7d7d','font-size':10},Math.round(BUD-tot)+' GHz idle'));
      }
      gB.appendChild(el('text',{x:X0+4,y:Y+15,fill:'#2a0f08','font-size':10,'font-weight':600},far>150?'to B':''));
      gB.appendChild(el('text',{x:X0+far*sc+4,y:Y+15,fill:'#0e2620','font-size':10,'font-weight':600},loc>150?'local links':''));
    }
    function bars(rl,rf){
      clear(gR);
      var X0=42,X1=630,YB=150,YT=16, all=LOCAL.concat(FAR);
      var vals=LOCAL.map(function(){return rl;}).concat(FAR.map(function(){return rf;}));
      var hi=7500;
      var bw=(X1-X0)/all.length;
      [0,0.25,0.5,0.75,1].forEach(function(f){
        var y=YB-f*(YB-YT);
        gR.appendChild(el('line',{x1:X0,y1:y,x2:X1,y2:y,stroke:f?'#282828':'#3d3d3d','stroke-width':1}));
        gR.appendChild(el('text',{x:X0-6,y:y+4,'text-anchor':'end',fill:'#7d7d7d','font-size':9},Math.round(f*hi).toLocaleString()));
      });
      all.forEach(function(L,i){
        var v=vals[i], far=FAR.indexOf(L)>=0, col=far?'#e2603f':'#32c29e';
        var x=X0+i*bw+bw*0.16, wd=bw*0.68, hh=Math.max(2,Math.min(1,v/hi)*(YB-YT));
        gR.appendChild(el('rect',{x:x,y:YB-hh,width:wd,height:hh,fill:col,'fill-opacity':0.78,stroke:col,'stroke-width':1,rx:2}));
        gR.appendChild(el('text',{x:x+wd/2,y:YB-hh-5,'text-anchor':'middle',fill:'#c8c8c8','font-size':9.5},Math.round(v)));
        gR.appendChild(el('text',{x:x+wd/2,y:YB+16,'text-anchor':'middle',fill:col,'font-size':11,'font-weight':600},L));
      });
    }
    function upd(){
      var wb=+sb.value, wo=+so.value;
      var rf=KG*TB*wb, rl=KG*wo;
      var mn=Math.min(rl,rf), mx=Math.max(rl,rf), tot=6*rl+4*rf;
      H.querySelector('#qwv-bv').textContent=wb.toFixed(0)+' GHz';
      H.querySelector('#qwv-ov').textContent=wo.toFixed(0)+' GHz';
      H.querySelector('#qwv-min').textContent=Math.round(mn).toLocaleString();
      H.querySelector('#qwv-r').textContent=(mx/Math.max(mn,1)).toFixed(1)+'x';
      H.querySelector('#qwv-tot').textContent=Math.round(tot).toLocaleString();
      budgetBar(wb,wo); bars(rl,rf);
      var ratio=mx/Math.max(mn,1), vd, vc;
      if(4*wb+6*wo>BUD+1){ vd='this allocation does not fit the band'; vc='#e2603f'; }
      else if(ratio<1.25){ vd='flat, and it cost you'; vc='#32c29e'; }
      else if(ratio<2.5){ vd='usable spread'; vc='#d8c257'; }
      else if(ratio<8){ vd='the distant user is a second-class citizen'; vc='#f2a03d'; }
      else { vd='B is barely on the network'; vc='#e2603f'; }
      var ve=H.querySelector('#qwv-v'); ve.textContent=vd; ve.style.color=vc;
      var evenTot=6*KG*(BUD/10)+4*KG*TB*(BUD/10);
      var cost=(1-tot/evenTot)*100;
      var lead='B&rsquo;s four links have to survive 25 km first, so a GHz spent on them is worth about <b>a fifth</b> of a GHz spent locally. ';
      H.querySelector('#qwv-note').innerHTML= (4*wb+6*wo>BUD+1)
        ? lead+'This allocation asks for <b>'+Math.round(4*wb+6*wo)+' GHz</b> and only 1400 is reachable, so it is not a network you could actually configure. Take width off somebody.'
        : lead+'That exchange rate is the whole story: the network total here is <b>'+Math.round(tot).toLocaleString()+'</b> coincidences per 30 s, '
          +(cost>1?'which is <b>'+cost.toFixed(0)+'% below</b>':'which is <b>'+Math.abs(cost).toFixed(0)+'% above</b>')
          +' what the same 1400 GHz would deliver split evenly across all ten links. '
          +'Flattening the network is not free, and nothing in the paper puts a number on it.';
    }
    btnGroup(H,'ps',function(v){ sb.value=PRE[v][0]; so.value=PRE[v][1]; upd(); });
    sb.addEventListener('input',function(){ offGroup(H,'ps'); upd(); });
    so.addEventListener('input',function(){ offGroup(H,'ps'); upd(); });
    upd();
  })();
})();
</script>{% endraw %}