---
layout: post
section-type: post
title: "Quantum data centers : when a hop costs an exponent"
category: 'networking'
tags: [ 'networking', 'quantum', 'datacenter' ]
---

the classical data center topology argument was settled a long time ago. fat-tree and dragonfly in 2008, BCube the year after, and by about 2015 the interesting question had moved from *which topology* to *what does a port cost*. the topologies became a catalogue. pick your oversubscription ratio, pick a radix, wire it up, and if you got it wrong you buy more spine switches.

a quantum data center is the same shape of problem with one term changed. still machines in racks, still switches between them. but what moves between the machines is entanglement, and entanglement is not a packet you can buffer and forward. it is a thing you *attempt* to create, over and over, until one attempt survives the fiber.

which means an extra hop does not add latency. it multiplies it.

this paper takes four topologies straight out of the classical catalogue, drops them into a quantum data center model, and reports what survives the translation. [Pouryousef, Towsley, Kaur, Kompella, Shapourian and Nejabati, "Benchmarking Quantum Data Center Architectures: A Performance and Scalability Perspective," arXiv:2601.01353](https://arxiv.org/abs/2601.01353), January 2026, Cisco Research and UMass Amherst. QFly, BCube, Clos and Fat-Tree, from 8 to 150 QPUs, under three workload shapes.

the headline finding is that **the ranking flips**. hold one resource fixed and Fat-Tree and Clos win. hold a different one fixed and QFly wins by a distance. the paper reports the flip as a result. i think it is a result about the experiment rather than about the architectures, and that reading it that way is more useful. more on that at the end.

below: what a quantum data center is made of, the one equation the whole paper runs on, why the fiber turns out to be irrelevant, what the four topologies actually do differently, and then what i'd want before i believed any of the rankings.

## what is in the box

a QPU here is a small quantum processor: 16 data qubits that hold the algorithm's state, and 5 **communication qubits** that talk to the outside world. the data qubits never leave. the communication qubits are the network interface.

between the QPUs sits an optical fabric with four kinds of part:

- **EPPS**, entangled photon pair sources. they fire probabilistically. you do not ask for a pair, you take what arrives.
- **optical switches**, which set up the light path. these are the topology.
- **BSM modules**, Bell state measurement. two photons arrive, interfere, and if the detectors click in the right pattern the two distant qubits behind them are now entangled. this is the thing that actually creates a link.
- **single photon detectors**.

the reason you want any of this is **gate teleportation**. if a circuit needs a two-qubit gate between a qubit on QPU 3 and a qubit on QPU 47, and those are different machines, you cannot just run the gate. what you do instead is consume a pre-shared entangled pair between the two QPUs, plus two classical bits, and the gate happens as if the qubits were adjacent.

so every non-local gate in the circuit is a *request to the network for one EPR pair*, and the question the paper is asking is how long that request takes and what it contends with. it is the same primitive that [an entanglement distribution network]({% post_url 2026-05-29-eight-users-on-one-chip-maximizing-the-source-side-of-a-flex-grid-quanntum-network %}) hands to its users, with the distances collapsed from kilometres to a hundred metres. collapsing them changes which term dominates, and that turns out to matter enormously.

## the one equation

here is the whole paper in three lines. an attempt to make an EPR pair takes

```
    T_att  =  T_src  +  2D/v_fiber  +  T_reset
```

the source period, the round trip for the heralding signal to come back, and the time to reset the communication qubit for another go. that attempt succeeds with a probability equal to the end-to-end transmittance of the optical path:

```
    T_chan  =  10^(−L_tot/10)
```

and attempts are independent, so the number of them you need is geometric and the expected time to get one pair is

```
    E[T_pair]  ≈  T_att / T_chan  =  T_att × 10^(L_tot/10)
```

**that is the mechanism behind every result in the paper.** loss goes into an exponent. every 3 dB you add to the path doubles the time to get a pair, forever, with no diminishing returns and no way to buffer your way out of it.

and the loss you are adding is:

```
    L_tot  =  L_fiber  +  L_sw  +  L_BSM  (+ L_mem)
```

with the switch term being the sum over every switch the photon passes through. a single switch of radix `k` is internally a tree of 2×2 elements, so its insertion loss goes as

```
    L_sw(k)  ≈  (2⌈log₂ k⌉ − 1) × ℓ_2×2
```

now look at the shape of those two expressions together, because it is the whole design argument. **hop count multiplies the switch loss. radix only adds a logarithm to it.** a path through five switches costs five times whatever one switch costs. making a switch eight times bigger costs you three more stages.

so in a quantum data center, flattening the topology is not a cost optimization the way it is classically. it is a physics optimization, and it is the only one available.

### and the fiber does not matter at all

this is the part that surprised me, and it falls out of the paper's own parameter table.

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Parameter</th><th>Value</th><th>What it decides</th></tr>
</thead>
<tbody>
<tr><td>QPU to switch, switch to switch</td><td>0.1 km</td><td>nothing, as it turns out</td></tr>
<tr><td>Fiber attenuation</td><td>0.1 dB/km</td><td>nothing, as it turns out</td></tr>
<tr><td>Switch insertion loss, per 2&times;2 stage</td><td>0.3 dB</td><td>the entire loss budget</td></tr>
<tr><td>Communication qubit memory loss</td><td>3 dB</td><td>whether repeaters are worth it</td></tr>
<tr><td>Communication qubit reset time</td><td>10 &micro;s</td><td>the attempt clock</td></tr>
<tr><td>EPR source rate</td><td>10<sup>6</sup> pairs/s</td><td>less than you would think</td></tr>
<tr><td>Coherence cutoff <code>&tau;_cut</code>, BCube only</td><td>200 &micro;s</td><td>whether BCube works at all</td></tr>
<tr><td>Data qubits per QPU</td><td>16</td><td>how many QPUs a circuit needs</td></tr>
<tr><td>Communication qubits per QPU</td><td>5</td><td>never swept</td></tr>
<tr><td>Optical channels per fiber</td><td>5</td><td>never swept</td></tr>
<tr><td>Total BSM budget</td><td>100</td><td>swept two ways, and that is the problem</td></tr>
</tbody>
</table>
</div>

a six-link path is 0.6 km of fiber, which at 0.1 dB/km is **0.06 dB**. one radix-8 switch is (2×3−1) × 0.3 = **1.5 dB**. a single switch traversal costs twenty-five times what the entire fiber plant costs.

which means the loss budget of a quantum data center is, to within a rounding error, *a count of how many switches you went through*. the glass is free. the same thing is true of the [ORNL campus network](https://doi.org/10.1109/JLT.2025.3581220) at 1.2 km, where a rerouted hop costs about 10 dB and the 250 m of fiber under it costs 0.05, and it holds even harder at 100 m.

play with it:

<div class="qw" id="qw-hop" data-qw><div class="qw-hd"><span class="qw-t">what a hop costs inside the building</span><span class="qw-s">loss is an exponent, so hop count multiplies the time to get a pair. the fiber is the thin sliver on the left of the bar, and it never grows.</span><span class="qw-lg"><i style="background:#6ba3f0"></i>fiber<i style="background:#32c29e"></i>optical switches<i style="background:#f2a03d"></i>BSM and detection</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">paths these topologies actually produce</span><span class="qw-b" data-grp="ps"><button type="button" data-v="rack">same edge switch, 1</button><button type="button" data-v="qfly">QFly, 2</button><button type="button" data-v="pod" class="on" aria-pressed="true">Fat-Tree same pod, 3</button><button type="button" data-v="cross">Fat-Tree across pods, 5</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">switches traversed <b id="qwh-nv">3</b></span><input type="range" id="qwh-n" min="1" max="7" step="1" value="3"></label><label class="qw-sr"><span class="qw-l">switch radix <b id="qwh-kv">8</b></span><input type="range" id="qwh-k" min="0" max="9" step="1" value="1"></label><label class="qw-sr"><span class="qw-l">insertion loss per 2&times;2 stage <b id="qwh-lv">0.30 dB</b></span><input type="range" id="qwh-l" min="0.05" max="2" step="0.05" value="0.3"></label><label class="qw-sr"><span class="qw-l">BSM and detection loss <b id="qwh-bv">0.0 dB</b></span><input type="range" id="qwh-b" min="0" max="8" step="0.25" value="0"></label></div><div class="qw-pane"><span class="qw-pl">where the decibels go</span><svg class="qw-svg" viewBox="0 0 640 58" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Loss budget split between fiber, switches and detection"><g id="qwh-bar"></g></svg></div><div class="qw-pane"><span class="qw-pl">expected time to one EPR pair, against switches traversed</span><svg class="qw-svg" viewBox="0 0 640 210" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Expected EPR generation latency versus number of switches traversed"><g id="qwh-plot"></g></svg></div><div class="qw-out"><div class="qw-oi"><span class="qw-ok">total loss</span><span class="qw-ov" id="qwh-loss">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">success per attempt</span><span class="qw-ov" id="qwh-p">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">attempts needed</span><span class="qw-ov" id="qwh-att">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">E[T<sub>pair</sub>]</span><span class="qw-ov" id="qwh-t">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">fiber&rsquo;s share</span><span class="qw-ov" id="qwh-fs">&mdash;</span></div><div class="qw-oi qw-grow"><span class="qw-ok">against the same path classically</span><span class="qw-ov qw-txt" id="qwh-cl">&mdash;</span></div></div><p class="qw-n" id="qwh-note"></p><noscript><p class="qw-n">At the paper&rsquo;s parameters, a path through one radix-8 switch costs 1.52 dB and delivers an EPR pair in about 18 microseconds. The same path through five switches costs 7.56 dB and takes about 97 microseconds, a factor of five, of which the fiber accounts for 0.8 percent of the loss budget. Raising the per-stage insertion loss to 1 dB turns that five-switch path into 5.5 milliseconds, a factor of 132.</p></noscript></div>

two things worth noticing while the sliders are moving.

**the attempt clock is the memory, not the light.** the source runs at 10⁶ pairs per second, so it could offer an attempt every microsecond. the communication qubit takes 10 µs to reset. so the real attempt rate is 100 kHz, and somewhere between 60 and 77% of every attempt is a qubit waiting to become reusable. everything optical in this system is idling most of the time.

**the classical comparison is not close.** five hops in a classical data center costs you a few extra microseconds of store-and-forward, additively, and nobody designs around it. five switch traversals here costs a factor of five in EPR latency at the paper's optimistic 0.3 dB, and a factor of a hundred and thirty if your switches are 1 dB per stage. **diameter went from a nice-to-have to the dominant design variable**, and that single fact is what the rest of the paper is exploring.

## the four topologies

three of the four are switch-centric: the QPUs sit at the edge and do nothing but consume entanglement. the fabric carries it, and a single BSM somewhere along the path heralds the pair end to end.

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Architecture</th><th>Kind</th><th>Shape</th><th>Switches on the worst path</th></tr>
</thead>
<tbody>
<tr><td>Fat-Tree</td><td>switch-centric</td><td>edge, aggregation, core. <code>k</code> pods, <code>5k&sup2;/4</code> switches, <code>k&sup3;/4</code> QPUs</td><td>5</td></tr>
<tr><td>Clos</td><td>switch-centric</td><td>same three tiers, but radix, rack fanout and fabric size dimensioned separately. two variants, <em>tight</em> and <em>compact</em></td><td>3 to 5</td></tr>
<tr><td>QFly</td><td>switch-centric</td><td>flattened, dragonfly-style. groups of QPUs under high-radix switches, switches linked directly to each other</td><td>2</td></tr>
<tr><td>BCube</td><td><strong>server-centric</strong></td><td><code>k+1</code> switch layers of <code>n</code>-radix switches, every QPU on one switch per layer. QPUs act as repeaters</td><td>2, plus a QPU in the middle</td></tr>
</tbody>
</table>
</div>

BCube is the interesting one, because it changes what a hop *is*.

in the three switch-centric designs, a long path means one long optical path with lots of switches in it, and one BSM at the end of it. the loss all multiplies together and you pay the exponent once, on a big number.

in BCube, a QPU in the middle of the path stores half of a pair, then does an entanglement swap. so the path is broken into segments, and **each segment gets its own exponent on a small number**. that is repeaters, applied at data center scale, which is a slightly startling thing to do over 200 metres of fiber and is exactly why it is worth testing. [the wavelength-selective switch in a metro network]({% post_url 2026-04-04-Flex-grid-for-entanglement %}) is transparent for the same reason a switch-centric fabric is: nothing in the path stores or measures the photon. BCube gives that up on purpose.

the cost of doing that is two-fold. storing a photon in a memory has its own loss, 3 dB in the paper's model. and you now need every segment to be alive at the same time, which is where the **coherence cutoff** `τ_cut` comes in: an early-finishing segment sits there decohering while it waits for its neighbours, and if it waits longer than `τ_cut` the whole thing is thrown away and restarted.

the paper reduces the trade to one number:

```
    ρ_ℓ  =  L_sw per hop  /  L_mem
```

if a switch traversal is cheap relative to a memory access, stay switch-centric and let the light go all the way. if it is expensive, break the path and swap. at the paper's parameters, a radix-12 switch is 2.1 dB and a memory is 3 dB, so `ρ_ℓ = 0.7`, and switch-centric is comfortably ahead. push the insertion loss up and it stops being ahead.

<div class="qw" id="qw-sc" data-qw><div class="qw-hd"><span class="qw-t">let the light through, or break the path and swap</span><span class="qw-s">switch-centric pays one exponent on the whole path. server-centric pays a small exponent per segment, plus a memory, plus the risk that a segment ages out before its neighbours arrive.</span><span class="qw-lg"><i style="background:#32c29e"></i>switch-centric, one BSM<i style="background:#f2a03d"></i>server-centric, swap at each QPU</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">how segments are generated</span><span class="qw-b" data-grp="pr"><button type="button" data-v="par" class="on" aria-pressed="true">all at once</button><button type="button" data-v="seq">one after another</button></span></div><div class="qw-g"><span class="qw-l">jump to</span><span class="qw-b" data-grp="ps"><button type="button" data-v="paper" class="on" aria-pressed="true">the paper&rsquo;s numbers</button><button type="button" data-v="lossy">lossy switches, 1 dB</button><button type="button" data-v="tight">tight coherence, 40 &micro;s</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">hops end to end <b id="qws-nv">4</b></span><input type="range" id="qws-n" min="2" max="7" step="1" value="4"></label><label class="qw-sr"><span class="qw-l">insertion loss per 2&times;2 stage <b id="qws-lv">0.30 dB</b></span><input type="range" id="qws-l" min="0.05" max="2" step="0.05" value="0.3"></label><label class="qw-sr"><span class="qw-l">memory loss <span class="qw-gk">L<sub>mem</sub></span> <b id="qws-mv">3.0 dB</b></span><input type="range" id="qws-m" min="0.5" max="10" step="0.25" value="3"></label><label class="qw-sr"><span class="qw-l">coherence cutoff <span class="qw-gk">&tau;<sub>cut</sub></span> <b id="qws-cv">200 &micro;s</b></span><input type="range" id="qws-c" min="20" max="1000" step="10" value="200"></label></div><div class="qw-pane"><span class="qw-pl">expected time to one end-to-end pair, against path length</span><svg class="qw-svg" viewBox="0 0 640 240" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Switch-centric and server-centric EPR latency versus path length"><g id="qws-plot"></g></svg></div><div class="qw-out"><div class="qw-oi"><span class="qw-ok qw-bs">switch-centric</span><span class="qw-ov" id="qws-a">&mdash;</span></div><div class="qw-oi"><span class="qw-ok qw-fs">server-centric</span><span class="qw-ov" id="qws-b">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">&rho;<sub>&ell;</sub></span><span class="qw-ov" id="qws-r">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">rounds surviving &tau;<sub>cut</sub></span><span class="qw-ov" id="qws-w">&mdash;</span></div><div class="qw-oi qw-grow"><span class="qw-ok">verdict</span><span class="qw-ov qw-txt" id="qws-v">&mdash;</span></div></div><p class="qw-n" id="qws-note"></p><noscript><p class="qw-n">At the paper&rsquo;s parameters a four-hop switch-centric path costs about 8.5 dB and delivers a pair in roughly 110 microseconds, while the server-centric equivalent pays only 5.1 dB per segment but has to keep all four segments alive at once, and lands near 220 microseconds. Raising the per-stage insertion loss to 1 dB reverses the answer: switch-centric goes to about 10 milliseconds and swapping to about 4.6. Shrinking the coherence cutoff to 40 microseconds reverses it back, and by a much larger margin.</p></noscript></div>

the shape to take away: **switch-centric latency curves upward with path length and server-centric does not.** breaking a path into segments converts an exponential in total loss into something much closer to linear, at the price of a memory per segment and a coherence budget that has to cover all of them. that is the same argument for repeaters that holds over 400 km of metro fiber, arriving at 200 metres because the switches are lossy enough to make it true.

whether it arrives depends entirely on `ρ_ℓ`, which is a number about your components and not about your topology.

## what the simulation says

the workloads are three shapes of circuit, each with 16 data qubits per QPU:

- **nearest neighbour**, gates between adjacent qubits in a chain. mostly local.
- **random Clifford+T**, scattered CNOTs.
- **long range**, uniformly random qubit pairs. every gate is a network request.

the metric is

```
    ρ_lat  =  T_distributed / T_monolithic
```

against a hypothetical single QPU that holds every qubit with all-to-all connectivity and no communication at all. ratios near 1 mean the network is nearly free; the paper reports roughly 1 at 16 QPUs and 2 to 4 at 128, depending on workload.

nearest-neighbour circuits stay almost flat, which is unsurprising and is worth stating anyway: **if your compiler can keep the gates local, the topology barely matters.** long-range circuits degrade steepest across every architecture. the entire architectural argument lives in the gap between those two.

### the flip

now the result the paper leads with, and the one i want to look at hardest.

there are two ways to hand out BSM modules, and they give opposite answers.

**fix the BSMs per switch.** every switch gets, say, two. then an architecture's total BSM capacity is proportional to its switch count, and Fat-Tree and Clos_tight, which have a lot of switches, end up with a large aggregate budget and the lowest latency ratios.

**fix the total budget at 100 and spread it evenly.** now more switches means fewer BSMs at each one, and the ones with few switches concentrate the resource. QFly wins across every scale, fully-connected QFly by the most.

at 128 QPUs a Fat-Tree needs about 80 switches and a QFly about 8. so the first accounting model hands Fat-Tree ten times the BSM hardware; the second hands QFly ten times the per-switch capacity. **the two normalizations differ by a factor of a hundred in what they give one architecture relative to the other**, and the ranking flipping between them is not really news about the topologies.

<div class="qw" id="qw-bsm" data-qw><div class="qw-hd"><span class="qw-t">the ranking is a function of the accounting</span><span class="qw-s">same three switch-centric fabrics (Clos in both variants), same workload, three ways of paying for BSM modules. the third one is not in the paper.</span><span class="qw-lg"><i style="background:#32c29e"></i>best<i style="background:#d8c257"></i>middle<i style="background:#e2603f"></i>worst</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">how you pay for BSMs</span><span class="qw-b" data-grp="md"><button type="button" data-v="per" class="on" aria-pressed="true">fixed per switch</button><button type="button" data-v="tot">fixed total budget</button><button type="button" data-v="cost">fixed money</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">non-local gates in flight <b id="qwb-dv">120</b></span><input type="range" id="qwb-d" min="4" max="400" step="4" value="120"></label><label class="qw-sr"><span class="qw-l" id="qwb-xl">BSMs per switch <b id="qwb-xv">2</b></span><input type="range" id="qwb-x" min="1" max="12" step="1" value="2"></label><label class="qw-sr"><span class="qw-l">a BSM costs this many switch ports <b id="qwb-cv">20</b></span><input type="range" id="qwb-c" min="1" max="120" step="1" value="20"></label></div><div class="qw-pane"><span class="qw-pl">time to finish the same long-range workload, lower is better</span><svg class="qw-svg" viewBox="0 0 640 210" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Relative workload completion time for four architectures"><g id="qwb-bars"></g></svg></div><div class="qw-out"><div class="qw-oi qw-grow"><span class="qw-ok">winner</span><span class="qw-ov qw-txt" id="qwb-w">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">spread, best to worst</span><span class="qw-ov" id="qwb-s">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">BSMs in the fabric</span><span class="qw-ov" id="qwb-n">&mdash;</span></div></div><p class="qw-n" id="qwb-note"></p><noscript><p class="qw-n">Under a fixed number of BSMs per switch, the architectures with the most switches (Fat-Tree, Clos_tight) have the largest aggregate BSM budget and finish fastest. Under a fixed total budget spread evenly, the same architectures have the fewest BSMs at each switch and QFly wins instead. The two accounting models differ by about a factor of a hundred in what they hand QFly relative to Fat-Tree at 128 QPUs, so the reversal says more about the normalization than about the fabrics. Pricing switch ports and BSM modules separately gives a single ranking that moves smoothly with the price ratio.</p></noscript></div>

the configurations behind that widget are mine, worked out from the standard topology formulas at 128 QPUs rather than lifted from the paper's own table, and the contention model is a crude `demand / available` queue. it is there to show the shape of the argument, not to reproduce the paper's figures.

but the shape is the point. **hold hardware fixed and you get one ranking, hold money fixed and you get another, and only one of those is a question an operator can act on.**

### two sensitivity results that are worth more than the rankings

**coherence.** BCube's performance rises steadily with `τ_cut` from 50 µs upward. that is the "segments alive together" term in the widget above, and it means BCube's viability is a memory specification, not an architecture choice. a group with 50 µs communication qubits and a group with 500 µs ones should build different data centers.

**insertion loss.** everything degrades as `ℓ_2×2` rises, but Clos and Fat-Tree degrade much faster than QFly and BCube. this is the multiplication from the top of the post showing up in the results, and it is the cleanest finding in the paper: **deep fabrics are a bet on cheap switches.** if optical switch insertion loss improves, the tree topologies get more attractive. if it does not, the flat ones win and it is not close.

that second one is the sentence i'd hang the paper on. the ranking between the four architectures is essentially a forecast of one component parameter.

## what's missing

this is a simulation study, and simulation studies get graded on what they chose to model. these are the choices i'd push on.

**there is no fidelity anywhere.** the whole paper measures latency. it never tracks what state actually arrives. that matters enormously for the switch-centric versus server-centric comparison, because BCube's advantage comes from entanglement swapping, and **every swap compounds the infidelity of both segments** while the switch-centric path has one BSM and no intermediate memories. a server-centric fabric that delivers pairs twice as fast at fidelity 0.8 is not better than a switch-centric one delivering at 0.95. for a distributed circuit it may be useless, because gate teleportation with a low-fidelity pair is just a noisy gate. the paper's own framing is fault-tolerant distributed computing, where the relevant threshold is a fidelity, and the metric it reports cannot see it. this is the gap i'd close first.

**the monolithic baseline is a fiction, and it hides the interesting number.** `T_mono` assumes one QPU holding all the qubits, all-to-all connected, no communication cost. at 128 QPUs that is a 2,048-qubit all-to-all machine, which is precisely the thing distributed quantum computing exists because nobody can build. normalizing to it is fine as a denominator, but it means every reported number is a ratio to an impossibility, and the comparison an operator actually needs, this fabric against that fabric in seconds, is not printed. it also flatters everyone equally: four architectures at ρ_lat of 2 to 4 could all be unusable.

**switch reconfiguration is excluded.** the paper says so plainly, to isolate loss and memory effects, which is a reasonable thing to do in a sensitivity study. but MEMS optical switches settle in a millisecond or so and LCoS parts are slower, while the entire EPR generation being modelled takes tens to hundreds of *microseconds*. **if the fabric reconfigures per gate, reconfiguration is not a correction term, it is the entire runtime**, and the topology ranking is irrelevant next to how many gate requests you can serve per switch configuration. that is a scheduling question the classical optical data center literature already fought over, and it is the biggest omission here.

**the scarce resource that is fixed is the one worth sweeping.** BSM budget gets swept two ways. meanwhile *optical channels per fiber* sits at 5 and never moves, and *communication qubits per QPU* sits at 5. multiplexing is the standard answer to probabilistic entanglement generation: run `m` attempts in parallel on `m` wavelengths and the geometric mean time drops by roughly `m`. that is a direct multiplier on the exponent that the whole paper is organized around. my guess is that a serious multiplexing sweep would compress the differences between all four architectures, which is a more actionable finding than the ranking.

**the workloads are the wrong worst case.** "long range" here means uniformly random qubit pairs. no compiler would ever hand a fabric that. real distributed compilation partitions the circuit to keep gates local, and the interesting benchmark is not *how does topology X do on random gates* but *how much does a bad partition cost me on topology X*. one of these architectures probably degrades gracefully under a mediocre partitioner and another falls off a cliff, and that is the number a systems team would want.

**the swap success probability never appears.** BCube's whole case rests on entanglement swapping at intermediate QPUs, and the probability that a swap succeeds is folded into a loss term rather than stated. with linear-optical Bell measurement it caps at 50%; with matter qubits it can be near-deterministic. those two worlds give completely different answers for BCube and the paper does not say which one it is in.

**the coherence cutoff is applied to BCube only.** switch-centric endpoints also hold entangled state while they wait: for a contended BSM, for the classical correction, for the other pairs a multi-qubit gate needs. giving decoherence to the server-centric design and not to the switch-centric ones puts a thumb on a scale that the paper is otherwise carefully balancing.

**0.1 dB/km and 10 µs are optimistic in opposite directions.** standard single-mode fiber at 1550 nm is about 0.2 dB/km, though at 100 m it makes no difference. the 10 µs reset does make a difference, since it is 60 to 77% of `T_att` depending on path length and therefore sets the clock for everything. trapped ions take hundreds of microseconds to milliseconds to recool and reinitialize, neutral atoms are slower still, and only a couple of platforms are anywhere near 10 µs. the results are a statement about one corner of the hardware space and are presented as if they were general.

**no code.** there is a simulator behind this, and a fairly elaborate one. event-driven, layer by layer, with contention and random service order. nothing in the paper points at a repository. a benchmarking paper whose conclusion is *it depends on your parameters* is exactly the kind that needs its tool published, because the useful artifact is not the rankings, it is the ability to re-run them with your own numbers.

## what i'd build next

**1. add fidelity and report a two-axis result.** track the state through swaps and BSMs, and report the ratio that matters: time to deliver a pair *above a target fidelity*. i suspect this reorders the switch-centric versus server-centric comparison, and possibly reverses it, because BCube pays fidelity for the latency it saves and the paper's metric only sees the saving.

**2. put reconfiguration back in and make it a scheduling problem.** measure gate requests served per switch configuration, then design a scheduler that batches non-local gates by the configuration they need. this is the actual co-design opportunity in the paper's own conclusion, and it maps almost directly onto the classical hybrid optical-electrical data center scheduling literature, where the same "the switch is slow, so amortize it" argument was worked out at length a decade ago.

**3. price it.** define a cost unit for a switch port, a BSM module and a communication qubit, and sweep the total budget. report a Pareto front instead of two contradictory rankings. **an operator never holds BSMs-per-switch fixed; they hold a purchase order fixed.** the flip in this paper is a symptom of the missing cost model, and a cost model would be maybe a week of work on top of what already exists.

**4. make multiplexing a first-class axis.** channels per fiber and communication qubits per QPU, swept alongside topology. the hypothesis worth testing is that enough multiplexing makes the topology choice nearly irrelevant, which if true is the most useful thing the simulator could tell anyone, because buying wavelengths is cheaper than rewiring a building.

**5. benchmark against a partitioner, not against random gates.** take real circuits, run a real qubit-to-QPU placement, and measure *sensitivity to placement quality* per architecture. that turns the output from a topology ranking into a design rule.

**6. do it at the logical layer.** the intro cites lattice surgery and distributed surface codes, then benchmarks bare physical circuits. under error correction a non-local logical operation is a merge that consumes a *stream* of EPR pairs at a sustained rate, so the question stops being latency per gate and becomes entanglement bandwidth per logical operation. that is a throughput problem with completely different architectural implications, and it is where this field is actually heading.

**7. publish the design rule, not just the plots.** everything in the loss half of this paper reduces to `E[hops] × (2⌈log₂ k⌉ − 1) × ℓ_2×2`. that is a closed form. print it per architecture as a single coefficient, and a reader can evaluate their own fabric against their own switch datasheet without running the simulator at all. papers that hand you a number you can multiply get used; papers that hand you a bar chart get cited.

**8. release the simulator.** see above.

## the short version

- a quantum data center moves entanglement, not packets, and entanglement is *attempted* rather than forwarded. so an extra hop does not add latency, it multiplies it: `E[T_pair] ≈ T_att × 10^(L_tot/10)`.
- hop count multiplies switch loss; radix only adds `⌈log₂ k⌉` to it. **flattening the topology is a physics optimization, not a cost one.**
- at 100 m links and 0.1 dB/km, the entire fiber plant is 0.06 dB against 1.5 dB for one radix-8 switch. **the glass is free and the switches are everything.**
- the attempt clock is the 10 µs communication-qubit reset, not the 1 µs source. the source can offer ten attempts for every one the memory can accept.
- four topologies: Fat-Tree, Clos and QFly are switch-centric with one BSM per path; BCube is server-centric and swaps at intermediate QPUs, trading one big exponent for several small ones plus a memory and a coherence budget.
- the switch-versus-server trade collapses to `ρ_ℓ = L_sw per hop / L_mem`, which is a statement about your components rather than your architecture.
- **the ranking flips with the accounting.** fixed BSMs per switch favors the deep fabrics; a fixed total budget favors QFly. at 128 QPUs those two normalizations differ by roughly 100× in relative resourcing, so the flip is a property of the experiment.
- the two sensitivity results outlast the rankings: BCube's viability is a coherence-time specification, and **deep fabrics are a bet on optical switch insertion loss getting cheaper.**
- what is missing: fidelity anywhere at all, reconfiguration delay, multiplexing as a swept parameter, a cost model, a real partitioner, the swap success probability, and the code.

what i keep coming back to is the sentence the paper ends on, that performance cannot be inferred from topology alone. that is true, and it is also the conclusion of every classical data center benchmarking paper ever written, which is why classical work stopped ranking topologies and started publishing cost-normalized design rules instead. the quantum version has a cleaner story available than the classical one ever did, because the dominant term is a single closed-form expression in hops and radix. **the loss model in the front half of this paper is more useful than the simulation results in the back half**, and i would like to see somebody build the next paper on it rather than around it.

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
.qw-sr.qw-off{opacity:.3;pointer-events:none}
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
  function setGroup(root,grp,v){
    Array.prototype.forEach.call(root.querySelectorAll('[data-grp="'+grp+'"] button'),function(b){
      var on=b.getAttribute('data-v')===String(v);
      b.classList.toggle('on',on);
      if(on) b.setAttribute('aria-pressed','true'); else b.removeAttribute('aria-pressed');
    });
  }
  function offGroup(root,grp){
    Array.prototype.forEach.call(root.querySelectorAll('[data-grp="'+grp+'"] button'),function(b){
      b.classList.remove('on'); b.removeAttribute('aria-pressed');
    });
  }
  /* time is carried in microseconds throughout */
  function fmtT(us){
    if(!isFinite(us)) return 'never';
    if(us<1000) return us.toFixed(us<100?1:0)+' µs';
    if(us<1e6) return (us/1000).toFixed(us<1e5?1:0)+' ms';
    if(us<6e7) return (us/1e6).toFixed(us<1e7?1:0)+' s';
    if(us<3.6e9) return (us/6e7).toFixed(us<6e8?1:0)+' min';
    if(us<8.64e10) return (us/3.6e9).toFixed(1)+' h';
    return (us/8.64e10).toFixed(1)+' days';
  }
  function num(x){ return x>=100?Math.round(x).toLocaleString():(x>=10?x.toFixed(1):x.toFixed(2)); }
  /* a radix-k switch is a tree of 2x2 elements */
  function stages(k){ return 2*Math.ceil(Math.log(k)/Math.LN2)-1; }

  /* ---------- 1. what a hop costs inside the building ---------- */
  (function(){
    var H=document.getElementById('qw-hop');
    if(!H) return;
    var RADIX=[4,8,12,16,24,32,48,64,96,128];
    var sn=H.querySelector('#qwh-n'), sk=H.querySelector('#qwh-k'),
        sl=H.querySelector('#qwh-l'), sb=H.querySelector('#qwh-b');
    var gBar=H.querySelector('#qwh-bar'), gP=H.querySelector('#qwh-plot');
    var PRE={ rack:{n:1,k:1}, qfly:{n:2,k:4}, pod:{n:3,k:1}, cross:{n:5,k:1} };

    function model(n,k,ell,bsm){
      var fib=0.01*(n+1);                 /* 0.1 km per link at 0.1 dB/km */
      var sw=n*stages(k)*ell;
      var tot=fib+sw+bsm;
      var p=Math.pow(10,-tot/10);
      var tatt=1+(n+1)+10;                /* source + round trip + reset, in us */
      return { fib:fib, sw:sw, bsm:bsm, tot:tot, p:p, tatt:tatt, T:tatt/p };
    }
    function bar(m){
      clear(gBar);
      var X0=8, X1=632, Y=14, Hh=24;
      var scale=(X1-X0)/Math.max(m.tot,0.5);
      var segs=[['#6ba3f0',m.fib,'fiber'],['#32c29e',m.sw,'switches'],['#f2a03d',m.bsm,'BSM']];
      var x=X0;
      segs.forEach(function(s){
        var w=s[1]*scale;
        if(w>0.4){
          gBar.appendChild(el('rect',{x:x,y:Y,width:w,height:Hh,fill:s[0],'fill-opacity':0.78,stroke:s[0],'stroke-width':1,rx:2}));
          if(w>52) gBar.appendChild(el('text',{x:x+w/2,y:Y+16,'text-anchor':'middle',fill:'#10231f','font-size':11,'font-weight':600},s[1].toFixed(2)+' dB'));
        }
        x+=w;
      });
      gBar.appendChild(el('text',{x:X0,y:Y+Hh+16,fill:'#8f8f8f','font-size':10},
        'fiber ' + m.fib.toFixed(2) + ' dB'));
      gBar.appendChild(el('text',{x:X1,y:Y+Hh+16,'text-anchor':'end',fill:'#8f8f8f','font-size':10},
        m.tot.toFixed(2) + ' dB end to end'));
    }
    function plot(k,ell,bsm,ncur){
      clear(gP);
      var X0=52,X1=624,Y0=18,Y1=164;
      var lo=1e9,hi=0,pts=[];
      for(var n=1;n<=7;n++){ var m=model(n,k,ell,bsm); pts.push([n,m.T]); if(m.T<lo)lo=m.T; if(m.T>hi)hi=m.T; }
      var l0=Math.floor(Math.log(lo)/Math.LN10), l1=Math.ceil(Math.log(hi)/Math.LN10);
      if(l1-l0<1) l1=l0+1;
      function cx(n){ return X0+(n-1)/6*(X1-X0); }
      function cy(v){ return Y1-(Math.log(v)/Math.LN10-l0)/(l1-l0)*(Y1-Y0); }
      for(var e=l0;e<=l1;e++){
        var y=cy(Math.pow(10,e));
        gP.appendChild(el('line',{x1:X0,y1:y,x2:X1,y2:y,stroke:'#2a2a2a','stroke-width':1}));
        gP.appendChild(el('text',{x:X0-8,y:y+3.5,'text-anchor':'end',fill:'#7d7d7d','font-size':9},fmtT(Math.pow(10,e))));
      }
      var d='';
      pts.forEach(function(pt,i){ d+=(i?'L':'M')+cx(pt[0]).toFixed(1)+' '+cy(pt[1]).toFixed(1); });
      gP.appendChild(el('path',{d:d,fill:'none',stroke:'#32c29e','stroke-width':2.4,'stroke-linecap':'round','stroke-linejoin':'round'}));
      pts.forEach(function(pt){
        var on=pt[0]===ncur;
        gP.appendChild(el('circle',{cx:cx(pt[0]),cy:cy(pt[1]),r:on?5:3,fill:on?'#fff':'#32c29e',stroke:'#32c29e','stroke-width':on?2.4:0}));
        gP.appendChild(el('text',{x:cx(pt[0]),y:Y1+16,'text-anchor':'middle',fill:on?'#fff':'#7d7d7d','font-size':10},pt[0]));
      });
      gP.appendChild(el('text',{x:(X0+X1)/2,y:Y1+34,'text-anchor':'middle',fill:'#8f8f8f','font-size':10},'optical switches on the path'));
    }
    function upd(){
      var n=+sn.value, k=RADIX[+sk.value], ell=+sl.value, bsm=+sb.value;
      var m=model(n,k,ell,bsm), one=model(1,k,ell,bsm);
      H.querySelector('#qwh-nv').textContent=n;
      H.querySelector('#qwh-kv').textContent=k;
      H.querySelector('#qwh-lv').textContent=ell.toFixed(2)+' dB';
      H.querySelector('#qwh-bv').textContent=bsm.toFixed(1)+' dB';
      H.querySelector('#qwh-loss').textContent=m.tot.toFixed(2)+' dB';
      H.querySelector('#qwh-p').textContent=(100*m.p).toFixed(m.p<0.01?3:1)+'%';
      H.querySelector('#qwh-att').textContent=num(1/m.p);
      H.querySelector('#qwh-t').textContent=fmtT(m.T);
      H.querySelector('#qwh-fs').textContent=(100*m.fib/m.tot).toFixed(2)+'%';
      /* a classical switch is store-and-forward: call it 0.5 us a hop, plus propagation */
      var cl=n*0.5+(n+1)*0.5, cl1=1.0;
      var ce=H.querySelector('#qwh-cl');
      ce.innerHTML=(n===1)?'this is the baseline, one switch'
        :('+'+(cl-cl1).toFixed(1)+' &micro;s classically, <b>+'+fmtT(m.T-one.T)+'</b> here');
      bar(m); plot(k,ell,bsm,n);
      var ratio=m.T/one.T;
      H.querySelector('#qwh-note').innerHTML=
        'A radix-'+k+' switch is a tree of <b>'+stages(k)+'</b> 2&times;2 stages, so it costs <b>'+(stages(k)*ell).toFixed(2)+
        ' dB</b> to cross once. '+n+' of them plus '+(0.1*(n+1)).toFixed(1)+' km of fiber comes to <b>'+m.tot.toFixed(2)+
        ' dB</b>, of which the fiber is <b>'+(100*m.fib/m.tot).toFixed(2)+'%</b>. That leaves '+(100*m.p).toFixed(m.p<0.01?3:1)+
        '% of attempts surviving, so you need about <b>'+num(1/m.p)+'</b> of them at '+m.tatt.toFixed(0)+
        ' &micro;s each: <b>'+fmtT(m.T)+'</b> per EPR pair, '+
        (n===1?'which is the floor.':('<b>'+ratio.toFixed(1)+'&times;</b> the single-switch path.'))+
        ' Note that '+(100*10/m.tatt).toFixed(0)+'% of each attempt is the communication qubit resetting, not anything optical.';
    }
    [sn,sk,sl,sb].forEach(function(s){ s.addEventListener('input',function(){ offGroup(H,'ps'); upd(); }); });
    btnGroup(H,'ps',function(v){ sn.value=PRE[v].n; sk.value=PRE[v].k; upd(); });
    upd();
  })();

  /* ---------- 2. switch-centric against server-centric ---------- */
  (function(){
    var H=document.getElementById('qw-sc');
    if(!H) return;
    var K=12, ST=stages(K), TSEG=13, par=true;
    var sn=H.querySelector('#qws-n'), sl=H.querySelector('#qws-l'),
        sm=H.querySelector('#qws-m'), sc=H.querySelector('#qws-c');
    var gP=H.querySelector('#qws-plot');
    var PRE={ paper:{n:4,l:0.3,m:3,c:200}, lossy:{n:4,l:1,m:3,c:200}, tight:{n:4,l:0.3,m:3,c:40} };

    function sw(n,ell){                       /* one long path, one BSM */
      var tot=n*ST*ell+0.01*(n+1);
      return (1+(n+1)+10)/Math.pow(10,-tot/10);
    }
    function sv(n,ell,mem,tau){               /* n segments, swap at each QPU */
      var seg=ST*ell+mem+0.02, p=Math.pow(10,-seg/10), t1=TSEG/p;
      var Hn=0; for(var i=1;i<=n;i++) Hn+=1/i;
      var T = par ? t1*Hn : n*t1;             /* time until the last segment lands */
      var W = par ? n*t1*(Hn-1) : t1*n*(n-1)/2;  /* memory-seconds spent waiting */
      var ps=Math.exp(-W/tau);
      return { T:T/ps, ps:ps, seg:seg, t1:t1 };
    }
    function plot(ell,mem,tau,ncur){
      clear(gP);
      var X0=54,X1=624,Y0=18,Y1=192;
      var lo=1e12,hi=0,A=[],B=[];
      for(var n=2;n<=7;n++){
        var a=sw(n,ell), b=sv(n,ell,mem,tau).T;
        A.push([n,a]); B.push([n,b]);
        lo=Math.min(lo,a,b); hi=Math.max(hi,a,b);
      }
      var l0=Math.floor(Math.log(lo)/Math.LN10), l1=Math.ceil(Math.log(hi)/Math.LN10);
      if(l1-l0<1) l1=l0+1;
      var clipped=(l1-l0)>6; if(clipped) l1=l0+6;   /* keep the axis readable */
      function cx(n){ return X0+(n-2)/5*(X1-X0); }
      function cy(v){ return Math.max(Y0-6, Y1-(Math.log(v)/Math.LN10-l0)/(l1-l0)*(Y1-Y0)); }
      for(var e=l0;e<=l1;e++){
        var y=cy(Math.pow(10,e));
        gP.appendChild(el('line',{x1:X0,y1:y,x2:X1,y2:y,stroke:'#2a2a2a','stroke-width':1}));
        gP.appendChild(el('text',{x:X0-8,y:y+3.5,'text-anchor':'end',fill:'#7d7d7d','font-size':9},fmtT(Math.pow(10,e))));
      }
      [[A,'#32c29e'],[B,'#f2a03d']].forEach(function(s){
        var d='';
        s[0].forEach(function(pt,i){ d+=(i?'L':'M')+cx(pt[0]).toFixed(1)+' '+cy(pt[1]).toFixed(1); });
        gP.appendChild(el('path',{d:d,fill:'none',stroke:s[1],'stroke-width':2.4,'stroke-linecap':'round','stroke-linejoin':'round'}));
        s[0].forEach(function(pt){
          var on=pt[0]===ncur;
          gP.appendChild(el('circle',{cx:cx(pt[0]),cy:cy(pt[1]),r:on?5:3,fill:on?'#fff':s[1],stroke:s[1],'stroke-width':on?2.4:0}));
        });
      });
      for(var n=2;n<=7;n++)
        gP.appendChild(el('text',{x:cx(n),y:Y1+16,'text-anchor':'middle',fill:n===ncur?'#fff':'#7d7d7d','font-size':10},n));
      gP.appendChild(el('text',{x:(X0+X1)/2,y:Y1+34,'text-anchor':'middle',fill:'#8f8f8f','font-size':10},'hops end to end'));
      if(clipped) gP.appendChild(el('text',{x:X1,y:Y0-8,'text-anchor':'end',fill:'#e2603f','font-size':9.5},'axis clipped, the orange curve runs off the top'));
    }
    function upd(){
      var n=+sn.value, ell=+sl.value, mem=+sm.value, tau=+sc.value;
      var a=sw(n,ell), b=sv(n,ell,mem,tau), rho=(ST*ell)/mem;
      H.querySelector('#qws-nv').textContent=n;
      H.querySelector('#qws-lv').textContent=ell.toFixed(2)+' dB';
      H.querySelector('#qws-mv').textContent=mem.toFixed(1)+' dB';
      H.querySelector('#qws-cv').textContent=tau.toFixed(0)+' µs';
      H.querySelector('#qws-a').textContent=fmtT(a);
      H.querySelector('#qws-b').textContent=fmtT(b.T);
      H.querySelector('#qws-r').textContent=rho.toFixed(2);
      H.querySelector('#qws-w').textContent=(100*b.ps).toFixed(b.ps<0.1?2:0)+'%';
      var ve=H.querySelector('#qws-v'), win=a<b.T, f=win?b.T/a:a/b.T;
      ve.textContent=(win?'let the light through, ':'break the path and swap, ')+f.toFixed(1)+'x';
      ve.style.color=win?'#32c29e':'#f2a03d';
      plot(ell,mem,tau,n);
      H.querySelector('#qws-note').innerHTML=
        'One switch traversal costs <b>'+(ST*ell).toFixed(2)+' dB</b> and a memory access costs <b>'+mem.toFixed(1)+
        ' dB</b>, so <b>&rho;<sub>&ell;</sub> = '+rho.toFixed(2)+'</b>. The switch-centric path puts all '+n+
        ' traversals into one exponent and gets a pair in <b>'+fmtT(a)+'</b>. The server-centric path pays only <b>'+
        b.seg.toFixed(2)+' dB</b> per segment, so a segment lands in '+fmtT(b.t1)+', but '+
        (par?'all '+n+' segments have to be alive at the same time':'the first segment has to survive while the other '+(n-1)+' are built')+
        ', and at &tau;<sub>cut</sub> = '+tau.toFixed(0)+' &micro;s only <b>'+(100*b.ps).toFixed(b.ps<0.1?2:0)+
        '%</b> of rounds make it, giving <b>'+fmtT(b.T)+'</b>. '+
        (a<b.T?'Switch-centric wins here.':'Swapping wins here.')+
        ' The model is mine: geometric attempts, a harmonic-number wait for the slowest segment, and an exponential memory penalty on the waiting. The paper reports the same crossover without publishing the closed form.';
    }
    [sn,sl,sm,sc].forEach(function(s){ s.addEventListener('input',function(){ offGroup(H,'ps'); upd(); }); });
    btnGroup(H,'pr',function(v){ par=(v==='par'); offGroup(H,'ps'); upd(); });
    btnGroup(H,'ps',function(v){
      var p=PRE[v]; sn.value=p.n; sl.value=p.l; sm.value=p.m; sc.value=p.c; upd();
    });
    upd();
  })();

  /* ---------- 3. the ranking is a function of the accounting ---------- */
  (function(){
    var H=document.getElementById('qw-bsm');
    if(!H) return;
    var ELL=0.3, CH=5, BUDGET=2500, mode='per';
    /* configurations for 128 QPUs, worked out from the standard topology
       formulas rather than lifted from the paper's own table            */
    var A=[
      {id:'fat',name:'Fat-Tree',       note:'k = 8',        sw:80, radix:8,  hops:5, links:256},
      {id:'ct', name:'Clos, tight',    note:'3 stages',     sw:56, radix:8,  hops:5, links:192},
      {id:'cc', name:'Clos, compact',  note:'leaf-spine',   sw:24, radix:16, hops:3, links:128},
      {id:'qf', name:'QFly',           note:'8 groups, full mesh', sw:8, radix:24, hops:2, links:28}
    ];
    A.forEach(function(a){
      a.dB=a.hops*stages(a.radix)*ELL+0.01*(a.hops+1);
      a.Tepr=(1+(a.hops+1)+10)/Math.pow(10,-a.dB/10);
      a.ports=a.sw*a.radix;
      a.portCost=a.ports*stages(a.radix)/5;      /* a port on a bigger switch drives more stages */
      a.slots=a.links*CH;
    });
    var sd=H.querySelector('#qwb-d'), sx=H.querySelector('#qwb-x'), scst=H.querySelector('#qwb-c');
    var gB=H.querySelector('#qwb-bars');

    function bsms(a,x,cbsm){
      if(mode==='per') return a.sw*x;
      if(mode==='tot') return 100;
      return Math.max(4,(BUDGET-a.portCost)/cbsm);
    }
    function evaluate(){
      var D=+sd.value, x=+sx.value, cbsm=+scst.value;
      return A.map(function(a){
        var B=bsms(a,x,cbsm);
        var ub=Math.max(1,D/B);                        /* one BSM held per pair */
        var uc=Math.max(1,D*Math.max(a.hops-1,1)/a.slots); /* optical channels on inter-switch links */
        return { a:a, B:B, ub:ub, uc:uc, T:a.Tepr*ub*uc };
      });
    }
    function draw(){
      var rows=evaluate(), best=Math.min.apply(null,rows.map(function(r){return r.T;}));
      var worst=Math.max.apply(null,rows.map(function(r){return r.T;}));
      var sorted=rows.slice().sort(function(p,q){ return p.T-q.T; });
      clear(gB);
      var X0=112,X1=560,YT=14,rh=46;
      rows.forEach(function(r,i){
        var y=YT+i*rh;
        var rank=sorted.indexOf(r);
        var col=rank===0?'#32c29e':(rank===rows.length-1?'#e2603f':'#d8c257');
        var w=Math.max(3,(r.T/worst)*(X1-X0));
        gB.appendChild(el('text',{x:X0-10,y:y+15,'text-anchor':'end',fill:'#e6e6e6','font-size':11.5,'font-weight':600},r.a.name));
        gB.appendChild(el('text',{x:X0-10,y:y+28,'text-anchor':'end',fill:'#7d7d7d','font-size':9.5},
          r.a.sw+' switches, '+r.a.hops+' deep'));
        gB.appendChild(el('rect',{x:X0,y:y+2,width:w,height:24,fill:col,'fill-opacity':0.75,stroke:col,'stroke-width':1,rx:3}));
        gB.appendChild(el('text',{x:X0+w+8,y:y+19,fill:col,'font-size':11},(r.T/best).toFixed(2)+'x'));
        gB.appendChild(el('text',{x:X0+6,y:y+38,fill:'#8f8f8f','font-size':9.5},
          Math.round(r.B)+' BSMs'+(r.ub>1?', '+r.ub.toFixed(1)+'x BSM queue':'')+(r.uc>1?', '+r.uc.toFixed(1)+'x channel queue':'')));
      });
      var top=sorted[0];
      H.querySelector('#qwb-w').textContent=top.a.name;
      H.querySelector('#qwb-w').style.color='#32c29e';
      H.querySelector('#qwb-s').textContent=(worst/best).toFixed(2)+'x';
      H.querySelector('#qwb-n').textContent=rows.map(function(r){return Math.round(r.B);}).join(' / ');
      var lbl=H.querySelector('#qwb-xl'), lv=H.querySelector('#qwb-xv');
      var D=+sd.value;
      H.querySelector('#qwb-dv').textContent=D;
      lv.textContent=(mode==='per')?sx.value:'—';
      scst.parentNode.classList.toggle('qw-off',mode!=='cost');
      sx.parentNode.classList.toggle('qw-off',mode!=='per');
      H.querySelector('#qwb-cv').textContent=scst.value;
      var note;
      if(mode==='per') note='Every switch gets <b>'+sx.value+'</b> BSM modules, so total capacity scales with switch count and the deep fabrics are handed <b>'+
        Math.round(rows[0].B)+'</b> against QFly&rsquo;s <b>'+Math.round(rows[3].B)+'</b>. ';
      else if(mode==='tot') note='One hundred BSM modules for the whole fabric however many switches it has, so the shallow designs concentrate them and QFly&rsquo;s eight switches get <b>12.5 each</b> against Fat-Tree&rsquo;s <b>1.25</b>. ';
      else note='A fixed budget of '+BUDGET+' port-equivalents buys ports first and BSMs with the change, at <b>'+scst.value+
        ' ports per BSM</b>. Drag that price and the ranking moves smoothly instead of flipping. ';
      note+='Underneath, the loss model is the paper&rsquo;s and the contention model is a crude <code>demand / capacity</code> queue on two resources: BSM modules, and the five optical channels on each inter-switch link. '+
        'That second one is bisection bandwidth coming back in a new costume, and it is what stops QFly running away with it once the demand rises. '+
        'BCube is not on this chart because it contends for a different thing. Configurations are my derivation at 128 QPUs, not the paper&rsquo;s table.';
      H.querySelector('#qwb-note').innerHTML=note;
    }
    [sd,sx,scst].forEach(function(s){ s.addEventListener('input',draw); });
    btnGroup(H,'md',function(v){ mode=v; draw(); });
    draw();
  })();
})();
</script>{% endraw %}