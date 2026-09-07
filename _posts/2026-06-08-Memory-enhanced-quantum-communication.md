---
layout: post
section-type: post
title: "Memory-enhanced quantum communication : store and forward for single photons"
category: 'networking'
tags: [ 'networking', 'fiber', 'quantum' ]
---

every quantum networking post on this blog has been written under the same constraint. you cannot copy a single photon, so you cannot amplify one, so every dB of loss is a dB you never get back. [the coexistence post]({% post_url 2026-03-23-coexistence-running-a-quantum-channel-and-its-clock-down-the-same-fiber %}) put the practical ceiling at about 100 km and the absolute wall somewhere past 400.

the way out has been on paper since 1998. put a box in the middle that can hold a quantum state, build the link in two short halves, and join them. that is a quantum repeater. for twenty years nobody had one that beat the plain fiber it was supposed to replace.

this is the paper where somebody did. [Bhaskar et al., "Experimental demonstration of memory-enhanced quantum communication," *Nature* 580, 60 (2020)](https://doi.org/10.1038/s41586-020-2103-5) ([arXiv](https://arxiv.org/abs/1909.01323)). one silicon atom sitting in a gap in a diamond lattice, inside a nanophotonic cavity, inside a dilution refrigerator at 20 mK. the box in the middle beat the best box you could ever build out of beam splitters, by a factor of four.

the result matters and the absolute rate is about one secret bit per minute. both of those are true, and the second one usually gets left out.

below: why the middle of an MDI link is a circuit and not a buffer, what one atom does about it, what the four-fold advantage actually rests on, and where the number goes when you replace emulated loss with glass.

## the coincidence tax

[measurement device independent QKD](https://doi.org/10.1103/PhysRevLett.108.130503) puts a third party, Charlie, between Alice and Bob. they each send him a photon carrying a randomly chosen qubit. he performs a Bell state measurement on the pair and announces which of the four Bell states he got. that announcement correlates Alice's bit with Bob's bit without revealing either. the useful property is that Charlie can be the eavesdropper and it does not matter, because every detector side channel that has ever broken a QKD system lives in Charlie's box and Charlie is not trusted.

the catch is in the word *pair*. a Bell state measurement built out of linear optics is two photons interfering on a beam splitter. they have to be there at the same time. if Alice's photon survives her half of the link with probability t and Bob's survives his with probability t, then Charlie sees a coincidence with probability t², which is the end-to-end transmission p<sub>A→B</sub>. the bound on what you can get out of that is

```
    R_max  =  p_A→B / 2      bits per channel use
```

for an unbiased basis choice, and the [repeaterless capacity](https://doi.org/10.1038/ncomms15043) sits a factor of 2.9 above it at 1.44 p<sub>A→B</sub>.

put that in networking terms. Charlie's beam splitter is a circuit switch with a zero-length buffer. both halves of the path have to be up in the same nanosecond or the transaction is lost. we stopped building networks that way a very long time ago, and the reason we stopped is the same reason it hurts here: the probability that two independent unreliable things happen simultaneously is the product, and products of small numbers get small fast.

a quantum memory turns Charlie into a store and forward node. the first photon to arrive is written into the memory and held. when the second one turns up, at any point inside the memory's coherence time, the gate happens then. now Charlie does not need both photons in the same slot. he needs each of them to show up *sometime*, and the probability of that is the sum, not the product. the [scaling goes from p to √p](https://doi.org/10.1088/1367-2630/16/4/043005), which is the same exponent a single repeater node buys you.

<div class="qw" id="qw-tax" data-qw><div class="qw-hd"><span class="qw-t">why the buffer changes the exponent</span><span class="qw-s">photons arriving at Charlie from Alice (green, above) and Bob (orange, below). without a memory only same-slot arrivals pair up. with one, the first arrival waits.</span><span class="qw-lg"><i style="background:#32c29e"></i>from Alice<i style="background:#f2a03d"></i>from Bob<i style="background:#7f8fd6"></i>completed Bell measurement</span></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">arrivals per slot <b id="qwt-dv">12%</b></span><input type="range" id="qwt-d" min="4" max="45" step="1" value="12"></label><label class="qw-sr"><span class="qw-l">memory window, slots <b id="qwt-wv">6</b></span><input type="range" id="qwt-w" min="1" max="30" step="1" value="6"></label></div><svg class="qw-svg" viewBox="0 0 640 160" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Two rows of photon arrival slots with arcs showing which pairs complete a Bell state measurement"><g id="qwt-g"></g></svg><div class="qw-out"><div class="qw-oi"><span class="qw-ok">pairs, no memory</span><span class="qw-ov" id="qwt-a">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">pairs, with memory</span><span class="qw-ov" id="qwt-b">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">gain</span><span class="qw-ov" id="qwt-r">&mdash;</span></div></div><p class="qw-n" id="qwt-note"></p><noscript><p class="qw-n">Without a memory, Charlie only records a Bell measurement when photons from Alice and Bob land in the same time slot, so the success probability is the product of the two survival probabilities. With a memory that holds the first arrival for W slots, any Alice arrival can pair with any Bob arrival inside the window, and the number of chances grows with W. In the paper the window holds up to 504 slots.</p></noscript></div>

drag the arrival rate down and the difference stops being cosmetic. at the paper's real numbers the arrivals are about one photon per three thousand slots, coincidences essentially never happen, and every single successful measurement in the whole experiment came from the buffer.

## one atom, critically coupled

the memory is a negatively charged [silicon vacancy center](https://doi.org/10.1126/science.aau4691): two carbon atoms removed from a diamond lattice and one silicon atom put back between them. that defect has an electron spin, the spin has two states, and those two states are the qubit. it sits inside a photonic crystal cavity etched into the diamond itself.

the number that matters is the cooperativity, C = 4g²/κγ, which compares how fast the atom talks to a cavity photon against how fast everything leaks. this device gets **C = 105 ± 11**, from a mode volume of 0.5(λ/n)³ and a Q of 2 × 10⁴. at that number one atom decides whether the cavity reflects at all, and that is what the protocol runs on.

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Property</th><th>Value</th><th>Why it matters</th></tr>
</thead>
<tbody>
<tr><td>Cooperativity</td><td>105 &plusmn; 11</td><td>puts the reflection contrast where it needs to be</td></tr>
<tr><td>Reflectivity, spin up / spin down</td><td>94.4% / 4.1%</td><td>this is the gate</td></tr>
<tr><td>Heralding efficiency &eta;</td><td>0.423 &plusmn; 0.004</td><td>enters the rate squared</td></tr>
<tr><td>Single-shot spin readout</td><td>30 &micro;s, F = 0.9998</td><td>the third measurement of the BSM</td></tr>
<tr><td>Spin coherence T<sub>2</sub></td><td>&gt; 200 &micro;s</td><td>how long the buffer can hold</td></tr>
<tr><td>Coherence actually used</td><td>~20 &micro;s</td><td>the microwave line heats up</td></tr>
<tr><td>Time-bin spacing &delta;t</td><td>142 ns</td><td>one slot</td></tr>
<tr><td>Qubits per memory load, N</td><td>up to 504</td><td>the buffer depth, and the whole advantage</td></tr>
<tr><td>Operating temperature</td><td>100 to 300 mK</td><td>the part that goes in the hut</td></tr>
</tbody>
</table>
</div>

the cavity is critically coupled, so on resonance an empty one swallows the light completely. put a resonant atom in it and that condition is spoiled, and the light bounces back instead. the spin state decides which of those happens. spin up reflects 94.4% of the time, spin down 4.1%. a reflection is therefore an interaction, and the phase of what comes back depends on the spin.

Alice's qubit is a time bin: one photon spread across an early pulse and a late pulse with a relative phase φ₁. the spin starts in a superposition. the early pulse reflects off one spin state, the late pulse off the other, and after the reflection the photon and the spin are entangled. Charlie then measures the reflected photon in the X basis through a delay interferometer. that measurement reveals nothing about φ₁, and it teleports φ₁ onto the spin.

Bob's photon arrives later and does the same thing. now the spin carries φ₁ + φ₂. a final X measurement of the spin gives a third bit, and the product of the three measurement outcomes m₁m₂m₃ tells you whether Alice and Bob sent the same qubit or opposite ones, without telling you what either of them sent. that is the Bell state measurement, and it is spread across three separate points in time.

**Alice's photon and Bob's photon never meet.** there is no two-photon interference anywhere in this experiment. each photon talks to the same atom at a different time. that removes an entire category of problem. there is no Hong-Ou-Mandel dip to hold, the two remote lasers are free to be as distinguishable as they like, and the long fibers need no phase stabilization, because the only interferometer that has to stay locked is 142 ns wide and lives in one rack. anyone who has tried to keep [twin-field QKD](https://doi.org/10.1038/s41586-018-0066-6) phase-locked over 500 km of buried glass will recognize what that is worth.

### the buffer depth is set by a warm wire

the enhancement over direct transmission is essentially η²N, where N is how many qubit slots fit inside one memory load. more slots, more chances to catch a second photon, bigger advantage. so N is the knob.

N is bounded by coherence time divided by slot time. the coherence time under XY8 decoupling is over 200 µs, the slot is 142 ns, and four optical pulses fit in each free-precession window, so the arithmetic says a few thousand. the experiment used 504.

the reason is that each microwave π pulse is 32 ns long and delivered through a gold coplanar waveguide on the chip, and gold at 100 mK has resistance. past about 128 π pulses the ohmic heating dephases the spin faster than the decoupling protects it. **the memory is idle ninety percent of the time because the control wire gets warm.** the paper names the fix in one sentence, which is to deliver the microwaves superconductingly.

hold onto that, because it is worth 20 dB later.

### the parts list nobody quotes

- a dilution refrigerator with a base temperature of 20 mK and a superconducting vector magnet.
- superconducting nanowire detectors mounted at the 1 K stage, dielectric-coated for 737 nm specifically.
- a 28 m fiber delay line forming the 142 ns interferometer, bolted to a weighted breadboard, packed into a **sand-filled briefcase lined with polyurethane foam**, and glued shut. the paper states this in the same tone it uses for the cooperativity, which is the best thing about it.
- every fiber in the network wrapped in foil to stop thermal polarization drift.
- that interferometer relocked for 200 ms out of roughly every 400 ms. half the wall clock time is spent holding still.
- a preselection loop that watches for the emitter spectrally diffusing or ionizing, and reruns an automatic laser relock when it does.

none of that is a criticism. it is what a working spin-photon interface costs in 2019, and the automation is good enough to run unattended for days. but if you are the person who has to put a Charlie in a hut halfway between two cities, this is the bill of materials, and the first line on it is a dilution refrigerator.

### a 20 dB circulator made of a beam splitter

photons reach the device through the **1% port of a 99:1 fiber beam splitter** and come back out through the 99% port. that is a 20 dB insertion loss on the way in, by construction, because a proper low-loss circulator or switch was not in the setup.

the paper is straightforward about it: that loss is folded into the "estimated channel loss", and it says that a real implementation should use a circulator instead. which is correct and fair for a benchmark of the memory itself. it also means that part of what is being called channel loss is a component inside Charlie's own rack, whose replacement is a catalogue part.

## what came out

two numbers come out of it. how often the node gets the answer wrong, and how much it beats the wire by.

the error rate first. averaged over random input bit strings the QBER is **E = 0.116 ± 0.002**, comfortably below the 0.146 threshold for security against individual attacks. the threshold for [unconditional security](https://doi.org/10.1103/PhysRevLett.85.441) is 0.110, and the average does not clear it. for specific periodic input patterns the QBER drops to **0.097 ± 0.006**, which does clear it, with a stated confidence of 0.986.

that gap is not physics. the qubits are generated by sending phase patterns to a modulator through a pulse amplifier with a 25 kHz low-frequency cutoff, and random patterns have a DC component the amplifier cannot reproduce. it is an RF engineering problem, it costs about two points of QBER, and it happens to sit exactly on top of the security threshold.

they also ran a CHSH test on the input correlations conditioned on the BSM outcome, getting S₊ = 2.21 ± 0.04 and S₋ = 2.19 ± 0.04. the correlations are non-classical.

now the advantage. at an emulated 88 dB of end-to-end loss and N = 504, the sifted key rate is **78.4 ± 0.7 times** what an ideal linear-optics MDI-QKD system could do at the same loss. after error correction and privacy amplification, at the node's best operating point of N ≈ 124 and about 69 dB, the distilled secret key rate is **4.1 ± 0.5 times** the ideal direct-transmission bound, and **1.43 times** the repeaterless capacity, with 99.2% confidence.

<div class="qw" id="qw-rate" data-qw><div class="qw-hd"><span class="qw-t">secret key against channel loss</span><span class="qw-s">bits per channel use. calibrated to three numbers the paper states: 78.4&times; sifted at 88 dB, 4.1&times; secret at 69 dB, 1.43&times; the repeaterless capacity.</span><span class="qw-lg"><i style="background:#7f8fd6"></i>repeaterless capacity, 1.44p<i style="background:#8f8f8f"></i>ideal direct MDI-QKD, p/2<i style="background:#32c29e"></i>memory node</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">how much memory time the node can use</span><span class="qw-b" data-grp="cap"><button type="button" data-v="504" class="on" aria-pressed="true">20 &micro;s, N &le; 504</button><button type="button" data-v="5040">200 &micro;s, N &le; 5040</button></span></div><div class="qw-g"><span class="qw-l">count the channel as</span><span class="qw-b" data-grp="nrm"><button type="button" data-v="1" class="on" aria-pressed="true">one use per pair</button><button type="button" data-v="0.5">one use per photon</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">heralding efficiency <span class="qw-gk">&eta;</span> <b id="qwr-ev">0.42</b></span><input type="range" id="qwr-e" min="5" max="90" step="1" value="42"></label><label class="qw-sr"><span class="qw-l">end-to-end loss <b id="qwr-lv">69 dB</b></span><input type="range" id="qwr-l" min="20" max="120" step="1" value="69"></label></div><svg class="qw-svg" viewBox="0 0 640 340" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Log-log plot of secret key per channel use against end to end channel loss"><g id="qwr-grid"></g><g id="qwr-curves"></g><g id="qwr-cur"></g></svg><div class="qw-out"><div class="qw-oi"><span class="qw-ok">equivalent fiber</span><span class="qw-ov" id="qwr-km">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">slots in use, N</span><span class="qw-ov" id="qwr-n">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">modelled QBER</span><span class="qw-ov" id="qwr-q">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">vs ideal direct</span><span class="qw-ov" id="qwr-g">&mdash;</span></div><div class="qw-oi qw-grow"><span class="qw-ok">vs repeaterless capacity</span><span class="qw-ov qw-txt" id="qwr-p">&mdash;</span></div></div><p class="qw-n" id="qwr-note"></p><noscript><p class="qw-n">At 69 dB the memory node produces about four times the ideal direct-transmission MDI-QKD rate and 1.43 times the repeaterless capacity. The advantage grows as the square root of transmission only while the node can keep adding time-bin slots; once N hits its ceiling at 504 slots, around 88 dB, the curve bends back to being linear in transmission and parallel to the direct-transmission line. Raising the usable memory time from 20 to 200 microseconds moves that corner out by roughly 20 dB.</p></noscript></div>

the shape of that plot is the argument. the memory curve has half the slope of the other two, because as the loss rises the node is allowed to spread the same average photon number over more slots, and the enhancement η²N grows as fast as the transmission falls. that is where the √p comes from, and it is worth being precise about the condition. **the square root lasts exactly as long as the node can keep filling the buffer, and stops when it cannot.**

and there is a corner in it. once N hits 504 the node cannot add slots, the enhancement freezes, and the curve goes back to being parallel to plain direct transmission. that corner lands at about 88 dB, which is exactly where the paper's highest-loss data point sits. they ran the experiment right up to the edge of the buffer and stopped, and the plot shows why there was nothing past it.

switch the memory-time button to 200 µs and the corner moves out by roughly 20 dB, which is 100 km. that is the superconducting microwave line from two sections ago. it is the largest single number available anywhere in this paper and it costs a fabrication change.

### the accounting question

the second button on that widget is the part i want to argue with, and to be clear the paper reports both sides of it honestly in its supplementary Table S4. it just does not decide.

in ordinary MDI-QKD Alice and Bob transmit simultaneously, so one "channel use" naturally means one pulse from each of them. with an asynchronous BSM they never transmit at the same time. Alice uses the channel, then later Bob does. so is that one channel use or two?

count it the first way, matching how synchronous MDI-QKD is scored, and the node delivers 2.37 × 10⁻⁷ bits per use and sits at **1.43 times** the repeaterless capacity. count each one-way transmission separately, which the paper calls channel occupancy, and the same data gives 1.19 × 10⁻⁷ and **0.71 times** the capacity. the same experiment either beats the bound or does not, depending on a convention.

i do not think the paper picked wrong. the per-channel-use convention is the one that makes the comparison to direct-transmission MDI-QKD apples to apples, which is the comparison the paper is actually making, and the abstract only ever claims the four-fold advantage over MDI-QKD. the repeaterless-capacity claim is the one that leans on the convention, and it is the one that got repeated.

there is also a reading where the conservative number is the right one and the node is still fine, because the half of the channel Alice is not using is genuinely idle and could be carrying something else. the paper says so: it could be routed to a second BSM device. nobody has built that, and until somebody does, the spare half is an accounting asset rather than a real one.

## the number nobody printed

the paper never prints a rate in bits per second. i went looking twice.

everything is normalized. per channel use, per channel occupancy, ratios to bounds. the closest thing to an absolute figure is a line in the supplement saying that at the operating point the node produces **BSM successes at a rate of roughly 0.1 Hz**.

work forward from that. half the successes survive basis sifting. at an 11% error rate roughly 40% of what is left survives error correction and privacy amplification. that is about 0.02 secret bits per second, so **call it a bit a minute**. the paper has the other ingredient as well, an effective clock rate of 1.2 MHz measured across whole multi-day runs, and it never multiplies the two together and prints the answer.

for scale, in 2018 a Geneva group ran plain decoy-state BB84 over [421 km of ultralow-loss fiber](https://doi.org/10.1103/PhysRevLett.121.190502) and got 6.5 bits per second at 405 km, at a comparable total loss. that is a factor of three hundred more key at the same loss, using no memory, no cryogenics and no atom, because the clock was 2.5 GHz instead of about 1 MHz.

the comparison is unfair, and not in the direction it first looks. that system trusts its detectors and this one does not, and one of them scales past a single intermediate node and the other never will. but it is the right frame for reading the four-fold advantage. **the memory node wins per channel use and loses by two and a half orders of magnitude per second, and the whole gap is clock rate.** the exponent is the result worth having. the prefactor is what an operator actually buys, and right now the prefactor is dreadful.

## what 350 km means

the paper says its best operating point, about 69 dB, corresponds to roughly 350 km of telecommunications fiber. that arithmetic is 0.2 dB/km, which is right for the C band.

the silicon vacancy does not emit in the C band. it emits at **737 nm**, deep red, where fused silica scatters about fifteen times harder and where standard singlemode fiber is not singlemode at all, because the cutoff is up at 1260 nm. call it 3 dB/km in a small-core fiber built for that window. at 3 dB/km, 69 dB of budget is 23 km between Alice and Bob, not 350.

so the number in the paper is a promissory note against quantum frequency conversion, which the paper lists as required future work. that work now exists, and it is not free: every conversion is a nonlinear crystal with an insertion loss and a noise floor.

<div class="qw" id="qw-band" data-qw><div class="qw-hd"><span class="qw-t">what the loss budget buys, by wavelength</span><span class="qw-s">69 dB of end-to-end budget, split evenly, one frequency conversion per side at Charlie&rsquo;s input.</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">fiber</span><span class="qw-b" data-grp="pl"><button type="button" data-v="spec" class="on" aria-pressed="true">data sheet</button><button type="button" data-v="depl">deployed plant</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">conversion efficiency, each side <b id="qwb-ev">50%</b></span><input type="range" id="qwb-e" min="5" max="95" step="1" value="50"></label><label class="qw-sr"><span class="qw-l">loss budget <b id="qwb-lv">69 dB</b></span><input type="range" id="qwb-l" min="40" max="100" step="1" value="69"></label></div><svg class="qw-svg" viewBox="0 0 640 210" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Bar chart of reachable Alice to Bob distance for three operating wavelengths"><g id="qwb-g"></g></svg><div class="qw-out"><div class="qw-oi"><span class="qw-ok">737 nm, no conversion</span><span class="qw-ov" id="qwb-a">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">1350 nm, one stage</span><span class="qw-ov" id="qwb-b">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">1550 nm, two stages</span><span class="qw-ov" id="qwb-c">&mdash;</span></div></div><p class="qw-n" id="qwb-note"></p><noscript><p class="qw-n">Spending a 69 dB end-to-end budget at the emitter&rsquo;s native 737 nm, where fiber loses about 3 dB/km, separates Alice and Bob by roughly 23 km. Converting to 1350 nm at 0.35 dB/km with a 50% efficient converter on each side gives about 180 km. Converting all the way to 1550 nm through two stages gives about 285 km on data-sheet fiber. The paper&rsquo;s 350 km figure assumes 0.2 dB/km and no conversion loss at all.</p></noscript></div>

the deployed-plant button is the one worth pressing. when this group later ran a 35 km loop through Cambridge, Somerville, Watertown and Boston, the measured loss was 17 dB, which is 0.49 dB/km at 1350 nm against a data sheet figure of about 0.35. splices, patch panels, connectors, and a route that does not go in a straight line. anyone who has walked a real span knows that ratio, and it is the difference between 180 km and 130 km here.

## what's missing

the paper is careful and most of what follows is something it names and does not solve, which is fair for a first demonstration. some of it is the gap between what was measured and what the result is usually described as.

**Alice and Bob were the same laser.** one narrow-linewidth Ti:Sapphire generated both parties' photons, attenuated to weak coherent pulses and phase-modulated by one AWG. that is the right way to benchmark a device. it also means the experiment contains no independent parties, shares one phase reference between them, and never tests what happens when Alice's frame and Bob's frame drift apart. the paper lists "truly independent, distant communicating parties" first among the things still needed. five years on, that specific composition, two remote transmitters through a memory node with a key at the end, still has not been published for this platform.

**the loss was emulated, not incurred.** channel transmission was set by dialling the mean photon number down, not by putting fiber in the path. for benchmarking a memory that is legitimate, and for this protocol the propagation delay is common to both arms, so the memory never has to cover it. what the attenuator removes is everything else: chromatic dispersion, polarization drift, background from anything else sharing the glass, and the calibration problem of getting Alice's and Bob's 142 ns encoders to agree on what the X basis means from 175 km apart. it also hides the constraint that governs the repeater this is a step towards, where every node has to hold entanglement for at least the classical round trip to its neighbour. at 350 km that round trip is 3.4 ms and the coherence time is 0.2.

**no decoy states.** weak coherent pulses without decoy states are vulnerable to photon number splitting, so this is not yet a secure protocol against a general attacker, only a benchmark of the device. the paper says so and lists decoy states, biased bases and finite-key analysis as required. all three would reduce the numbers.

**the QBER that clears the security threshold came from non-random inputs.** 0.116 with random bit strings, 0.110 threshold, 0.097 with tailored periodic patterns. the paper explains exactly why and blames its pulse amplifier. it is still true that the headline security claim rests on the input pattern that the RF chain handles best.

**the enhancement is against an ideal, and the node is not.** 4.1 times a theoretical bound assuming perfect sources and detectors is the right comparison for the physics and the wrong one for procurement. against a real deployed system at the same loss, this node is three hundred times slower.

**the 20 dB beam splitter is inside the box.** loss you own and loss the channel imposes are different line items, and folding one into the other makes the node look worse and the channel look longer at the same time. an efficiency budget that separated them would be more useful to everyone.

**there is one memory and one wavelength.** every photon that arrives while the memory is occupied is discarded. a multiplexed node with several emitters, or several spectral or temporal modes per emitter, does not have that problem, and multiplexing is the standard answer to exactly this bottleneck in every repeater proposal that has ever been written down.

## what i'd build next

**1. two lasers.** the cheapest missing experiment in the paper. put an independent transmitter at each end with its own reference, run the same protocol, and report what the QBER does over six hours as the two frames drift. because there is no two-photon interference, this should be much easier here than in twin-field QKD, and that is a claim worth actually testing rather than asserting.

**2. superconducting microwave delivery.** the memory can hold coherence for 200 µs and is being used for 20 because a gold wire warms up. recovering that factor of ten multiplies the enhancement by ten and moves the point where the √p scaling stops out by about 20 dB. nothing else in the paper offers that much for that little.

**3. report bits per second at a stated distance, and publish the duty cycle.** half the wall clock went into relocking an interferometer and an unknown further fraction into spectral relocks. those are real availability numbers and they belong next to the key rate. an operator reads a rate as an average over a month, not over the good minutes.

**4. separate node loss from channel loss in the budget.** publish η for the device with the injection optics excluded, and publish the injection loss as its own line. then two nodes built on different platforms can be compared, and the 20 dB beam splitter stops silently inflating the apparent channel.

**5. put a clock in it.** every detector in this experiment was in one lab on one time tagger, so synchronization never appears. the moment Alice is 175 km away, the 142 ns time-bin structure has to be recovered at the far end to a small fraction of a bin, over a fiber whose length moves with temperature. that is the [coexistence problem]({% post_url 2026-03-23-coexistence-running-a-quantum-channel-and-its-clock-down-the-same-fiber %}) again, and here it is stricter, because the memory node's slot grid is 142 ns and its acceptance window is a few nanoseconds.

**6. write the transmitter spec.** what does an interoperable Alice have to do? emit time-bin qubits on a 142 ns grid, phase-locked to a shared reference, at a wavelength that lands on this particular emitter's line after conversion, with a frame counter so she can track the parity flips caused by Charlie's π pulses. all of that is implied by the protocol and none of it is written down anywhere as an interface. a repeater is only useful if more than one group can build the thing that talks to it.

**7. multiplex the node.** one emitter serving one link at a time is a single-channel line card. the same fridge could hold many emitters, and the same emitter could serve several spectral modes. the entanglement distribution work in the [flex-grid posts]({% post_url 2026-05-29-eight-users-on-one-chip-maximizing-the-source-side-of-a-flex-grid-quanntum-network %}) has spent five years learning to slice one source among many users. nobody has connected that literature to this one, and the allocation question is the same question.

## what happened since

most of the missing pieces exist now, and a good fraction of them came from the same group.

the platform grew a second qubit. [Stas et al.](https://doi.org/10.1126/science.add9771) (Science 2022) turned the node into a two-qubit register using the ²⁹Si nuclear spin, with integrated error detection, and did it in a highly strained device that works at 1.5 K rather than in a dilution refrigerator. [Knall et al.](https://doi.org/10.1103/PhysRevLett.129.053603) (PRL 2022) made the same system into a deterministic single photon source with shaped pulses, which removes the weak-coherent-state assumption and the decoy state requirement with it.

the wavelength problem got solved. [Bersin et al.](https://doi.org/10.1103/PRXQuantum.5.010303) (PRX Quantum 2024) built bidirectional conversion between 737 nm and 1350 nm and used it to store a photon sent 50 km over deployed fiber from MIT Lincoln Laboratory onto an SiV at Harvard, at 87% storage fidelity. the same group's [Boston-area testbed paper](https://doi.org/10.1103/PhysRevApplied.21.014024) documents the plant.

and then the composed experiment, for entanglement rather than key. [Knaut et al.](https://doi.org/10.1038/s41586-024-07252-z) (Nature 2024) entangled two SiV nodes in separate laboratories through 40 km of spooled fiber and through the 35 km deployed loop, holding the entanglement in nuclear spins for over a second. two nodes, real glass, real urban fiber. still no secret key at the end of it, and still not two independent Alices through a memory in the middle.

meanwhile other platforms got to the repeater milestone first in other ways. [Langenfeld et al.](https://doi.org/10.1103/PhysRevLett.126.230506) (PRL 2021) built a repeater node from two atoms in an optical cavity, doubled the effective attenuation length and came in under the 11% threshold, one year after this paper. [Krutyanskiy et al.](https://doi.org/10.1103/PhysRevLett.130.213601) (PRL 2023) did a telecom-wavelength repeater node on a trapped-ion processor.

the biggest one is recent. [Liu et al.](https://doi.org/10.1038/s41586-026-10177-4) (Nature 2026) entangled two trapped-ion memories through 10 km of spooled telecom fiber and got the entanglement to survive **550 ms against a 450 ms average establishment time**. that inequality is the whole ballgame for multi-stage repeaters: until the stored link outlives the time it takes to build the next one, chaining segments does not work at all. they also ran device-independent QKD over the same link. and [Zhu et al.](https://doi.org/10.1038/s41566-026-01911-5) (Nature Photonics 2026) put a multiplexed repeater on 14.5 km of metropolitan fiber at 78.6% fidelity, more than a hundred times faster than earlier metropolitan relays. that is the multiplexing fix arriving on real fiber.

and the competition never stopped. [twin-field QKD](https://doi.org/10.1103/PhysRevLett.130.210801) reached 1002 km of fiber in 2023 with the same √p scaling, no memory, no cryostat and no atom, at the price of stabilizing the phase of two lasers a thousand kilometres apart. for point-to-point links that is the thing to beat, and it is winning. what it cannot do is chain. a twin-field link has exactly one middle, and the memory node is the only one of the two that turns into a repeater with more nodes bolted on.

## the short version

- an MDI-QKD link needs both photons at Charlie in the same instant, so the rate goes as the product of the two half-link transmissions. a memory turns that into store and forward, and the rate goes as the square root.
- the memory is one silicon vacancy in a diamond nanocavity with cooperativity 105. spin up reflects, spin down does not, and that reflection is the gate.
- **the two photons never interfere with each other**, so there is no indistinguishability requirement between remote sources and no long-baseline phase stabilization. the only interferometer that matters is 142 ns wide.
- the enhancement is η²N, where N is how many slots fit in one memory load. η = 0.423 and N up to 504, giving 78.4 times the ideal direct-transmission sifted rate at 88 dB and 4.1 times the secret rate at 69 dB.
- **the square-root scaling lasts only while you can keep adding slots.** N tops out at 504 and the curve bends back to linear at about 88 dB, which is exactly where the paper's last data point is.
- N is capped at a tenth of the coherence time by resistive heating in a gold microwave line. fixing that is worth 20 dB.
- beating the repeaterless capacity depends on the normalization: 1.43 times if a pair of one-way transmissions counts as one channel use, 0.71 times if each counts separately. the paper reports both.
- the paper never states bits per second. worked out from its own BSM rate it is about **one secret bit per minute**, against 6.5 bits per second for plain BB84 at comparable loss in 2018. the gap is entirely clock rate.
- 69 dB is 350 km at 0.2 dB/km and 23 km at the emitter's native 737 nm. the difference is frequency conversion, which now exists and costs a few dB per side.
- Alice and Bob were one laser, the loss was emulated with an attenuator, there were no decoy states, and 20 dB of the "channel" was a beam splitter inside Charlie's rack.

almost everything on the list above is a prefactor, and prefactors get fixed by people with fab time and soldering irons. the exponent is the hard part, and this is the paper where it moved inside a real device for the first time. that the device then produced one bit a minute is not much of an argument against it. the first transatlantic telegraph cable carried Queen Victoria's ninety-eight words to Washington in sixteen and a half hours, which is about ten minutes a word.

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
  var NS = 'http://www.w3.org/2000/svg';
  function el(tag, attrs, txt){
    var e = document.createElementNS(NS, tag);
    for (var k in attrs) e.setAttribute(k, attrs[k]);
    if (txt != null) e.textContent = txt;
    return e;
  }
  function clear(g){ while (g.firstChild) g.removeChild(g.firstChild); }
  function log10(x){ return Math.log(x) / Math.LN10; }
  function log2(x){ return Math.log(x) / Math.LN2; }
  function H2(x){
    if (x <= 0) return 0;
    if (x >= 1) return 1;
    return -x * log2(x) - (1 - x) * log2(1 - x);
  }
  function sci(v){
    if (!(v > 0)) return '0';
    var e = Math.floor(log10(v)), m = v / Math.pow(10, e);
    return m.toFixed(1) + ' &times; 10<sup>' + e + '</sup>';
  }
  // Stop Hammer.js (bound to #post for swipe navigation) from seeing drags
  // that belong to these widgets. Bubble-phase stopPropagation on the
  // gesture-start events is enough; never preventDefault, or range inputs die.
  function shield(node){
    ['pointerdown','touchstart','mousedown','touchmove'].forEach(function(ev){
      node.addEventListener(ev, function(e){ e.stopPropagation(); }, {passive:true});
    });
  }
  Array.prototype.forEach.call(document.querySelectorAll('[data-qw]'), shield);
  function btnGroup(root, grp, cb){
    var wrap = root.querySelector('[data-grp="' + grp + '"]');
    if (!wrap) return;
    wrap.addEventListener('click', function(e){
      var b = e.target.closest ? e.target.closest('button') : null;
      if (!b || !wrap.contains(b)) return;
      Array.prototype.forEach.call(wrap.children, function(x){
        x.classList.remove('on'); x.removeAttribute('aria-pressed');
      });
      b.classList.add('on'); b.setAttribute('aria-pressed', 'true');
      cb(b.getAttribute('data-v'));
    });
  }

  /* ---------- 1. the coincidence tax ---------- */
  (function(){
    var H = document.getElementById('qw-tax');
    if (!H) return;
    var SLOTS = 40;
    // Two fixed pseudo-random streams, so the picture only grows as the
    // density slider rises instead of reshuffling on every input event.
    function stream(seed){
      var v = seed % 2147483647, o = [];
      for (var i = 0; i < SLOTS; i++){ v = (v * 48271) % 2147483647; o.push(v / 2147483647); }
      return o;
    }
    var ra = stream(1015), rb = stream(7021);
    var sd = H.querySelector('#qwt-d'), sw = H.querySelector('#qwt-w'),
        g = H.querySelector('#qwt-g'), note = H.querySelector('#qwt-note');

    function draw(){
      var d = +sd.value / 100, W = +sw.value, i;
      H.querySelector('#qwt-dv').textContent = (+sd.value) + '%';
      H.querySelector('#qwt-wv').textContent = W;

      var A = [], B = [];
      for (i = 0; i < SLOTS; i++){ A.push(ra[i] < d); B.push(rb[i] < d); }

      // No memory: a Bell measurement only happens on a same-slot pair.
      var same = [];
      for (i = 0; i < SLOTS; i++) if (A[i] && B[i]) same.push(i);

      // With memory: hold the first arrival, complete on the next arrival
      // from the other party inside the window, then reset.
      var held = -1, from = 0, pairs = [];
      for (i = 0; i < SLOTS; i++){
        if (held >= 0 && i - held > W) held = -1;
        var a = A[i], b = B[i];
        if (held >= 0 && ((from === 1 && b) || (from === 2 && a))){
          pairs.push([held, i]); held = -1; continue;
        }
        if (a && b && held < 0){ pairs.push([i, i]); continue; }
        if (held < 0 && (a || b)){ held = i; from = a ? 1 : 2; }
      }

      clear(g);
      var x0 = 26, x1 = 620, yA = 44, yB = 108, mid = 76;
      var step = (x1 - x0) / (SLOTS - 1);
      g.appendChild(el('line', {x1:x0-8, y1:mid, x2:x1+8, y2:mid, stroke:'#2e2e2e', 'stroke-width':1}));
      g.appendChild(el('text', {x:x0-14, y:yA+4, fill:'#8f8f8f', 'font-size':10, 'text-anchor':'end'}, 'A'));
      g.appendChild(el('text', {x:x0-14, y:yB+4, fill:'#8f8f8f', 'font-size':10, 'text-anchor':'end'}, 'B'));
      for (i = 0; i < SLOTS; i++){
        var x = x0 + i * step;
        g.appendChild(el('line', {x1:x, y1:mid-4, x2:x, y2:mid+4, stroke:'#2e2e2e', 'stroke-width':1}));
        if (A[i]) g.appendChild(el('circle', {cx:x, cy:yA, r:4, fill:'#32c29e'}));
        if (B[i]) g.appendChild(el('circle', {cx:x, cy:yB, r:4, fill:'#f2a03d'}));
      }
      pairs.forEach(function(p){
        var xa = x0 + p[0] * step, xb = x0 + p[1] * step;
        if (p[0] === p[1]){
          g.appendChild(el('circle', {cx:xa, cy:mid, r:5, fill:'none', stroke:'#7f8fd6', 'stroke-width':1.8}));
          return;
        }
        g.appendChild(el('path', {
          d:'M' + xa + ' ' + mid + ' Q' + ((xa+xb)/2) + ' ' + (mid+32) + ' ' + xb + ' ' + mid,
          fill:'none', stroke:'#7f8fd6', 'stroke-width':1.8
        }));
      });
      same.forEach(function(k){
        var x = x0 + k * step;
        g.appendChild(el('rect', {x:x-7, y:yA-14, width:14, height:yB-yA+28, rx:3,
          fill:'none', stroke:'#8f8f8f', 'stroke-width':1.2, 'stroke-dasharray':'3 3'}));
      });
      g.appendChild(el('text', {x:x0, y:152, fill:'#6f6f6f', 'font-size':10},
        SLOTS + ' slots of 142 ns, ' + (SLOTS * 0.142).toFixed(1) + ' \u00b5s of memory time'));

      H.querySelector('#qwt-a').textContent = same.length;
      H.querySelector('#qwt-b').textContent = pairs.length;
      H.querySelector('#qwt-r').textContent = same.length
        ? ('\u00d7' + (pairs.length / same.length).toFixed(1))
        : (pairs.length ? '\u221e' : '\u2014');

      var msg;
      if (!same.length && pairs.length){
        msg = 'nothing landed in the same slot, so a Charlie built from beam splitters records <b>nothing at all</b>, while the buffered node completes ' + pairs.length + '. this is the regime the experiment actually runs in.';
      } else if (!pairs.length){
        msg = 'nothing arrives. drag the arrival rate up.';
      } else {
        msg = 'at this arrival rate coincidences are still common enough to see, so both approaches work and the memory is only a multiplier. push the rate below about 15% and the dashed boxes disappear while the arcs do not.';
      }
      note.innerHTML = msg + ' the real experiment ran at roughly one arrival per 3000 slots, with a window of up to 504.';
    }
    sd.addEventListener('input', draw);
    sw.addEventListener('input', draw);
    draw();
  })();

  /* ---------- 2. secret key against loss ----------
     Calibrated to three numbers in the paper: the sifted enhancement of 78.4
     over ideal direct-transmission MDI-QKD at 88 dB with N = 504, the secret
     enhancement of 4.1 at 69 dB with N ~ 124, and the resulting 1.43 times
     the repeaterless capacity. QBER is fitted to rise with N, which is what
     Fig. S5b shows happening through microwave heating.                    */
  (function(){
    var H = document.getElementById('qw-rate');
    if (!H) return;
    var NM = 0.02, KCAL = 0.869, RCAL = 0.461;
    var cap = 504, nrm = 1;
    var se = H.querySelector('#qwr-e'), sl = H.querySelector('#qwr-l');

    function slots(p){ return Math.max(1, Math.min(cap, NM / Math.sqrt(p))); }
    function qber(N){ return 0.105 + 6e-5 * (504 / cap) * N; }
    function frac(E){
      var r = RCAL * (0.146 - E) / 0.036;
      return Math.max(0, Math.min(1 - H2(E), r));
    }
    function memRate(L, eta){
      var p = Math.pow(10, -L / 10), N = slots(p);
      return nrm * frac(qber(N)) * (p / 2) * KCAL * eta * eta * N;
    }
    function plob(L){ var p = Math.pow(10, -L / 10); return -log2(1 - p); }
    function direct(L){ return Math.pow(10, -L / 10) / 2; }

    var X0 = 46, X1 = 626, Y0 = 16, Y1 = 292, L0 = 20, L1 = 120, E0 = -19, E1 = -3;
    function px(L){ return X0 + (L - L0) / (L1 - L0) * (X1 - X0); }
    function py(v){
      var e = Math.max(E0, Math.min(E1, log10(Math.max(v, 1e-30))));
      return Y1 - (e - E0) / (E1 - E0) * (Y1 - Y0);
    }
    function path(fn, eta){
      var d = '';
      for (var L = L0; L <= L1 + 0.001; L += 1){
        var v = fn(L, eta);
        d += (d ? 'L' : 'M') + px(L).toFixed(1) + ' ' + py(v).toFixed(1);
      }
      return d;
    }

    function draw(){
      var eta = +se.value / 100, L = +sl.value;
      H.querySelector('#qwr-ev').textContent = eta.toFixed(2);
      H.querySelector('#qwr-lv').textContent = L + ' dB';

      var grid = H.querySelector('#qwr-grid'); clear(grid);
      for (var e = E0; e <= E1; e += 2){
        var y = py(Math.pow(10, e));
        grid.appendChild(el('line', {x1:X0, y1:y, x2:X1, y2:y, stroke:'#262626', 'stroke-width':1}));
        grid.appendChild(el('text', {x:X0-6, y:y+3.5, fill:'#6f6f6f', 'font-size':9.5, 'text-anchor':'end'}, '1e' + e));
      }
      for (var L2 = L0; L2 <= L1; L2 += 20){
        var x = px(L2);
        grid.appendChild(el('line', {x1:x, y1:Y0, x2:x, y2:Y1, stroke:'#262626', 'stroke-width':1}));
        grid.appendChild(el('text', {x:x, y:Y1+15, fill:'#6f6f6f', 'font-size':9.5, 'text-anchor':'middle'}, L2 + ' dB'));
        grid.appendChild(el('text', {x:x, y:Y1+28, fill:'#565656', 'font-size':9, 'text-anchor':'middle'}, (L2/0.2) + ' km'));
      }
      grid.appendChild(el('text', {x:X0-6, y:Y0-4, fill:'#6f6f6f', 'font-size':9.5, 'text-anchor':'end'}, 'bits/use'));

      var cur = H.querySelector('#qwr-curves'); clear(cur);
      cur.appendChild(el('path', {d:path(function(l){ return plob(l); }), fill:'none', stroke:'#7f8fd6', 'stroke-width':1.8}));
      cur.appendChild(el('path', {d:path(function(l){ return direct(l); }), fill:'none', stroke:'#8f8f8f', 'stroke-width':1.8, 'stroke-dasharray':'5 4'}));
      cur.appendChild(el('path', {d:path(memRate, eta), fill:'none', stroke:'#32c29e', 'stroke-width':2.2}));
      [[69, direct(69) * 4.13], [88, direct(88) * 78.4 * frac(qber(504))]].forEach(function(pt){
        cur.appendChild(el('circle', {cx:px(pt[0]), cy:py(pt[1]), r:3.5, fill:'none', stroke:'#e8e8e8', 'stroke-width':1.4}));
      });

      var c = H.querySelector('#qwr-cur'); clear(c);
      var xc = px(L);
      c.appendChild(el('line', {x1:xc, y1:Y0, x2:xc, y2:Y1, stroke:'#4a4a4a', 'stroke-width':1, 'stroke-dasharray':'2 3'}));
      c.appendChild(el('circle', {cx:xc, cy:py(memRate(L, eta)), r:4, fill:'#32c29e'}));

      var p = Math.pow(10, -L / 10), N = slots(p), E = qber(N), R = memRate(L, eta);
      H.querySelector('#qwr-km').textContent = (L / 0.2).toFixed(0) + ' km';
      H.querySelector('#qwr-n').textContent = N.toFixed(0);
      H.querySelector('#qwr-q').textContent = (100 * E).toFixed(1) + '%';
      H.querySelector('#qwr-g').innerHTML = R > 0 ? ('\u00d7' + (R / direct(L)).toFixed(1)) : 'no key';
      var ratio = R / plob(L);
      H.querySelector('#qwr-p').textContent = R > 0
        ? ('\u00d7' + ratio.toFixed(2) + (ratio > 1 ? ', above' : ', below'))
        : 'no key';

      var corner = 20 * log10(NM / cap) * -1;
      var msg;
      if (R <= 0){
        msg = 'the modelled error rate has passed the threshold, so there is no key here at any block size.';
      } else if (N >= cap - 0.5){
        msg = 'the buffer is <b>full</b> at ' + cap + ' slots. past about ' + corner.toFixed(0) + ' dB the node cannot add chances any faster than the channel takes them away, so the green curve runs parallel to the grey one and the advantage stops growing.';
      } else {
        msg = 'the buffer still has room, so every extra dB of loss is answered with more slots and the green curve falls at half the slope of the other two. that is the whole square-root result.';
      }
      if (L > 100) msg += ' the model carries no detector dark counts, so this far right the curve is optimistic for everybody on it.';
      note.innerHTML = msg + ' circles are the two operating points the paper reports.';
    }
    var note = H.querySelector('#qwr-note');
    btnGroup(H, 'cap', function(v){ cap = +v; draw(); });
    btnGroup(H, 'nrm', function(v){ nrm = +v; draw(); });
    se.addEventListener('input', draw);
    sl.addEventListener('input', draw);
    draw();
  })();

  /* ---------- 3. what the budget buys, by wavelength ---------- */
  (function(){
    var H = document.getElementById('qw-band');
    if (!H) return;
    var BANDS = [
      {n:'737 nm, native',    spec:3.00, depl:3.40, st:0, k:'a'},
      {n:'1350 nm, one stage', spec:0.35, depl:0.49, st:1, k:'b'},
      {n:'1550 nm, two stage', spec:0.20, depl:0.25, st:2, k:'c'}
    ];
    var plant = 'spec';
    var se = H.querySelector('#qwb-e'), sl = H.querySelector('#qwb-l'),
        g = H.querySelector('#qwb-g'), note = H.querySelector('#qwb-note');

    function draw(){
      var eff = +se.value / 100, budget = +sl.value;
      H.querySelector('#qwb-ev').textContent = (+se.value) + '%';
      H.querySelector('#qwb-lv').textContent = budget + ' dB';
      var half = budget / 2, conv = -10 * log10(eff);

      var res = BANDS.map(function(b){
        var a = plant === 'spec' ? b.spec : b.depl;
        var left = half - b.st * conv;
        return {b:b, a:a, dB:left, km: left > 0 ? 2 * left / a : 0};
      });
      var top = Math.max(400, Math.ceil(Math.max.apply(null, res.map(function(r){ return r.km; })) / 50) * 50);

      clear(g);
      var X0 = 120, X1 = 610, Y = 30, GAP = 52;
      for (var t = 0; t <= top; t += top / 4){
        var x = X0 + t / top * (X1 - X0);
        g.appendChild(el('line', {x1:x, y1:14, x2:x, y2:Y + 2 * GAP + 22, stroke:'#262626', 'stroke-width':1}));
        g.appendChild(el('text', {x:x, y:Y + 2 * GAP + 36, fill:'#6f6f6f', 'font-size':9.5, 'text-anchor':'middle'}, t.toFixed(0)));
      }
      g.appendChild(el('text', {x:X1, y:Y + 2 * GAP + 50, fill:'#565656', 'font-size':9.5, 'text-anchor':'end'}, 'Alice to Bob, km'));
      var xm = X0 + Math.min(350, top) / top * (X1 - X0);
      g.appendChild(el('line', {x1:xm, y1:14, x2:xm, y2:Y + 2 * GAP + 22, stroke:'#7f8fd6', 'stroke-width':1.2, 'stroke-dasharray':'4 3'}));
      g.appendChild(el('text', {x:xm + 4, y:24, fill:'#7f8fd6', 'font-size':9.5}, "the paper's 350 km"));

      res.forEach(function(r, i){
        var y = Y + i * GAP, w = Math.max(1, r.km / top * (X1 - X0));
        g.appendChild(el('text', {x:X0-10, y:y+12, fill:'#c8c8c8', 'font-size':11, 'text-anchor':'end'}, r.b.n));
        g.appendChild(el('text', {x:X0-10, y:y+25, fill:'#6f6f6f', 'font-size':9.5, 'text-anchor':'end'}, r.a.toFixed(2) + ' dB/km'));
        g.appendChild(el('rect', {x:X0, y:y, width:w, height:20, rx:3, fill: i === 0 ? '#f2a03d' : '#32c29e', opacity: i === 2 ? 1 : 0.82}));
        g.appendChild(el('text', {x:X0 + w + 8, y:y+14, fill:'#e6e6e6', 'font-size':10.5}, r.km.toFixed(0) + ' km'));
      });

      H.querySelector('#qwb-a').textContent = res[0].km.toFixed(0) + ' km';
      H.querySelector('#qwb-b').textContent = res[1].km.toFixed(0) + ' km';
      H.querySelector('#qwb-c').textContent = res[2].km.toFixed(0) + ' km';

      note.innerHTML = 'each conversion costs <b>' + conv.toFixed(1) + ' dB</b> at this efficiency, taken out of the ' +
        half.toFixed(1) + ' dB available on each side. ' +
        (plant === 'spec'
          ? 'this is data-sheet fiber. press <em>deployed plant</em> for the 0.49 dB/km the group actually measured on its 35 km Boston loop.'
          : 'deployed numbers include splices, patch panels and a route that does not go in a straight line.');
    }
    btnGroup(H, 'pl', function(v){ plant = v; draw(); });
    se.addEventListener('input', draw);
    sl.addEventListener('input', draw);
    draw();
  })();
})();
</script>{% endraw %}