---
layout: post
section-type: post
title: "Flex grid for entanglement : one switch instead of a tree of filters"
category: 'networking'
tags: [ 'networking', 'fiber', 'quantum' ]
---

classical network operators have a word for deciding who gets how much capacity: provisioning. you pick the split, you push a config, the box does it, and nobody drives to the site with a screwdriver.

quantum networks have exactly the same need and, for a long time, none of the tooling. the entangled photons got divided up at install time by the physical layout of a tree of optical filters. want a different split? go into the rack.

this paper is the one that fixed that. [Lingaraju et al., "Adaptive bandwidth management for entanglement distribution in quantum networks," *Optica* 8(3), 329 (2021)](https://doi.org/10.1364/OPTICA.413657). four users, one entangled photon source, one wavelength selective switch, and two-party rates that start out a factor of 4200 apart and end up within a factor of two. it is four pages long and most of it is a networking paper wearing a quantum hat.

below: what the paper did, what the following five years did with it, and what i think is still missing.

## the setup, for people who don't do quantum

### one source, many users

the architecture here is called entanglement distribution by a central provider. one box in the middle makes entangled photon pairs, and every user gets a fiber to that box. no user needs a source, a laser, or a crystal. they need a fiber and a detector.

the source in this paper is spontaneous parametric down-conversion: a pump photon goes into a nonlinear crystal and occasionally splits into two lower-energy photons. the pair comes out entangled in polarization, and, importantly, over a wide band of wavelengths at once. that band is the resource being divided.

### wavelength is the address

this is the part that makes the whole scheme work, and it is not obvious.

the split obeys energy conservation. the two daughter photons always add up to the pump:

```
    ω_signal  +  ω_idler  =  ω_pump
```

so if you hand Alice a slice of spectrum 100 GHz above the center of the band, her photons' partners are all sitting 100 GHz below the center. give that lower slice to Bob and Alice and Bob now share entanglement. give it to Carol instead and Alice and Bob share nothing at all, even though both are connected to the same box by the same kind of fiber.

**you do not route entanglement by choosing a destination. you route it by handing two people the two halves of the same energy budget.** the switch in the middle is not deciding where photons go so much as deciding which pairs of people are energy-matched. that is why this is a spectrum allocation problem rather than a switching problem, and why the entire vocabulary of flexible-grid WDM turns out to apply.

### what you actually measure

each user has a polarization analyzer and a single photon detector. a detector clicking on its own tells you almost nothing, because dark counts and stray light click too. what matters is a **coincidence**: Alice clicks and Bob clicks inside the same short window, so those two probably came from the same pair.

the coincidence rate on a link is the closest thing this network has to bandwidth. it is what the paper uses as its service metric, and, as we will get to, that choice is one of the paper's own stated weaknesses.

## the problem with a tree of filters

the state of the art before this paper was passive. [Wengerowsky et al. (Nature 564, 225, 2018)](https://doi.org/10.1038/s41586-018-0766-y) built a fully and simultaneously connected four-user network out of nothing but DWDM filters, which is a lovely result: no active components, no power, nothing to configure. [Joshi et al. (Sci. Adv. 6, eaba0959, 2020)](https://doi.org/10.1126/sciadv.aba0959) took it to eight users by having pairs of users share a slice through a 50:50 splitter, at the cost of a hard 3 dB.

the trouble is what happens to the filter tree as N grows.

full connectivity needs `2N² − 3N` filters. worse than the count is the path length. a photon headed for the lucky channel passes through two filters. the photon headed for the worst channel gets bounced off, at minimum, `N² − N` filter ports before it is finally transmitted. at the paper's assumed 0.25 dB per reflection and 0.6 dB per transmission, that is 3.6 dB at four users and 60.6 dB at sixteen.

the filter count and the worst case are both bad. the **spread** between channels is worse. one link on your network is fine and another is 56 dB down. you cannot engineer a service around that, because every link needs its own integration time, its own noise budget, and its own expectations.

<div class="qw" id="qw-loss" data-qw><div class="qw-hd"><span class="qw-t">worst-case channel loss as the network grows</span><span class="qw-s">the paper's own comparison, Fig. 1 inset. 0.25 dB per DWDM reflection, 0.6 dB for the final transmission, 4.5 dB for the switch.</span><span class="qw-lg"><i style="background:#f2a03d"></i>DWDM filter tree, worst channel<i style="background:#32c29e"></i>wavelength selective switch, every channel</span></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">users on the network <span class="qw-gk">N</span> <b id="qwl-nv">16</b></span><input type="range" id="qwl-n" min="2" max="20" step="1" value="16"></label></div><svg class="qw-svg" viewBox="0 0 640 300" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Worst-case channel loss versus number of users"><g id="qwl-grid"></g><g id="qwl-curves"></g><g id="qwl-mark"></g></svg><div class="qw-out"><div class="qw-oi"><span class="qw-ok">filters needed</span><span class="qw-ov" id="qwl-cnt">&mdash;</span></div><div class="qw-oi"><span class="qw-ok qw-fs">tree, worst channel</span><span class="qw-ov" id="qwl-dw">&mdash;</span></div><div class="qw-oi"><span class="qw-ok qw-bs">switch, any channel</span><span class="qw-ov" id="qwl-ws">4.5 dB</span></div><div class="qw-oi qw-grow"><span class="qw-ok">photons that survive the worst path</span><span class="qw-ov qw-txt" id="qwl-sv">&mdash;</span></div></div><p class="qw-n" id="qwl-note"></p><noscript><p class="qw-n">A fully connected DWDM tree needs 2N&sup2;&nbsp;&minus;&nbsp;3N filters, and its worst channel takes N&sup2;&nbsp;&minus;&nbsp;N reflections before one transmission. That is 3.6 dB at 4 users, 14.6 dB at 8, 60.6 dB at 16 and 95.6 dB at 20, against a flat 4.5 dB for the switch at any size. The two approaches cross at about 4 or 5 users.</p></noscript></div>

the crossover sits between four and five users. below that the passive tree really is better, which the authors say plainly. above it, the tree stops being a design and starts being a liability.

## the switch

a wavelength selective switch is a stock ROADM part. light comes in one fiber, a grating spreads it across a liquid crystal spatial light modulator, and each column of pixels steers its little bit of spectrum to whichever output fiber you tell it to. it is how carriers add and drop wavelengths without touching hardware, and it is the reason "flexible grid" exists as a concept: the channels do not have to be a fixed 50 GHz or 100 GHz wide any more, they can be whatever width the job needs.

the paper uses a Finisar WaveShaper 4000S/X: one input, four outputs, C and L band, about 20 GHz of spectral resolution and 4 GHz of addressability. three properties matter here.

**loss does not grow with N.** roughly 4.5 dB, on every connection, whether the device is feeding two users or twenty. this is the headline claim, and it is the one the flat green line in the chart above is making.

**it reconfigures electronically.** a new allocation is a new profile loaded over USB, not a new patch layout.

**slice widths are free above the resolution floor.** you can give one link 12 GHz and another 200 GHz in the same configuration.

that is the whole idea: replace a hard-wired passive tree with one active component whose cost is fixed and whose configuration is software.

## the testbed

the source is a 10 mm type-II PPLN ridge waveguide pumped by about 24 mW of continuous wave light at 780 nm, tuned for degenerate down-conversion so the pair spectrum is centered in the C band. the biphoton is **2.5 nm wide, about 310 GHz**. temporal walk-off between the two polarizations is undone with a 90 degree splice of polarization-maintaining fiber.

that 310 GHz gets carved into **24 slices of 24 GHz each**, plus a stopband in the middle. 24 GHz is chosen because it is a multiple of the switch's 4 GHz addressability while comfortably clearing its 20 GHz resolution. an energy-matched pair of slices is one channel, so there are 12 channels.

then the interesting bit. the four users do not get equivalent equipment:

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>User</th><th>Detector</th><th>Efficiency</th><th>Singles rate as a 1:4 splitter</th></tr>
</thead>
<tbody>
<tr><td>Alice</td><td>SNSPD, free running</td><td>~0.85</td><td>2.6 &times; 10<sup>5</sup> s<sup>&minus;1</sup></td></tr>
<tr><td>Bob</td><td>SNSPD, free running</td><td>~0.85</td><td>3.3 &times; 10<sup>5</sup> s<sup>&minus;1</sup></td></tr>
<tr><td>Carol</td><td>InGaAs APD, gated 20 MHz at 10% duty</td><td>~0.2</td><td>5.5 &times; 10<sup>3</sup> s<sup>&minus;1</sup></td></tr>
<tr><td>Dave</td><td>InGaAs APD, gated 20 MHz at 10% duty</td><td>~0.1</td><td>3.3 &times; 10<sup>3</sup> s<sup>&minus;1</sup></td></tr>
</tbody>
</table>
</div>

two good detectors, two bad ones, and a factor of about 80 in singles rate between the best and worst user before you have allocated anything. this is not sloppiness, it is the experiment. a real network will have heterogeneous nodes, and the question is whether the provider can do anything about it from the middle.

before allocating, they check the entanglement is actually there. polarization correlations in two mutually unbiased bases, run through Bayesian mean estimation rather than a linear inversion, with no accidental subtraction. **channels 1 through 11 come out above 95% fidelity to the target Bell state.** channel 12, way out in the wing of the spectrum where there is barely any flux, is worse.

## three ways to divide the same spectrum

now the actual result. six two-party links exist on a four-user network: AB, AC, AD, BC, BD, CD. the paper allocates the same 310 GHz three different ways.

<div class="qw" id="qw-alloc" data-qw><div class="qw-hd"><span class="qw-t">allocate the biphoton spectrum yourself</span><span class="qw-s">pick a link, then click channels to give them to it. clicking a channel that already belongs to the selected link releases it.</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">the paper's three allocations</span><span class="qw-b" data-grp="ps"><button type="button" data-v="alpha" class="on" aria-pressed="true">alphabetical, 48 GHz</button><button type="button" data-v="rebal">rebalanced, 48 GHz</button><button type="button" data-v="flex">full flex, 24 GHz</button></span></div></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">assigning to</span><span class="qw-b qw-links" data-grp="lk"><button type="button" data-v="AB" class="on" aria-pressed="true">AB</button><button type="button" data-v="AC">AC</button><button type="button" data-v="AD">AD</button><button type="button" data-v="BC">BC</button><button type="button" data-v="BD">BD</button><button type="button" data-v="CD">CD</button></span></div></div><div class="qw-pane"><span class="qw-pl">the biphoton spectrum, 24 slices of 24 GHz, energy-matched about the center</span><svg class="qw-svg" viewBox="0 0 640 190" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Biphoton spectrum divided into channels coloured by owning link"><g id="qwa-spec"></g></svg></div><div class="qw-ch" id="qwa-chans"></div><div class="qw-pane"><span class="qw-pl">each linkresulting two-party coincidence rate, relative, log scalersquo;s coincidence rate, relative to the best link on the network</span><svg class="qw-svg" viewBox="0 0 640 190" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Coincidence rate per link relative to the best link"><g id="qwa-bars"></g></svg></div><div class="qw-out"><div class="qw-oi"><span class="qw-ok">links carrying entanglement</span><span class="qw-ov" id="qwa-live">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">imbalance, best over worst</span><span class="qw-ov" id="qwa-ratio">&mdash;</span></div><div class="qw-oi qw-grow"><span class="qw-ok">verdict</span><span class="qw-ov qw-txt" id="qwa-v">&mdash;</span></div></div><p class="qw-n" id="qwa-note"></p><noscript><p class="qw-n">Allocating the 12 channels alphabetically on a fixed 48 GHz grid leaves links AB and CD a factor of ~4200 apart. Keeping the same grid but sending the brightest slices to the weakest links closes that to ~26. Allocating all 24 GHz slices freely brings five links within a factor of 2, at the cost of dropping link CD entirely.</p></noscript></div>

**(a) fixed 48 GHz grid, allocated alphabetically.** pair up adjacent slices, hand them out in order: AB gets the two brightest, CD gets the two faintest. AB is two good detectors on the brightest part of the spectrum, CD is two bad detectors on the dimmest. the measured ratio between them is about **4200**.

this configuration is not stupid, though. it is a different policy. read the other way round, it is a premium tier: AB gets the best service the network can give. flexible allocation is as useful for deliberately favoring a link as it is for leveling them.

**(b) same fixed grid, allocated to balance.** keep the 48 GHz channels, but send the brightest ones to the weakest links. that alone takes the imbalance from 4200 to about **26**.

that is the cheap win and it deserves emphasis. **two orders of magnitude came from not allocating alphabetically.** no new hardware, no finer grid, just a sensible ordering.

**(c) full flex, 24 GHz slices, no grid at all.** now every slice is allocated independently. five links land **within a factor of two** of each other.

there are two details in here that are easy to skim past. the first is that balancing required using the *wings* of the spectrum, not just the flat bit in the middle, which earlier work had stuck to. the second is that interleaving matters: links with comparable detection efficiency get channels alternated with each other, so they see the same mean flux rather than one getting a bright block and the other a dim one.

## the part they don't hide

link CD got dropped.

carol and dave have the two worst detectors, and their link pays both penalties at once. the paper works out that even in the most favorable allocation, giving CD the seven highest-flux channels, it still could not reach the rates of the other links. so they stopped trying and equalized across the five remaining links instead.

that is the honest boundary of the whole technique. **a switch in the middle redistributes entanglement, it does not create any.** if the total pair flux times the product of two users' efficiencies is below what you need, no allocation saves you. flex grid gives you a budget to spend, and that is all.

the second caveat they flag is subtler. the fidelity numbers were measured channel by channel between Alice and Bob. the paper says outright that this does not guarantee the fidelity of arbitrary *groupings* of those channels, because crosstalk, multipair emission and frequency-dependent birefringence all show up when you combine slices. any allocation you actually intend to run has to be tested where it will run. that sentence is doing a lot of quiet work, and five years later it is still the reason nobody can just solve the allocation on paper and walk away.

## how far this scales

the scaling argument at the end is the part that made this paper matter.

a commercial WSS available at the time had 20 output ports, 4.8 THz of bandwidth and 6.25 GHz of resolution. carve that into 12.5 GHz slices and you get enough channels for **all N(N−1) connections of a fully connected 20-user network**, through one device, at one fixed loss.

the nested DWDM equivalent would need **740 filters**, and its worst channel would be roughly 96 dB down, which is not a link, it is a wall.

and the better argument is the one after that: you do not actually need all 380 channels standing by. the provider allocates the subgraph that is in demand right now and spends the leftover bandwidth boosting the links that are busy. a passive tree has to pre-build every connection it might ever need. the switch builds only the ones you asked for.

## what happened after 2021

the paper ends with a list of its own limitations, which turns out to be an unusually accurate roadmap. it wanted a metric better than coincidence rate, it wanted clock synchronization between separated nodes, and it wanted a source with flat flux across the whole band. all three got done, mostly by the same people.

### it went outside, the same year

[Alshowkan et al., "Reconfigurable Quantum Local Area Network Over Deployed Fiber," *PRX Quantum* 2, 040304 (2021)](https://doi.org/10.1103/PRXQuantum.2.040304) took the idea off the optical table and onto deployed fiber between three buildings on the Oak Ridge campus, GPS synchronized, with eight reconfigurable channels.

it also swapped the metric. instead of coincidence rate, links are rated by **log-negativity, in entangled bits per second**, which is a real information-theoretic quantity rather than a proxy for one. and they ran remote state preparation across it, the first time that protocol had been done on deployed fiber.

### then it got a proper clock

[Alshowkan et al., "Advanced architectures for high-performance quantum networking," *JOCN* 14, 493 (2022)](https://opg.optica.org/jocn/abstract.cfm?uri=jocn-14-6-493) replaced GPS with White Rabbit switches between the three nodes, and added a QKD channel to secure the classical control traffic that runs the instruments. measured fidelities of 0.938 to 0.971 and entanglement rates of 19.6 to 145 ebits/s on the allocated channels.

if you read [the coexistence post]({% post_url 2026-03-23-coexistence-running-a-quantum-channel-and-its-clock-down-the-same-fiber %}), that is the same White Rabbit, doing the same job, and the wavelength-choice argument in that post applies directly to this architecture.

### the source got much wider

[Alshowkan et al., "Broadband polarization-entangled source for C+L-band flex-grid quantum networks," *Opt. Lett.* 47, 6480 (2022)](https://opg.optica.org/ol/abstract.cfm?uri=ol-47-24-6480) covers 1530 to 1625 nm, **7.5 THz in one box**, characterized as 150 pairs of 25 GHz channels with an average fidelity of 0.98 and more than 181 kebits/s of distillable entanglement.

put next to the 310 GHz and 12 channels of the original paper, that is a factor of 24 in bandwidth and 12 in channel count, and it is exactly the "uniform flux across the entire C band" the 2021 paper asked for.

### allocation turned into a solver

this is where it stopped being a photonics problem.

**2022.** [Optimal resource allocation for flexible-grid entanglement distribution networks](https://arxiv.org/abs/2204.06642) put upper bounds on achievable fidelity and ebit rate, derived a condition on detector quality and link efficiency for whether a link is viable at all, and ran a genetic algorithm to find allocations, checked against the deployed network.

**2024.** [Routing and Spectrum Allocation in Broadband Quantum Entanglement Distribution](https://arxiv.org/abs/2404.08744) settled the complexity question: the routing half admits an optimal polynomial-time algorithm, and max-min fair spectrum allocation is **NP-hard**. so you are in approximation territory, and the paper offers two polynomial-time approximations that behave well. it also makes a point worth remembering, which is that maximizing the minimum rate can be a bad objective if you also care about the median.

**2026.** [Goisman et al., "Efficient routing and spectrum allocation in arbitrary flex-grid entanglement networks"](https://arxiv.org/abs/2607.15465) is the current state of it: a three-stage pipeline, Yen's k-shortest-paths run twice to get low-loss candidate routes between users and sources, then APOPT to pick channel allocations that maximize rate under a fidelity constraint, then a CP-SAT solver to assign actual frequency bins without two sources colliding on the same one. tested on a ring and on a **Manhattan incumbent local exchange carrier topology**, and it beats the genetic algorithm on speed, accuracy and scale.

that last one is worth sitting with. the problem has fully converged onto routing and spectrum assignment, the same RSA problem elastic optical networks have been chewing on since about 2010, with a fidelity constraint bolted on and multiple sources instead of one transmitter. twenty years of classical work just became applicable.

### the network grew a control plane

[Alshowkan et al., "Resilient Entanglement Distribution in a Multihop Quantum Network," *JLT* 43, 9016 (2025)](https://arxiv.org/abs/2407.20443) runs six nodes across three buildings in a mesh, with quantum, data and control planes, and reroutes entanglement around a failed link. that is protection switching, on a quantum network, which is roughly the least glamorous and most necessary thing on this list.

### the switch started shrinking

the WaveShaper in the original paper is a rack instrument. in 2024 the group demonstrated a [foundry-fabricated photonic integrated circuit doing flex-grid entanglement distribution](https://www.osti.gov/biblio/2472722) on a CMOS process. if that path works, the middle box goes from an instrument to a die.

### other people went other ways

not everything descends from this paper.

a group at Nanjing put the source [on a silicon chip](https://arxiv.org/abs/2503.07198), a nanowire generating polarization-entangled pairs by four-wave mixing across the whole 4.5 THz C band, and ran it over 30 km of deployed metro fiber between two campuses with 22 channel pairs available.

a group at USTC built a [state-multiplexing source](https://www.nature.com/articles/s41377-025-01805-1) that pumps a microring with two lasers at once, so one wavelength correlates with three others. four users on six wavelength channels instead of twelve, which attacks the spectrum problem from the source end rather than the switch end.

and a group at Heriot-Watt [skipped wavelength entirely](https://www.nature.com/articles/s41566-025-01806-x), connecting eight users through the natural mode mixing inside 30 cm of multimode fiber used as a programmable 8x8 circuit, with entanglement swapping between distant pairs above 77% fidelity and two weeks of stability without recalibration. different degree of freedom, same provisioning idea.

## what i'd build next

here is what i think is still missing. some of these are small and some are not.

**1. put the Raman term in the objective function.** every RSA solver above optimizes against loss and fidelity. but on a real deployed fiber the noise floor in a given quantum channel depends on which *classical* channels are lit on the same glass and how hard they are being driven, because spontaneous Raman scattering in silica dumps noise across tens of THz. that cost is wavelength-dependent, distance-dependent and traffic-dependent, and as far as i can tell nobody has written a spectrum assignment problem where the cost of a frequency bin is a function of the classical traffic matrix. the same group publishes on both, and has a whole [review of hybrid classical-quantum networks](https://doi.org/10.1016/j.pquantelec.2025.100586) to draw the noise model from. this is sitting right there.

**2. close the loop.** allocation today is open-loop: solve it, program the switch, then measure and hope. the missing input just arrived, though. [in situ process tomography of a polarization-stabilized channel](https://arxiv.org/abs/2510.26034) gives you a live quantum map per channel, including noise the classical polarization tracker cannot see. feed that back into the allocator and you have a controller with a time constant instead of an optimization you run at commissioning.

**3. somebody publish the reconfiguration time.** i cannot find it stated anywhere. the switch itself changes profile quickly, but after the spectrum moves, the polarization compensation at each user has to re-settle, and that part could be much slower. the total is the service-change latency of a flex-grid quantum network, and it decides whether "on demand" means seconds or means minutes. it is an afternoon's measurement and it is the number an operator would ask for first.

**4. flatten the source instead of scheduling around it.** a good chunk of the allocation difficulty is the sinc-squared envelope. the outermost channel carries about four percent of the flux of the innermost one, so the solver spends its effort compensating for a shape the source imposed for no useful reason. chirped or apodized poling can flatten a joint spectrum considerably. every GHz of flatness is a constraint the allocator no longer has to satisfy, and unlike solver improvements it helps every allocation at once.

**5. admission control, not silent dropping.** link CD was dropped. on a real network that is an outage, and "the max-min solver decided" is not an incident report. classical networks have committed and peak rates, admission control, and a refusal with a reason. a quantum network layer should be able to tell Carol and Dave at request time that their link is infeasible with the detectors they own, and offer them a lower service class or a longer integration time rather than nothing. the hardware knows. the protocol does not ask.

**6. reconcile the grid with memory bandwidth.** the moment repeaters enter the picture, slices have to match the acceptance bandwidth of a quantum memory, which is typically hundreds of MHz to a few GHz. [spectrally multiplexed repeater work](https://www.nature.com/articles/s41534-024-00946-2) is already asking for mode spacings in that range. a WSS bottoms out around 6.25 GHz. so either sources get cavity-enhanced down to memory linewidths, or a second, finer filtering stage appears after the switch, and either way the flexible grid gets a lot less flexible. this is where the current architecture and the repeater architecture collide, and nobody has reconciled them yet.

**7. treat the slice width as a clock decision too.** this one seems under-stated in the literature.

a narrower slice is a longer photon. that is just the transform limit. and from [the coexistence post]({% post_url 2026-03-23-coexistence-running-a-quantum-channel-and-its-clock-down-the-same-fiber %}), the indistinguishability of two photons meeting at a beamsplitter for entanglement swapping depends only on the ratio of clock jitter to photon duration:

```
    I  =  1 / sqrt( 1 + δt² / 2σ² )
```

so the width you choose for bandwidth reasons also sets how good your clock has to be. the 24 GHz slices in this paper give roughly 18 ps photons, against which White Rabbit's ~2 ps of jitter is nothing. the full 310 GHz biphoton, undivided, is a 1.4 ps photon, and the same White Rabbit link is nowhere near good enough for a swap.

<div class="qw" id="qw-slice" data-qw><div class="qw-hd"><span class="qw-t">one knob, three consequences</span><span class="qw-s">slice width sets how many users you can serve, how much flux each gets, and how good your clock has to be. transform-limited Gaussian approximation.</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">real values</span><span class="qw-b" data-grp="ps"><button type="button" data-v="6.25">WSS floor, 6.25</button><button type="button" data-v="12.5">20-user plan, 12.5</button><button type="button" data-v="24" class="on" aria-pressed="true">this paper, 24</button><button type="button" data-v="310">whole biphoton, 310 GHz</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">slice width <b id="qws-bv">24.0 GHz</b></span><input type="range" id="qws-b" min="0" max="100" step="0.2" value="46"></label><label class="qw-sr"><span class="qw-l">clock jitter <span class="qw-gk">δt</span> <b id="qws-jv">2.18 ps</b></span><input type="range" id="qws-j" min="0.1" max="50" step="0.02" value="2.18"></label></div><div class="qw-two"><div class="qw-pane"><span class="qw-pl">indistinguishability vs slice width</span><svg class="qw-svg" viewBox="0 0 320 180" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Indistinguishability versus slice width"><g id="qws-curve"></g></svg></div><div class="qw-pane"><span class="qw-pl">what the same knob costs you</span><svg class="qw-svg" viewBox="0 0 320 180" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Channel count and per-channel flux versus slice width"><g id="qws-cost"></g></svg></div></div><div class="qw-out"><div class="qw-oi"><span class="qw-ok">photon duration</span><span class="qw-ov" id="qws-dur">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">indistinguishability</span><span class="qw-ov" id="qws-I">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">channels in 7.5 THz</span><span class="qw-ov" id="qws-n">&mdash;</span></div><div class="qw-oi qw-grow"><span class="qw-ok">verdict</span><span class="qw-ov qw-txt" id="qws-v">&mdash;</span></div></div><p class="qw-n" id="qws-note"></p><noscript><p class="qw-n">A transform-limited 24 GHz slice is an 18 ps photon, against which 2.18 ps of clock jitter gives an indistinguishability of 0.98. The undivided 310 GHz biphoton is a 1.4 ps photon, and the same clock gives 0.37. Narrower slices serve more users and relax the clock, at the cost of flux per link.</p></noscript></div>

which means the grid granularity, the user count, the per-link rate and the synchronization budget are all one decision, and right now they get made by different people in different papers. an allocator that knew the clock quality of each node could pick slice widths that keep every link swappable, instead of optimizing rate and finding out later that nothing on the network can interfere.

## the short version

- entanglement from a central source is addressed by wavelength, because the two photons of a pair always add up to the pump. handing two users energy-matched slices is what connects them.
- the passive DWDM tree that did this before scales at `2N² − 3N` filters, and its worst channel is 60 dB down at 16 users. the spread between channels hurts more than the average does.
- one wavelength selective switch replaces the tree at a **fixed 4.5 dB regardless of user count**, and can be reprogrammed.
- the crossover is around four or five users. below that, the passive tree wins, and the paper says so.
- **most of the win came from not allocating alphabetically.** same fixed grid, sensible ordering: 4200x imbalance down to 26x. going to full flex bought the last factor of 13.
- balancing required using the low-flux wings of the spectrum, which earlier work avoided.
- link CD was dropped because two bad detectors cannot be rescued by any allocation. a switch redistributes entanglement, it does not make any.
- per-channel fidelity does not predict the fidelity of a *group* of channels. allocations have to be tested in place.
- one 20-port switch at 12.5 GHz slices covers a fully connected 20-user network. the DWDM equivalent is 740 filters.
- since 2021: deployed over three buildings, White Rabbit clocked, sources widened to 7.5 THz, metric moved to ebits/s, allocation formalized as NP-hard RSA and solved with Yen plus APOPT plus CP-SAT on a Manhattan carrier topology, and the switch is being put on a chip.
- what is missing: Raman noise in the objective, closed-loop reallocation, a published reconfiguration time, flatter sources, honest admission control, and a grid that a quantum memory can actually accept.

what strikes me about this line of work is how quickly the interesting problems stopped being physical. the 2021 paper is an optics experiment. the 2026 paper is a constraint solver run on a map of Manhattan. the entangled photons in the middle did not change much. what changed is that somebody decided to treat them as a resource with a scheduler, and once you do that, the questions become the ones network engineers have been arguing about for decades: fairness, admission, protection, and who gets the good wavelength.

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
.qw-links button{font-family:ui-monospace,SFMono-Regular,Menlo,Consolas,monospace;font-weight:600;min-width:46px;border-left-width:4px}
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
.qw-ch{display:flex;flex-wrap:wrap;gap:4px;margin:8px 0 12px}
.qw-ch button{font:inherit;font-family:ui-monospace,SFMono-Regular,Menlo,Consolas,monospace;font-size:12px;line-height:1;padding:8px 0;width:44px;background:#242424;color:#c8c8c8;border:1px solid #3a3a3a;border-bottom-width:4px;border-radius:4px;cursor:pointer;text-align:center;transition:background .12s,color .12s}
.qw-ch button:hover{background:#2e2e2e;color:#fff}
.qw-ch button:focus-visible{outline:2px solid #32c29e;outline-offset:2px}
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
@media(max-width:600px){.qw{padding:14px 12px 12px}.qw-ctl{gap:12px}.qw-ch button{width:38px;padding:7px 0}}
</style>
<script>
(function(){
  var NS = 'http://www.w3.org/2000/svg';
  function el(t,a,txt){
    var n = document.createElementNS(NS,t);
    for (var k in a) n.setAttribute(k,a[k]);
    if (txt !== undefined) n.appendChild(document.createTextNode(txt));
    return n;
  }
  function clear(g){ while(g.firstChild) g.removeChild(g.firstChild); }
  function btnGroup(root, grp, cb){
    var bs = root.querySelectorAll('[data-grp="'+grp+'"] button');
    Array.prototype.forEach.call(bs, function(b){
      b.addEventListener('click', function(){
        Array.prototype.forEach.call(bs, function(o){ o.classList.remove('on'); o.removeAttribute('aria-pressed'); });
        b.classList.add('on'); b.setAttribute('aria-pressed','true');
        cb(b.getAttribute('data-v'));
      });
    });
  }
  function offGroup(root, grp){
    Array.prototype.forEach.call(root.querySelectorAll('[data-grp="'+grp+'"] button'), function(b){
      b.classList.remove('on'); b.removeAttribute('aria-pressed');
    });
  }
  function sinc2(u){
    if (!u) return 1;
    var x = Math.PI*u;
    var s = Math.sin(x)/x;
    return s*s;
  }

  /* ---------- 1. loss scaling: filter tree vs switch ---------- */
  (function(){
    var H = document.getElementById('qw-loss');
    if (!H) return;
    var WSS = 4.5, NMAX = 20, YMAX = 100;
    var X0 = 44, X1 = 620, Y0 = 16, Y1 = 244;
    var gG = H.querySelector('#qwl-grid'), gC = H.querySelector('#qwl-curves'), gM = H.querySelector('#qwl-mark');
    var sl = H.querySelector('#qwl-n');
    function dwdm(n){ return (n*n - n)*0.25 + 0.6; }
    function count(n){ return 2*n*n - 3*n; }
    function px(n){ return X0 + (n-2)/(NMAX-2)*(X1-X0); }
    function py(d){ return Y1 - Math.min(d,YMAX)/YMAX*(Y1-Y0); }
    function grid(){
      clear(gG);
      [0,20,40,60,80,100].forEach(function(v){
        gG.appendChild(el('line',{x1:X0,y1:py(v),x2:X1,y2:py(v),stroke:'#2e2e2e','stroke-width':1}));
        gG.appendChild(el('text',{x:X0-8,y:py(v)+4,'text-anchor':'end',fill:'#7d7d7d','font-size':10}, v));
      });
      [2,4,8,12,16,20].forEach(function(n){
        gG.appendChild(el('text',{x:px(n),y:Y1+18,'text-anchor':'middle',fill:'#7d7d7d','font-size':10}, n));
      });
      gG.appendChild(el('text',{x:(X0+X1)/2,y:Y1+38,'text-anchor':'middle',fill:'#8f8f8f','font-size':11},'users on the network'));
      gG.appendChild(el('text',{x:X0+2,y:Y0-5,fill:'#8f8f8f','font-size':11},'dB'));
    }
    function curves(){
      clear(gC);
      var d = '';
      for (var n=2;n<=NMAX;n+=0.25) d += (n===2?'M':'L') + px(n).toFixed(1) + ' ' + py(dwdm(n)).toFixed(1);
      gC.appendChild(el('path',{d:d,fill:'none',stroke:'#f2a03d','stroke-width':2.4,'stroke-linecap':'round'}));
      gC.appendChild(el('line',{x1:px(2),y1:py(WSS),x2:px(NMAX),y2:py(WSS),stroke:'#32c29e','stroke-width':2.4}));
      var xc = px(4.5);
      gC.appendChild(el('line',{x1:xc,y1:Y0,x2:xc,y2:Y1,stroke:'#4a4a4a','stroke-width':1,'stroke-dasharray':'4 4'}));
      gC.appendChild(el('text',{x:xc+6,y:Y0+12,fill:'#8f8f8f','font-size':10},'the tree wins to the left of here'));
    }
    function upd(){
      var n = +sl.value, d = dwdm(n), surv = Math.pow(10,-d/10)*100;
      H.querySelector('#qwl-nv').textContent = n;
      clear(gM);
      gM.appendChild(el('line',{x1:px(n),y1:Y0,x2:px(n),y2:Y1,stroke:'#5a5a5a','stroke-width':1}));
      gM.appendChild(el('circle',{cx:px(n),cy:py(WSS),r:4.5,fill:'#fff',stroke:'#32c29e','stroke-width':2}));
      if (d <= YMAX) gM.appendChild(el('circle',{cx:px(n),cy:py(d),r:4.5,fill:'#fff',stroke:'#f2a03d','stroke-width':2}));
      H.querySelector('#qwl-cnt').textContent = count(n).toLocaleString() + ' DWDMs';
      H.querySelector('#qwl-dw').textContent = d.toFixed(1) + ' dB';
      var s;
      if (surv >= 1) s = surv.toFixed(1) + '% vs 35.5% for the switch';
      else if (surv >= 1e-4) s = surv.toPrecision(2) + '% vs 35.5% for the switch';
      else s = 'effectively zero vs 35.5% for the switch';
      H.querySelector('#qwl-sv').textContent = s;
      var note;
      if (n <= 4) note = 'At <b>' + n + ' users</b> the passive tree is still ahead, which the paper says outright. Its worst channel costs <b>' + d.toFixed(1) + ' dB</b> against the switch&rsquo;s flat 4.5 dB.';
      else if (n < 10) note = 'At <b>' + n + ' users</b> the tree has crossed over: <b>' + d.toFixed(1) + ' dB</b> on the worst channel, and the spread between best and worst channel is now the thing you cannot engineer around.';
      else note = 'At <b>' + n + ' users</b> the tree needs <b>' + count(n).toLocaleString() + ' filters</b> and its worst channel is <b>' + d.toFixed(1) + ' dB</b> down. The switch is still 4.5 dB, on every channel, and it can be reprogrammed.';
      H.querySelector('#qwl-note').innerHTML = note;
    }
    grid(); curves();
    sl.addEventListener('input', upd);
    upd();
  })();

  /* ---------- 2. the allocation playground ---------- */
  (function(){
    var H = document.getElementById('qw-alloc');
    if (!H) return;
    var A = 350, SLICE = 24, NCH = 12;
    var W = [];
    for (var k=1;k<=NCH;k++) W.push(sinc2(SLICE*k/A));
    var ETA = {A:0.85,B:0.85,C:0.2,D:0.1}, GATED = {C:1,D:1};
    var LINKS = ['AB','AC','AD','BC','BD','CD'];
    var COL = {AB:'#32c29e',AC:'#f2a03d',AD:'#6ba3f0',BC:'#d8c257',BD:'#c77dd8',CD:'#e2603f'};
    function eff(L){
      var x = L[0], y = L[1], e = ETA[x]*ETA[y];
      if (GATED[x] || GATED[y]) e *= 0.1;
      return e;
    }
    var PRESET = {
      alpha: {AB:[1,2],AC:[3,4],AD:[5,6],BC:[7,8],BD:[9,10],CD:[11,12]},
      rebal: {CD:[1,2],AD:[3,4],BD:[5,6],AC:[7,8],BC:[9,10],AB:[11,12]},
      flex:  {AC:[1],BC:[2],AD:[3,4,7],BD:[5,6,8,9,10,11],AB:[12]}
    };
    var owner = {}, sel = 'AB';
    function load(name){
      owner = {};
      var p = PRESET[name];
      for (var L in p) p[L].forEach(function(c){ owner[c] = L; });
      draw();
    }
    var gS = H.querySelector('#qwa-spec'), gB = H.querySelector('#qwa-bars'), chWrap = H.querySelector('#qwa-chans');
    var chBtns = [];
    for (var c=1;c<=NCH;c++){
      (function(c){
        var b = document.createElement('button');
        b.type = 'button';
        b.textContent = c;
        b.addEventListener('click', function(){
          owner[c] = (owner[c] === sel) ? null : sel;
          offGroup(H,'ps');
          draw();
        });
        chWrap.appendChild(b);
        chBtns.push(b);
      })(c);
    }
    function rates(){
      var r = {};
      LINKS.forEach(function(L){
        var w = 0;
        for (var c=1;c<=NCH;c++) if (owner[c] === L) w += W[c-1];
        r[L] = w > 0 ? eff(L)*w : 0;
      });
      return r;
    }
    function spectrum(){
      clear(gS);
      var X0 = 30, X1 = 616, YB = 150, YT = 18, F = 310;
      function sx(f){ return (X0+X1)/2 + f/F*((X1-X0)/2); }
      function sy(v){ return YB - v*(YB-YT); }
      var d = '';
      for (var f=-F;f<=F;f+=3) d += (f===-F?'M':'L') + sx(f).toFixed(1) + ' ' + sy(sinc2(f/A)).toFixed(1);
      gS.appendChild(el('path',{d:d,fill:'none',stroke:'#4a4a4a','stroke-width':1.4,'stroke-dasharray':'3 3'}));
      for (var k=1;k<=NCH;k++){
        var o = owner[k], col = o ? COL[o] : '#333';
        [1,-1].forEach(function(s){
          var cf = s*SLICE*k, x = sx(cf - s*SLICE/2), x2 = sx(cf + s*SLICE/2);
          var xa = Math.min(x,x2), wd = Math.abs(x2-x);
          var h = (YB - sy(W[k-1]));
          gS.appendChild(el('rect',{x:xa+0.6,y:sy(W[k-1]),width:Math.max(wd-1.2,1),height:Math.max(h,1.5),
            fill:col,'fill-opacity':o?0.75:0.5,stroke:col,'stroke-width':o?1:0.8}));
        });
      }
      gS.appendChild(el('line',{x1:X0,y1:YB,x2:X1,y2:YB,stroke:'#4a4a4a','stroke-width':1}));
      [-288,-144,0,144,288].forEach(function(f){
        gS.appendChild(el('text',{x:sx(f),y:YB+16,'text-anchor':'middle',fill:'#7d7d7d','font-size':10}, f));
      });
      gS.appendChild(el('text',{x:(X0+X1)/2,y:YB+34,'text-anchor':'middle',fill:'#8f8f8f','font-size':11},'detuning from the center of the biphoton, GHz'));
      gS.appendChild(el('text',{x:sx(0),y:YT-4,'text-anchor':'middle',fill:'#7d7d7d','font-size':10},'stopband'));
    }
    function bars(){
      clear(gB);
      var r = rates(), live = LINKS.filter(function(L){ return r[L] > 0; });
      var X0 = 60, X1 = 616, YB = 158, YT = 20, DEC = 4;
      var lo = live.length ? Math.min.apply(null, live.map(function(L){ return r[L]; })) : 1;
      var hi = live.length ? Math.max.apply(null, live.map(function(L){ return r[L]; })) : 1;
      var bw = (X1-X0)/LINKS.length;
      function hy(rel){ return Math.max(0, (1 + Math.log(rel)/Math.LN10/DEC))*(YB-YT); }
      function fmt(x){ return x < 9.95 ? x.toFixed(1) : (x < 9950 ? Math.round(x).toLocaleString() : (x/1000).toFixed(0)+'k'); }
      [1,0.1,0.01,0.001,0.0001].forEach(function(g,i){
        var y = YB - hy(g);
        gB.appendChild(el('line',{x1:X0,y1:y,x2:X1,y2:y,stroke:i?'#282828':'#3d3d3d','stroke-width':1}));
        gB.appendChild(el('text',{x:X0-8,y:y+4,'text-anchor':'end',fill:'#7d7d7d','font-size':9.5},
          i===0 ? 'the best link' : '\u00f7' + (g>=0.001 ? Math.round(1/g).toLocaleString() : '10k')));
      });
      LINKS.forEach(function(L,i){
        var x = X0 + i*bw + bw*0.18, w = bw*0.64;
        if (r[L] > 0){
          var rel = r[L]/hi, h = Math.max(3, hy(rel));
          gB.appendChild(el('rect',{x:x,y:YB-h,width:w,height:h,fill:COL[L],'fill-opacity':0.8,stroke:COL[L],'stroke-width':1,rx:2}));
          gB.appendChild(el('text',{x:x+w/2,y:YB-h-6,'text-anchor':'middle',fill:'#cfcfcf','font-size':10.5},
            rel > 0.999 ? 'best' : '\u00f7'+fmt(1/rel)));
        } else {
          gB.appendChild(el('text',{x:x+w/2,y:YB-8,'text-anchor':'middle',fill:'#6a6a6a','font-size':10.5},'dark'));
        }
        gB.appendChild(el('text',{x:x+w/2,y:YB+17,'text-anchor':'middle',fill:COL[L],'font-size':11.5,'font-weight':600}, L));
      });
      return {ratio:hi/lo, live:live.length};
    }
    function verdict(ratio, live){
      if (live < 2) return ['not much of a network','#e2603f'];
      if (ratio < 2.5) return ['balanced, this is the flex-grid result','#32c29e'];
      if (ratio < 12) return ['usable spread','#d8c257'];
      if (ratio < 200) return ['one link is a second-class citizen','#f2a03d'];
      return ['the weak links are barely links','#e2603f'];
    }
    function draw(){
      Array.prototype.forEach.call(H.querySelectorAll('[data-grp="lk"] button'), function(b){
        b.style.borderLeftColor = COL[b.getAttribute('data-v')];
      });
      chBtns.forEach(function(b,i){
        var o = owner[i+1];
        b.style.borderBottomColor = o ? COL[o] : '#3a3a3a';
        b.style.color = o ? COL[o] : '#7a7a7a';
        b.title = o ? 'channel '+(i+1)+', link '+o : 'channel '+(i+1)+', unallocated';
      });
      spectrum();
      var m = bars();
      var vd = verdict(m.ratio, m.live);
      H.querySelector('#qwa-live').textContent = m.live + ' of 6';
      H.querySelector('#qwa-ratio').textContent = m.live < 2 ? '\u2014'
        : (m.ratio < 9.95 ? m.ratio.toFixed(1) : Math.round(m.ratio).toLocaleString()) + 'x';
      var ve = H.querySelector('#qwa-v'); ve.textContent = vd[0]; ve.style.color = vd[1];
      var unal = 0; for (var c=1;c<=NCH;c++) if (!owner[c]) unal++;
      H.querySelector('#qwa-note').innerHTML =
        'Rates come from a simple model: pair flux follows the sinc-squared envelope, and a link&rsquo;s rate is its total allocated flux times the product of the two users&rsquo; detector efficiencies, with one factor of 10 for the APD gate duty cycle. It reproduces the paper&rsquo;s rebalanced case (24x against a measured ~26) and its full-flex case (1.9x against \"within a factor of 2\"), and runs about 30% high on the alphabetical case, where afterpulsing and dark counts on the APD links are doing the rest. '
        + (unal ? '<b>' + unal + ' channel' + (unal>1?'s are':' is') + ' unallocated</b>, which is spectrum you are paying for and not using.' : 'Every channel is allocated.');
    }
    btnGroup(H,'ps', load);
    btnGroup(H,'lk', function(v){ sel = v; });
    load('alpha');
  })();

  /* ---------- 3. slice width as a clock decision ---------- */
  (function(){
    var H = document.getElementById('qw-slice');
    if (!H) return;
    var BMIN = 5, BMAX = 400, TOT = 7500;
    var sb = H.querySelector('#qws-b'), sj = H.querySelector('#qws-j'), bFix = null;
    function pos2b(p){ return BMIN*Math.pow(BMAX/BMIN, p/100); }
    function b2pos(b){ return 100*Math.log(b/BMIN)/Math.log(BMAX/BMIN); }
    function sigma(b){ return 441/b/2.3548; }
    function ind(b,dt){ var s = sigma(b); return 1/Math.sqrt(1 + dt*dt/(2*s*s)); }
    var gC = H.querySelector('#qws-curve'), gK = H.querySelector('#qws-cost');
    function curve(b,dt){
      clear(gC);
      var X0 = 34, X1 = 310, Y0 = 14, Y1 = 140;
      function cx(v){ return X0 + Math.log(v/BMIN)/Math.log(BMAX/BMIN)*(X1-X0); }
      function cy(v){ return Y1 - v*(Y1-Y0); }
      [0,0.5,1].forEach(function(v){
        gC.appendChild(el('line',{x1:X0,y1:cy(v),x2:X1,y2:cy(v),stroke:v===0.5?'#4a3a2a':'#2e2e2e','stroke-width':1,'stroke-dasharray':v===0.5?'4 3':''}));
        gC.appendChild(el('text',{x:X0-7,y:cy(v)+4,'text-anchor':'end',fill:'#7d7d7d','font-size':9}, v.toFixed(1)));
      });
      [6.25,25,100,400].forEach(function(t){
        gC.appendChild(el('text',{x:cx(t),y:Y1+16,'text-anchor':'middle',fill:'#7d7d7d','font-size':9}, t));
      });
      gC.appendChild(el('text',{x:(X0+X1)/2,y:Y1+32,'text-anchor':'middle',fill:'#8f8f8f','font-size':10},'slice width, GHz'));
      var d = '';
      for (var p=0;p<=100;p+=1){ var v = pos2b(p); d += (p?'L':'M') + cx(v).toFixed(1) + ' ' + cy(ind(v,dt)).toFixed(1); }
      gC.appendChild(el('path',{d:d,fill:'none',stroke:'#32c29e','stroke-width':2.2,'stroke-linecap':'round'}));
      gC.appendChild(el('circle',{cx:cx(b),cy:cy(ind(b,dt)),r:4.5,fill:'#fff',stroke:'#32c29e','stroke-width':2}));
    }
    function cost(b){
      clear(gK);
      var X0 = 34, X1 = 310, Y0 = 14, Y1 = 140, DEC = 1.4;
      function cx(v){ return X0 + Math.log(v/BMIN)/Math.log(BMAX/BMIN)*(X1-X0); }
      function cy(r){ return (Y0+Y1)/2 - Math.log(r)/Math.LN10/DEC*((Y1-Y0)/2); }
      [0.1,1,10].forEach(function(r){
        gK.appendChild(el('line',{x1:X0,y1:cy(r),x2:X1,y2:cy(r),stroke:r===1?'#3a3a3a':'#2a2a2a','stroke-width':1}));
        gK.appendChild(el('text',{x:X0-7,y:cy(r)+4,'text-anchor':'end',fill:'#7d7d7d','font-size':9}, r===1?'1x':(r<1?'0.1x':'10x')));
      });
      [6.25,25,100,400].forEach(function(t){
        gK.appendChild(el('text',{x:cx(t),y:Y1+16,'text-anchor':'middle',fill:'#7d7d7d','font-size':9}, t));
      });
      gK.appendChild(el('text',{x:(X0+X1)/2,y:Y1+32,'text-anchor':'middle',fill:'#8f8f8f','font-size':10},'slice width, GHz'));
      var d1 = '', d2 = '';
      for (var p=0;p<=100;p+=1){
        var v = pos2b(p);
        d1 += (p?'L':'M') + cx(v).toFixed(1) + ' ' + cy(24/v).toFixed(1);
        d2 += (p?'L':'M') + cx(v).toFixed(1) + ' ' + cy(v/24).toFixed(1);
      }
      gK.appendChild(el('path',{d:d1,fill:'none',stroke:'#6ba3f0','stroke-width':2,'stroke-linecap':'round'}));
      gK.appendChild(el('path',{d:d2,fill:'none',stroke:'#f2a03d','stroke-width':2,'stroke-linecap':'round'}));
      gK.appendChild(el('text',{x:X0+4,y:Y0+10,fill:'#6ba3f0','font-size':9},'channels you can serve'));
      gK.appendChild(el('text',{x:X0+4,y:Y1-6,fill:'#f2a03d','font-size':9},'flux per link'));
      gK.appendChild(el('circle',{cx:cx(b),cy:cy(24/b),r:4,fill:'#fff',stroke:'#6ba3f0','stroke-width':2}));
      gK.appendChild(el('circle',{cx:cx(b),cy:cy(b/24),r:4,fill:'#fff',stroke:'#f2a03d','stroke-width':2}));
    }
    function verdict(I){
      if (I >= 0.98) return ['a swap works, the clock is a non-issue','#32c29e'];
      if (I >= 0.90) return ['a swap works, contrast slightly down','#32c29e'];
      if (I >= 0.75) return ['marginal for a Bell state measurement','#d8c257'];
      if (I >= 0.5)  return ['the clock is now your limit','#f2a03d'];
      return ['the photons are distinguishable, no swap','#e2603f'];
    }
    function upd(){
      var b = (bFix !== null) ? bFix : pos2b(+sb.value), dt = +sj.value;
      var fw = 441/b, sg = sigma(b), I = ind(b,dt), n = Math.floor(TOT/(2*b));
      H.querySelector('#qws-bv').textContent = (b<10?b.toFixed(2):b.toFixed(1)) + ' GHz';
      H.querySelector('#qws-jv').textContent = dt.toFixed(2) + ' ps';
      H.querySelector('#qws-dur').textContent = fw.toFixed(1) + ' ps';
      H.querySelector('#qws-I').textContent = I.toFixed(3);
      H.querySelector('#qws-n').textContent = n.toLocaleString();
      var vd = verdict(I);
      var ve = H.querySelector('#qws-v'); ve.textContent = vd[0]; ve.style.color = vd[1];
      curve(b,dt); cost(b);
      H.querySelector('#qws-note').innerHTML =
        'A <b>' + (b<10?b.toFixed(2):b.toFixed(0)) + ' GHz</b> slice is a <b>' + fw.toFixed(1) + ' ps</b> photon (σ = ' + sg.toFixed(1) +
        ' ps), so <b>' + dt.toFixed(2) + ' ps</b> of jitter gives an indistinguishability of <b>' + I.toFixed(3) +
        '</b>. The same width fits <b>' + n.toLocaleString() + ' channels</b> into a 7.5 THz source and gives each link <b>' +
        (b/24).toFixed(2) + 'x</b> the flux of this paper&rsquo;s 24 GHz slices. Narrow buys you users and clock margin, wide buys you rate.';
    }
    sb.addEventListener('input', function(){ bFix = null; offGroup(H,'ps'); upd(); });
    sj.addEventListener('input', upd);
    btnGroup(H,'ps', function(v){ bFix = +v; sb.value = b2pos(bFix); upd(); });
    bFix = 24; sb.value = b2pos(24);
    upd();
  })();
})();
</script>{% endraw %}