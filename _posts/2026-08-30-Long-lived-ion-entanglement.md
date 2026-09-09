---
layout: post
section-type: post
title: "Long-lived ion entanglement : when the memory outlasts the wait"
category: 'networking'
tags: [ 'networking', 'fiber', 'quantum' ]
---

every post on this blog so far has been about moving entanglement through glass. [which fiber]({% post_url 2026-01-03-five-kinds-of-fiber-and-when-to-use-which %}), [how to share it with the clock]({% post_url 2026-03-23-coexistence-running-a-quantum-channel-and-its-clock-down-the-same-fiber %}), [how to divide the spectrum]({% post_url 2026-04-04-Flex-grid-for-entanglement %}), [where the spectrum comes from]({% post_url 2026-05-29-eight-users-on-one-chip-maximizing-the-source-side-of-a-flex-grid-quanntum-network %}). all of it point to point. all of it dies at a couple of hundred kilometers, because photon loss in fiber is exponential and no amount of clever allocation changes the exponent.

the thing that changes the exponent is a repeater. you put nodes in the middle, entangle each node with its neighbour over a short hop, and then perform an entanglement swap so that the two end nodes share a pair that never travelled the whole distance. 1,000 km of direct fiber costs you a factor of 10<sup>20</sup> in photon loss. ten repeater nodes bring that to about 10<sup>2</sup>.

that has been the plan since 1998 and the reason it has not happened is not the swap. it is timing.

each hop succeeds at random. so the first hop entangles, and then you sit there holding that pair while the second hop keeps trying. if the pair falls apart before the second hop lands, you have nothing to swap, and you start again. **the memory has to outlive the wait for the next success.** [Delft cleared that bar in 2018](https://doi.org/10.1038/s41586-018-0200-5) with nitrogen-vacancy centres in diamond: 39 Hz of entangling against 5 Hz of decoherence, delivering a fresh pair every clock cycle. across two metres of lab bench, with the photons at their native wavelength. put ten kilometres of telecom fiber between the nodes and that ratio has never survived. the failures were not close.

this paper passes it. [Liu et al., "Long-lived remote ion–ion entanglement for scalable quantum repeaters," *Nature* (2026)](https://doi.org/10.1038/s41586-026-10177-4). two trapped calcium ions, 10 km of fiber between them, entanglement that lives 550 ms against an average wait of 450 ms. a ratio just above one across ten kilometres of fiber, which nobody had managed before.

there is also a device-independent QKD demonstration bolted on, which got most of the press, and which i think is the less interesting half. below: how the machine works, why the one knob it has moves both of the numbers you care about, what the QKD run actually bought, and what happens when you take the paper's own figures and ask what would come out of an actual swap.

## the one number

call `R` the rate at which a link successfully heralds entanglement, and `Γ` the rate at which the entanglement you are holding decoheres. the paper calls the ratio the quantum link efficiency:

```
    η_link  =  R / Γ
```

if that is much less than one, entanglement dies faster than you can make it, and a repeater chain never has two live pairs at once. if it is above one, the pair you are holding is likely still there when the next one arrives, and you can swap.

the numbers here are `R` = 2.226 Hz and `Γ` = 1.8 ± 0.1 Hz, so `η_link` = 1.2. the paper compares this to a critical threshold of 0.83 and reports clearing it.

that is the headline, and it is real. it is also the first place the paper stops being careful, which i will come back to.

<div class="qw" id="qw-race" data-qw><div class="qw-hd"><span class="qw-t">the race the whole field has been losing</span><span class="qw-s">one link holds a pair while the other keeps trying. the pair decays; the wait is random. drag the excitation knob and watch both curves move, in opposite directions.</span><span class="qw-lg"><i style="background:#e2603f"></i>fidelity of the pair you are holding<i style="background:#6ba3f0"></i>chance the next success has arrived<i style="background:#32c29e"></i>expected fidelity at swap time</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">the paper&rsquo;s operating points</span><span class="qw-b" data-grp="ps"><button type="button" data-v="2.5">DI-QKD, &alpha; = 2.5%</button><button type="button" data-v="9.4">breaks even</button><button type="button" data-v="17" class="on" aria-pressed="true">the headline, &alpha; = 17%</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">excitation probability <span class="qw-gk">&alpha;</span> <b id="qwr-av">17.0%</b></span><input type="range" id="qwr-a" min="1" max="22" step="0.1" value="17"></label><label class="qw-sr"><span class="qw-l">memory coherence time <b id="qwr-tv">550 ms</b></span><input type="range" id="qwr-t" min="100" max="4000" step="25" value="550"></label></div><div class="qw-pane"><span class="qw-pl">the pair decaying, against the distribution of how long you wait</span><svg class="qw-svg" viewBox="0 0 640 230" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Fidelity decay of a stored pair against the waiting time distribution for the next success"><g id="qwr-plot"></g></svg></div><div class="qw-out"><div class="qw-oi"><span class="qw-ok">heralding rate</span><span class="qw-ov" id="qwr-r">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">average wait</span><span class="qw-ov" id="qwr-w">&mdash;</span></div><div class="qw-oi"><span class="qw-ok qw-bs">link efficiency</span><span class="qw-ov" id="qwr-e">&mdash;</span></div><div class="qw-oi"><span class="qw-ok qw-fs">its own threshold</span><span class="qw-ov" id="qwr-th">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">fidelity when the next one lands</span><span class="qw-ov" id="qwr-f">&mdash;</span></div><div class="qw-oi qw-grow"><span class="qw-ok">verdict</span><span class="qw-ov qw-txt" id="qwr-v">&mdash;</span></div></div><p class="qw-n" id="qwr-note"></p><noscript><p class="qw-n">At the paper&rsquo;s headline operating point, &alpha; = 17%, the link heralds at 2.226 Hz against a decoherence rate of 1.8 Hz, so the link efficiency is 1.24 and the expected fidelity of a stored pair when the next success arrives is 0.578. Turning &alpha; down to the 2.5% used for the QKD run raises the fresh-pair fidelity to 0.92 but drops the heralding rate to 0.327 Hz, and the link efficiency with it, to 0.18.</p></noscript></div>

## the machine

each node is one <sup>40</sup>Ca<sup>+</sup> ion in a trap. Doppler cooled, then EIT cooled on every motional mode, then optically pumped into a specific Zeeman sublevel with 99.9% fidelity. the qubit is stored in two states of that ion, and read out by shining light on it and seeing whether it glows.

the photon comes out in two steps. a 729 nm π pulse moves the population up, and then a 2.25 ns pulse at 854 nm knocks a fraction `α` of it back down through an excited state, emitting a 393 nm photon on the way. that fraction is the knob this whole post is about.

393 nm is ultraviolet and useless for fiber, so each node has a quantum frequency conversion module: a periodically poled lithium niobate waveguide pumped at 527 nm, converting 393 nm straight to 1550 nm in one step. the conversion also generates a wall of broadband noise, 1.3 × 10<sup>5</sup> photons per second per nanometre around 1550, which gets knocked down to 35 counts per second by a three-stage filter and costs 72% of the signal to do it.

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Stage</th><th>What it is</th><th>Number</th></tr>
</thead>
<tbody>
<tr><td>Collection</td><td>NA 0.635 objective, perpendicular to the quantization axis</td><td>~9% of &sigma;-transition photons</td></tr>
<tr><td>Into fiber</td><td>SM300, before conversion</td><td>2.6% (Alice), 2.8% (Bob)</td></tr>
<tr><td>Conversion</td><td>PPLN waveguide, 527 nm pump at ~60 mW</td><td>393 &rarr; 1550 nm, single step</td></tr>
<tr><td>Filtering</td><td>100 GHz bandpass + 10 GHz VBG + 40 MHz etalon</td><td>28% transmission, noise down to 35 cps</td></tr>
<tr><td>Photon shape</td><td>etalon ring-down broadens it</td><td>~20 ns FWHM at 1550, against a 7.45 ns decay at 393</td></tr>
<tr><td>The 10 km link</td><td>conversion module to detectors</td><td>9.1% efficiency, 9.6 cps noise, SNR &gt; 100:1</td></tr>
<tr><td>Phase lock</td><td>1548 nm CW reference (WDM) + 393 nm pulse (TDM)</td><td>contrast 0.986 &plusmn; 0.005 over 10 km</td></tr>
<tr><td>Memory</td><td>transfer to a longer-lived sublevel, KDD decoupling</td><td>550 &plusmn; 36 ms</td></tr>
</tbody>
</table>
</div>

the paper never states the attempt rate, but you can back it out. a herald needs one photon from either node to reach the midpoint detectors, so the probability per attempt is roughly `2 × α × 0.026 × 0.091`. at α = 17% that is 8 × 10<sup>-4</sup>, or one attempt in twelve hundred. divide the 2.226 Hz herald rate by that and the trap is cycling at about 2.8 kHz, roughly 360 μs per attempt, which is about what a cool-prepare-excite cycle costs. run the same arithmetic at α = 2.5% and you get the same 2.8 kHz, which is a good sign the inference is sound.

so: the rate is not limited by anything exotic. it is limited by collecting 2.6% of the photons the ion emits.

### single-photon interference, and the price of it

the entangling protocol is the interesting design choice. there are two families.

two-photon interference: both nodes emit, both photons travel to a midpoint, and you herald on a coincidence. the good news is that the phase between the two paths cancels out, so you do not need the fiber to hold an optical phase. the bad news is that you need both photons to survive, so the rate goes as transmission *squared*.

single-photon interference (this paper, following [Cabrillo](https://doi.org/10.1103/PhysRevA.59.1025) and [DLCZ](https://doi.org/10.1038/35106500)): each node weakly excites, so that at most one photon is out there at a time, and a single click at the midpoint heralds entanglement without revealing which node it came from. rate goes as transmission to the *first* power. over 100 km of fiber that is the difference between a demonstration and a fantasy.

what you pay for it is two things. the first is that the interferometer is now real: the optical phase between the two nodes has to be stable to a fraction of a wavelength across the whole link, which is why half the apparatus is phase stabilization. the second is subtler and it is the α knob.

weak excitation means small α. if α is large, both nodes sometimes emit at once, you still get one click, and the state you herald is `|↓↓⟩` instead of a Bell pair. so the state you actually make is

```
    ρ_AB  =  (1 − α) |ψ±⟩⟨ψ±|  +  α |↓↓⟩⟨↓↓|
```

and the infidelity has a floor that is linear in α, while the rate is also linear in α. one knob, both directions.

<div class="qw" id="qw-alpha" data-qw><div class="qw-hd"><span class="qw-t">where the infidelity comes from, and which part you control</span><span class="qw-s">the error budget of Fig. 3c, split by source. only the first bar moves with the knob that sets your rate; the rest are the apparatus.</span><span class="qw-lg"><i style="background:#e2603f"></i>double excitation, &prop; &alpha;<i style="background:#f2a03d"></i>ion errors and decoherence<i style="background:#6ba3f0"></i>photon and detector noise<i style="background:#8f8f8f"></i>magnetic noise, without decoupling</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">spin-echo dynamical decoupling</span><span class="qw-b" data-grp="se"><button type="button" data-v="1" class="on" aria-pressed="true">on, as measured</button><button type="button" data-v="0">off</button></span></div><div class="qw-g"><span class="qw-l">jump to</span><span class="qw-b" data-grp="ps"><button type="button" data-v="2.5|10">tomography, 10 km</button><button type="button" data-v="5|101">the distance scan, 101 km</button><button type="button" data-v="17|10">the memory run</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">excitation probability <span class="qw-gk">&alpha;</span> <b id="qwa-av">2.5%</b></span><input type="range" id="qwa-a" min="1" max="22" step="0.1" value="2.5"></label><label class="qw-sr"><span class="qw-l">link length <b id="qwa-lv">10 km</b></span><input type="range" id="qwa-l" min="0" max="120" step="1" value="10"></label></div><div class="qw-two"><div class="qw-pane"><span class="qw-pl">infidelity, stacked by contributor</span><svg class="qw-svg" viewBox="0 0 320 200" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Stacked infidelity contributions versus link length"><g id="qwa-stack"></g></svg></div><div class="qw-pane"><span class="qw-pl">rate against fidelity, the whole tradeoff</span><svg class="qw-svg" viewBox="0 0 320 200" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Heralding rate against Bell state fidelity as alpha varies"><g id="qwa-curve"></g></svg></div></div><div class="qw-out"><div class="qw-oi"><span class="qw-ok">fresh Bell fidelity</span><span class="qw-ov" id="qwa-f">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">heralding rate</span><span class="qw-ov" id="qwa-r">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">double-excitation share</span><span class="qw-ov" id="qwa-s">&mdash;</span></div><div class="qw-oi qw-grow"><span class="qw-ok">what this state is good for</span><span class="qw-ov qw-txt" id="qwa-v">&mdash;</span></div></div><p class="qw-n" id="qwa-note"></p><noscript><p class="qw-n">At &alpha; = 2.5% over 10 km the paper measures Bell-state fidelities of 0.923 and 0.910, and the modelled error budget puts roughly half the infidelity on the protocol&rsquo;s own double-excitation term. Because spin-echo decoupling suppresses magnetic-field noise, the fidelity is nearly flat with distance: it is still above 0.90 at 101 km. Without that decoupling the same 101 km link would sit near 0.65.</p></noscript></div>

the second panel is the one to sit with. **distance is nearly free and α is not.** going from 10 km to 101 km costs a couple of points of fidelity, because spin-echo decoupling handles the magnetic-field noise that would otherwise dominate over the extra half-millisecond of flight time. going from α = 2.5% to α = 17% costs thirteen points, and it is the only way to make the link fast.

## the memory

once a pair is heralded, it has to survive. the qubit lives in a metastable D state with a natural lifetime of 1.16 seconds, which sounds generous until you notice that the same state is magnetically sensitive and the lab has a magnetic field with noise on it.

so they move it. a two-step transfer, a bichromatic Raman transition plus a π rotation, parks the entanglement in a different pair of sublevels, and then a Knill dynamical decoupling sequence with a 0.5 ms interval runs continuously to average out the field noise.

the result is the 550 ± 36 ms coherence time. but the shape of the decay is worth looking at, because it is two different things at once:

- ⟨XX⟩ decays exponentially, with the 550 ms time constant. this is the magnetic dephasing the decoupling is fighting, and it is the number that gets quoted.
- ⟨ZZ⟩ decays linearly, and the paper attributes it to accumulated gate errors from the repeated π rotations of the decoupling sequence itself. per-gate error probabilities are 6.1 × 10<sup>-4</sup> at Alice and 5.7 × 10<sup>-4</sup> at Bob, dominated by spontaneous emission.

that second bullet is a treadmill. **the thing protecting the memory is now one of the things degrading it**, and it degrades it in proportion to how long you hold. hold twice as long and you run twice as many pulses. the paper gives the per-gate error but not the pulse count, so you cannot extrapolate, and the pulse count is exactly the number you would need to know whether a second of storage is a hardware problem or a protocol problem.

put the decay together with the random wait and you get the two averages the paper reports:

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Quantity</th><th>Value</th><th>What it means</th></tr>
</thead>
<tbody>
<tr><td>Coherence time of the memory&ndash;memory pair</td><td>550 &plusmn; 36 ms</td><td>the headline</td></tr>
<tr><td>Average entanglement generation time</td><td>450 ms</td><td>at &alpha; = 17%</td></tr>
<tr><td>Fidelity at 450 ms</td><td>0.554 &plusmn; 0.050</td><td>a pair that waited the average time</td></tr>
<tr><td>Stays above 0.5 for</td><td>547 &plusmn; 34 ms</td><td>the useful window</td></tr>
<tr><td>Expected fidelity, 450 ms window</td><td>0.668 &plusmn; 0.005</td><td>conditioned on a fast second success</td></tr>
<tr><td>Expected fidelity, no window</td><td>0.578 &plusmn; 0.006</td><td><b>what a swap would actually get</b></td></tr>
</tbody>
</table>
</div>

that last row is the paper's real result. **when the next entanglement round succeeds, the pair you have been holding is at 0.578 on average.** it is above 0.5, it is entangled, and over fiber nothing had previously come near it.

## the DI-QKD half

the application section is device-independent QKD. it is worth a paragraph of setup, because it is the strongest security claim in cryptography and the reason anyone bothers to build a Bell test into a network in the first place.

ordinary QKD assumes your hardware does what the datasheet says. if your source emits two photons when it claims one, or your detector has a blind spot someone can steer, the security proof does not cover you, and most practical attacks on QKD have been exactly this. device-independent QKD throws the datasheet away. the only thing it trusts is that the boxes are in separate rooms and cannot signal each other. if the measured correlations violate a Bell inequality by enough, then no eavesdropper, and no dishonest manufacturer, can know the key, regardless of what is inside the boxes.

the cost is that you need very good entanglement, because a Bell violation is fragile, and you need a lot of rounds, because a device-independent security proof against general attacks has to be conservative about statistics it cannot assume anything about.

here is what they got over 10 km:

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Quantity</th><th>10 km</th><th>101 km</th></tr>
</thead>
<tbody>
<tr><td>Rounds collected</td><td>405,145</td><td>2,799</td></tr>
<tr><td>Data collection</td><td>386.1 h valid, ~2 months wall clock</td><td>not stated</td></tr>
<tr><td>CHSH value <i>S</i></td><td>2.5758 &plusmn; 0.0059</td><td>2.504 &plusmn; 0.075</td></tr>
<tr><td>QBER</td><td>3.60% &plusmn; 0.06</td><td>6.9% &plusmn; 1.1</td></tr>
<tr><td>Average round rate</td><td>0.291 s<sup>&minus;1</sup></td><td>not stated</td></tr>
<tr><td>Asymptotic key rate</td><td>0.325 bits/round</td><td>0.0974 bits/round</td></tr>
<tr><td>Finite-size key</td><td><b>1,917 bits</b>, ~4 &times; 10<sup>&minus;3</sup>/round</td><td><b>none computed</b></td></tr>
</tbody>
</table>
</div>

that asymptotic number is easy to check. the standard device-independent bound is

```
    r∞  =  1 − h(Q) − h( (1 + √(S²/4 − 1)) / 2 )
```

and plugging in S = 2.5758, Q = 0.0360 gives 0.326, against the paper's 0.325. at 101 km it gives 0.0989 against 0.0974. the model is the model they used.

now look at the finite-size column. ****1,917 secret bits**, from 405,145 rounds, is 4.7 × 10<sup>-3</sup> bits per round against an asymptotic 0.326.** the finite-size correction ate 98.5% of the key.

<div class="qw" id="qw-key" data-qw><div class="qw-hd"><span class="qw-t">the 98.5% you pay for not having enough rounds</span><span class="qw-s">device-independent security proofs are brutal on small samples. the two curves are the two analyses in the paper: the one they used, and the one they said they could not afford.</span><span class="qw-lg"><i style="background:#32c29e"></i>asymptotic<i style="background:#f2a03d"></i>complementarity, what they used<i style="background:#6ba3f0"></i>entropy accumulation</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">which link</span><span class="qw-b" data-grp="ln"><button type="button" data-v="10" class="on" aria-pressed="true">10 km, S = 2.576</button><button type="button" data-v="101">101 km, S = 2.504</button></span></div><div class="qw-g"><span class="qw-l">landmarks</span><span class="qw-b" data-grp="ps"><button type="button" data-v="405145" class="on" aria-pressed="true">their run</button><button type="button" data-v="1500000">what EAT needs</button><button type="button" data-v="1200000">the Science paper, 3 weeks later</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">rounds collected <b id="qwk-nv">405,145</b></span><input type="range" id="qwk-n" min="400" max="800" step="1" value="561"></label></div><div class="qw-pane"><span class="qw-pl">secret bits per round against sample size</span><svg class="qw-svg" viewBox="0 0 640 250" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Secret key rate per round versus number of rounds for three security analyses"><g id="qwk-plot"></g></svg></div><div class="qw-out"><div class="qw-oi"><span class="qw-ok qw-bs">asymptotic</span><span class="qw-ov" id="qwk-ra">&mdash;</span></div><div class="qw-oi"><span class="qw-ok qw-fs">complementarity</span><span class="qw-ov" id="qwk-rc">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">total secret bits</span><span class="qw-ov" id="qwk-b">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">time to collect</span><span class="qw-ov" id="qwk-t">&mdash;</span></div><div class="qw-oi qw-grow"><span class="qw-ok">verdict</span><span class="qw-ov qw-txt" id="qwk-v">&mdash;</span></div></div><p class="qw-n" id="qwk-note"></p><noscript><p class="qw-n">Over 10 km the paper collected 405,145 rounds in 386 hours of valid time and extracted 1,917 secret bits, about 4.7 &times; 10<sup>&minus;3</sup> per round against an asymptotic 0.326. An entropy-accumulation analysis under the same soundness parameter would need roughly 1.5 million rounds, which at their measured 0.291 rounds per second is another 1,432 hours of valid data, or about seven months of wall clock at the duty cycle they achieved.</p></noscript></div>

the paper is straightforward about this. it says an entropy accumulation analysis would need about 1.5 × 10<sup>6</sup> rounds under the same soundness parameter, which exceeds the data they have, and that they plan to report it separately. that is the right thing to do and the right way to say it.

it is worth knowing what that costs in wall clock. 1.5 million rounds at 0.291 per second is 1,432 hours of *valid* accumulation. their 405,145 rounds took 386 valid hours spread over about two months, so the duty cycle is roughly a quarter. the missing analysis is about seven more months of running the experiment, and that is at 10 km. at 101 km, where they collected 2,799 rounds, it is not a schedule, it is a career.

## what's missing

the paper is careful and mostly honest, and a lot of what follows it names itself. i am separating the limitations it states from the gap between what got measured and what the abstract claims.

**the two headline numbers live at opposite ends of the same knob.** the link efficiency of 1.2 is measured at α = 17%, where the fresh Bell state is at about 0.80. the QKD run needs a Bell violation, so it ran much weaker. the paper never states which α, but the 0.291 rounds per second it reports for the QKD data is what α = 2.5% gives to three decimal places, and the 3.6% QBER matches the 0.92 fidelity measured there. that operating point has a link efficiency of 0.18, a factor of three below even the friendlier 0.58 threshold that its own better fidelity earns it. both results are real. **the system that produced both of them never existed at the same time**, and the paper never puts the two operating points on the same page.

**0.83 is not a constant.** the paper cites it as the critical threshold for deterministic delivery and moves on without deriving it. here is a derivation that lands exactly on it. if a pair with initial fidelity `F₀` decays toward the maximally mixed state and the wait is exponential with rate `R`, the expected fidelity at swap time is

```
    E[F]  =  0.25  +  (F₀ − 0.25) · R / (R + Γ)
```

set that equal to 0.5 and solve for the ratio:

```
    R / Γ   >   x / (1 − x) ,      x  =  0.25 / (F₀ − 0.25)
```

at `F₀` = 0.80 that gives 0.833. the threshold is not a property of the universe, it is what you get if you assume the fresh pair is at 0.80, which is the fidelity at α = 17%. so the same move that raises your rate above the bar also lowers the bar. run the numbers across α and the link clears its own threshold only above about 9.4%, and the margin at 17% is real but smaller than a comparison against a fixed 0.83 makes it look. the honest statement is that the pair `(rate, fidelity)` has to clear a curve, not a line.

**there is no second link, and nothing was swapped.** this is the big one. the paper's claim is that the result "confirms that multiple memory–memory entanglements can be established and maintained across long distance, enabling multi-stage entanglement swapping and entanglement purification." there is one link. it was measured, and then a second entanglement round on the *same* link was used to define the waiting time. no two links ever existed simultaneously and no Bell measurement was ever performed on two pairs.

that would be a fair thing for a first paper to leave undone, except that the composition is checkable from the paper's own numbers, and the answer is uncomfortable.

**and if you do check it, the block does not compose.** take the standard rule: swapping two depolarized Bell pairs with parameters `p₁` and `p₂` gives a pair with parameter `p₁p₂`, where `p = (4F − 1)/3`. in a repeater the swap happens when the second link succeeds, so one pair is fresh and the other is the one that has been waiting:

```
    fresh pair at α = 17%   F = 0.797   →  p = 0.729
    stored pair, expected   F = 0.578   →  p = 0.437
    swapped                 p = 0.319   →  F = 0.49
```

**0.49 is below 0.5, which for a Werner state is the separability boundary.** and it is not an unlucky choice of α: sweep the knob across its whole range and the best swapped fidelity you can reach is 0.493, at α ≈ 14%. to clear 0.5 at α = 17% you would need the stored pair at 0.593 rather than 0.578. they miss by fifteen thousandths.

the caveat matters and i want to be clear about it: the noise here is not depolarizing. the error term is `α|↓↓⟩⟨↓↓|`, a specific product state, and structured noise composes differently from white noise, possibly better. so this estimate could be wrong in the paper's favour. but that is the point. the paper claims the result enables multi-stage swapping and never computes what a swap would produce, and under the standard composition rule the answer sits exactly on the boundary between "a repeater" and "two separable states". either the noise structure saves it, which would be a genuinely nice result worth showing, or it does not. nobody knows, and the calculation is one page.

**two nodes, one lab, one laser.** the paper says this plainly: Alice and Bob share a common laser system with independent electronics, and it argues that this cancels the lasers' intrinsic high-frequency phase noise at the interferometer, "representing a significant difference from experiments using independent lasers." it then argues that the residual phase noise, dominated by lab acoustics, is comparable to numbers reported for field-deployed links, and concludes that field deployment would be feasible.

that argument has a hole in the middle of it. single-photon interference requires optical phase stability between two remote nodes, and sharing a laser is precisely the mechanism that removes the hardest part of that requirement. the comparison is to a [twin-field QKD link over 511 km](https://doi.org/10.1038/s41566-021-00828-5), which is a different measurement of a different quantity. the substitution they propose, independent lasers phase-locked by optical frequency transfer with a comb at each node, is named as future work, and everything about whether this scheme survives outside one room depends on it.

**the fiber is on spools, and the herald delay is electrical.** the photons really do traverse 10 and 101 km of glass. but the delay between the midpoint heralding event and the subsequent local measurements was inserted electrically to emulate the fiber. that is a reasonable way to test the memory's ability to hold through the classical round trip. it is not a test of what buried fiber does to an interferometer over 50 km on each side while a train goes past.

**101 km is a limit, not a key.** the abstract says "a positive key rate over 101 km in the asymptotic limit." the asymptotic limit is where you have infinitely many rounds. they have 2,799. there is no finite-size analysis at 101 km and the paper says so, but "extending the achievable distance by more than two orders of magnitude" is a claim about a number that does not correspond to any key anyone could use.

**the collection efficiency is the whole story and it is 2.6%.** everything downstream (the conversion, the filtering, the link) is respectable. the rate is set almost entirely by how few of the ion's photons make it into a fiber, and the paper's own suggested fix, an optical cavity around the ion, is a known technique with [demonstrated results in this exact platform](https://doi.org/10.1103/PhysRevLett.130.050803). the difference between 2.6% and 25% is the difference between a 1.2 link efficiency and a 12.

**the storage protocol is a source of error and the pulse count is missing.** noted above. the linear ⟨ZZ⟩ decay means the decoupling sequence has its own error budget that grows with hold time, and the per-gate error is dominated by spontaneous emission, which is a property of the level scheme rather than of the laser. you cannot extrapolate this to one second of storage with what is in the paper.

**the DI-QKD half was overtaken within three weeks.** more on this below, but it belongs in this list: the same institution published a better DI-QKD result in *Science* eighteen days after this paper was accepted, on a different platform, using the analysis this paper says it could not afford.

## what i'd build next

**1. do the swap.** two links, four ions, one Bell measurement in the middle, and report the fidelity of the swapped pair. that is the experiment that turns "a building block" into a repeater segment, it is the thing the title claims, and everything in this paper says it is nearly within reach. i suspect the first attempt lands just under 0.5 and that this is worth publishing, because the gap between 0.49 and 0.55 is a small number of engineering decisions and knowing which ones is the useful result.

**2. before that, compute the composition properly.** the depolarizing estimate above is a stand-in. the actual state is known, the swap is a Bell measurement, and the output fidelity is a page of algebra. do it with the real noise structure, publish the curve of swapped fidelity against α, and the field gets a design rule instead of a threshold. this is a theory afternoon, not a grant.

**3. put a cavity on the ion.** the single largest lever in the whole system, by a distance. every other number in the paper is fine.

**4. report the pair (rate, fidelity), never the ratio alone.** the link efficiency is a useful metric and it is not sufficient, because the threshold it is compared against moves with the fidelity. a two-number figure of merit, or better, the expected swapped fidelity, would let two experiments on different platforms actually be compared. right now "we exceeded 0.83" can mean several incompatible things.

**5. state the attempt rate and the pulse count.** two numbers that are certainly in someone's lab notebook, both of which the reader has to reconstruct or guess. the attempt rate turns the herald probability into an engineering target; the decoupling pulse count says whether longer storage is free or quadratically expensive.

**6. run the DI-QKD at the memory's operating point.** the two halves of this paper are run at different α, and there is a genuinely interesting question sitting between them: what does the key rate look like as a function of α, when the link efficiency and the Bell violation are both functions of it? there is an optimum somewhere and nobody has found it. **it is the same shape of problem as the [finite-key-aware spectrum allocator]({% post_url 2026-05-29-eight-users-on-one-chip-maximizing-the-source-side-of-a-flex-grid-quanntum-network %}) i wanted from the flex-grid papers**: an objective function with a hard discontinuity in it, being optimized against a smooth proxy.

**7. independent lasers, then a real fiber.** in that order. the shared-laser question is answerable on a bench with an optical frequency transfer link and a comb at each node, and it should be answered before anyone buries anything. after that, the same 10 km on deployed fiber with the acoustics and thermal drift of an actual duct, and the herald delay carried on real glass instead of a cable.

**8. and eventually, the thing that makes this a network problem.** a repeater chain is not two links, it is a scheduling problem: which segments attempt, which hold, when to cut losses on a stored pair and re-attempt, how to allocate a purification budget. all of that is downstream of a rate and a coherence time that this paper is the first to put on the right side of one over fiber. the literature on those policies, [when to give up on a stored pair and re-attempt](https://doi.org/10.1038/s41534-023-00713-9) most obviously, is almost entirely simulation, because until now there were no hardware numbers to calibrate it against. there are now, and they are uncomfortably close to the boundary, which is exactly where a scheduling policy earns its keep.

## what happened since

the paper was accepted on 23 January 2026. it has been a busy eight months.

**the DI-QKD result was superseded almost immediately, by the same institution.** on 10 February, eighteen days after acceptance, [Lu et al. posted "Device-independent quantum key distribution over 100 km with single atoms"](https://arxiv.org/abs/2602.09596), published in *Science* ([10.1126/science.aec6243](https://doi.org/10.1126/science.aec6243)). single rubidium atoms rather than ions, the same single-photon interference trick, and an author list that overlaps this paper's: Feihu Xu, Qiang Zhang, Yi-Zheng Zhen, Ming-Yang Zheng and Jian-Wei Pan are on both.

the comparison is not close.

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th></th><th>Liu et al., <i>Nature</i></th><th>Lu et al., <i>Science</i></th></tr>
</thead>
<tbody>
<tr><td>Memory</td><td>trapped <sup>40</sup>Ca<sup>+</sup> ions</td><td>single <sup>87</sup>Rb atoms</td></tr>
<tr><td>Rounds, finite-size key</td><td>405,145</td><td>1.2 million</td></tr>
<tr><td>Data collection</td><td>386 h</td><td>624 h</td></tr>
<tr><td>Finite-size key rate</td><td>4.7 &times; 10<sup>&minus;3</sup> bits/round at 10 km</td><td>0.112 bits/round at 11 km</td></tr>
<tr><td>Security analysis</td><td>complementarity</td><td>R&eacute;nyi EAT, and original EAT at 0.034</td></tr>
<tr><td>Longest link with a Bell violation</td><td>101 km, asymptotic only</td><td>100 km, S &gt; 2 at 0.911 fidelity</td></tr>
</tbody>
</table>
</div>

twenty-four times the key rate per round, on the entropy accumulation analysis this paper says it cannot afford, with the same 100 km reach. if you were reading the *Nature* paper for the cryptography, read the *Science* one instead.

what the *Science* paper does not have is the memory result. its atoms hold coherence past 300 ms using clock states, which is respectable, but the quantity this paper is about, remote memory–memory entanglement outliving the average wait for the next round, is not in it. so the two papers divide cleanly, and it reinforces my read: **the repeater half is this paper's contribution and the QKD half was a race it had already lost when it went to press.**

**the ion-photon version of the same metric got there first, quietly.** in December 2025, [Cui et al. in *Nature Communications*](https://www.nature.com/articles/s41467-025-67311-5) reported metropolitan-scale ion–photon entanglement over 12 km with hybrid multiplexing: 44 time-bin modes plus ion shuttling across four communication qubits, a 15.6× multiplexing enhancement, 4.28 events per second at 89.8% fidelity, and "a record-high ratio of remote ion-photon entangling rate to memory decoherence rate of 1.16." same lab, same metric, one photon instead of two memories. the *Nature* paper's 1.2 is the harder version of the same number, and the multiplexing that got the earlier result to 1.16 is not in it, which is a fairly clear signpost for where the rate improvement comes from next.

**trapped ions cleared three nodes.** in June, [Duke and IonQ posted an event-ready three-node GHZ state](https://arxiv.org/abs/2606.17173) across three independently controlled barium ion modules, with no local two-qubit gates and no post-selection: fidelity 84.1 to 88.1%, 0.095 events per second, and a loophole-free Mermin violation at 27 standard deviations. three metres, not a hundred kilometres, and the rate is more than twenty times slower than the link in this paper. but it is the multi-node topology this paper argues toward and does not build.

**and the non-memory approach keeps embarrassing everybody on rate.** in February, [Craddock et al. ran entanglement swapping between remote sources over 17.6 km of deployed New York City fiber](https://arxiv.org/abs/2602.15653) at nearly 500 swapped pairs per second with the CHSH parameter above 2, using warm atomic vapour cells. that is two hundred times the heralding rate of the link in this paper, on real fiber, in a real city. it is also not a repeater: warm vapour cells do not store for 550 ms, so there is nothing to hold while the next hop tries. the two lines of work are measuring different things and the gap between them is the entire remaining problem.

## the short version

- a quantum repeater needs a memory that outlives the wait for the next entanglement attempt. the metric is `η_link = R/Γ`, the heralding rate over the decoherence rate, and no memory–memory experiment had ever put it above one.
- this one does: **550 ± 36 ms of coherence against a 450 ms average wait, η_link = 1.2.** two trapped calcium ions, 10 km of fiber, telecom conversion at each node.
- the protocol is single-photon interference, so the rate goes as transmission rather than transmission squared, which is what makes 100 km thinkable. the price is that the link is a real interferometer and that the infidelity has a floor linear in the excitation probability α.
- **α is one knob and it moves everything.** rate is linear in it, infidelity is linear in it. distance, by contrast, is nearly free: fidelity above 0.90 out to 101 km, because dynamical decoupling handles the magnetic noise.
- the rate is limited by collecting 2.6% of the ion's photons into a fiber. one herald per twelve hundred attempts. a cavity is the single biggest available improvement and the paper names it.
- the memory decays two ways: ⟨XX⟩ exponentially from magnetic noise, ⟨ZZ⟩ linearly from the decoupling pulses' own gate errors. the protection scheme is now one of the error sources, and the pulse count needed to extrapolate it is not in the paper.
- the expected fidelity of a stored pair when the next round succeeds is 0.578, which is the number the whole paper exists to report.
- **0.83 is not a constant.** it is `x/(1−x)` with `x = 0.25/(F₀ − 0.25)` evaluated at `F₀` = 0.80, which is the fidelity at α = 17%. the threshold moves with the same knob as the rate.
- **swap a stored pair with a fresh one, under the standard depolarizing composition, and you get 0.49**, just below the separability boundary, and the best any α reaches is 0.493. the real noise is not depolarizing so this may be pessimistic, but the paper claims multi-stage swapping and never does the calculation.
- DI-QKD over 10 km: 405,145 rounds over 386 hours yields 1,917 secret bits, 4.7 × 10<sup>-3</sup> per round against an asymptotic 0.326. **the finite-size correction ate 98.5% of it.** the stronger analysis needs 1.5 million rounds, about seven more months of wall clock.
- 101 km has 2,799 rounds, no finite-size analysis, and an asymptotic key rate. that is a statement about a limit, not a key.
- both nodes share a laser, sit in one lab, and are joined by spooled fiber with an electrically emulated herald delay. all stated. all load-bearing.
- three weeks after acceptance the same institution published DI-QKD at 0.112 bits per round over 100 km with single atoms, twenty-four times better, using the analysis this paper could not afford.

the thing i keep coming back to is how narrow the margin is, in both directions. the link efficiency clears its threshold by 40% and the threshold turns out to depend on the knob that got it there. the stored pair clears the separability bound by 0.078 and then loses all of it plus a little in a single swap. this is what the interesting part of an engineering problem looks like from the inside: not a wall, but a set of numbers that are all *just* on the wrong side of composing, where the next few papers are about finding the fifteen thousandths.

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
  function log2(x){ return Math.log(x)/Math.LN2; }
  function ms(t){ return t<1000 ? Math.round(t)+' ms' : (t/1000).toFixed(t<10000?2:1)+' s'; }
  function dur(h){
    if(h<48) return h.toFixed(0)+' h';
    if(h<24*70) return (h/24).toFixed(0)+' days';
    if(h<24*365*2.5) return (h/24/30.44).toFixed(1)+' months';
    return (h/24/365.25).toFixed(1)+' years';
  }

  /* ---------- shared physics ----------------------------------------------
     Fresh-pair fidelity is the linear fit through Fig. 3b, F0 = 0.95 - 0.9a,
     and the heralding rate is the exactly linear 0.1309 cps per % of alpha
     that Fig. 3b's top axis gives. The decay is the paper's two components:
     <XX> exponential with the measured 550 ms constant, <ZZ> linear from
     accumulated decoupling-gate errors. X0, Z0 and the <ZZ> slope were fitted
     to the four numbers the paper states at alpha = 17%: F(450 ms) = 0.554,
     the 0.5 crossing at 547 ms, and the two probability-weighted averages
     0.668 (450 ms window) and 0.578 (no window). The fit reproduces the last
     two to 0.001.                                                          */
  var X0=0.84, Z0=0.52, KZZ=2.80e-4, NORM=Z0+2*X0, TAU0=550, RPP=0.1309;
  function F0(a){ return 0.95-0.009*a; }                 /* a in percent */
  function rate(a){ return RPP*a; }                      /* Hz */
  function decay(t,tau){
    return (Math.max(0,Z0-KZZ*t)+2*X0*Math.exp(-t/tau))/NORM;
  }
  function Fof(t,a,tau){ return 0.25+(F0(a)-0.25)*decay(t,tau); }
  function wFid(a,tau,win){                              /* E[F], t ~ Exp(R) */
    var R=rate(a)/1000, hi=win?win:15/R, n=900, dt=hi/n, num=0, den=0;
    for(var i=0;i<n;i++){
      var t=(i+0.5)*dt, w=R*Math.exp(-R*t)*dt;
      num+=w*Fof(t,a,tau); den+=w;
    }
    return num/den;
  }
  function thresh(a){ var x=0.25/(F0(a)-0.25); return x/(1-x); }
  function pOf(F){ return Math.max(0,(4*F-1)/3); }
  function swapF(F1,F2){ return (1+3*pOf(F1)*pOf(F2))/4; }

  /* ---------- 1. the race ---------- */
  (function(){
    var H=document.getElementById('qw-race');
    if(!H) return;
    var sa=H.querySelector('#qwr-a'), st=H.querySelector('#qwr-t'), g=H.querySelector('#qwr-plot');
    function plot(a,tau){
      clear(g);
      var X0p=46, X1p=590, YB=182, YT=20;
      var R=rate(a)/1000, mean=1/R;
      var TMAX=Math.min(8000,Math.max(1200,2.4*mean,2.2*tau));
      function cx(t){ return X0p+t/TMAX*(X1p-X0p); }
      function cy(f){ return YB-(f-0.25)/0.75*(YB-YT); }
      [0.25,0.4,0.6,0.8,1.0].forEach(function(v){
        g.appendChild(el('line',{x1:X0p,y1:cy(v),x2:X1p,y2:cy(v),stroke:'#2a2a2a','stroke-width':1}));
        g.appendChild(el('text',{x:X0p-6,y:cy(v)+3.5,'text-anchor':'end',fill:'#7d7d7d','font-size':9},v.toFixed(2)));
      });
      g.appendChild(el('line',{x1:X0p,y1:cy(0.5),x2:X1p,y2:cy(0.5),stroke:'#e2603f','stroke-width':1.2,'stroke-dasharray':'5 4'}));
      g.appendChild(el('text',{x:X1p,y:cy(0.5)-5,'text-anchor':'end',fill:'#e2603f','font-size':9},'separable below 0.5'));
      var ticks=4, i;
      for(i=0;i<=ticks;i++){
        var tv=TMAX*i/ticks;
        g.appendChild(el('text',{x:cx(tv),y:YB+15,'text-anchor':'middle',fill:'#7d7d7d','font-size':9},
          tv>=1000?(tv/1000).toFixed(1)+'s':Math.round(tv)+'ms'));
      }
      g.appendChild(el('text',{x:(X0p+X1p)/2,y:YB+31,'text-anchor':'middle',fill:'#8f8f8f','font-size':10},'time since the first link succeeded'));
      /* waiting-time CDF, right axis 0..1 mapped onto 0.25..1.0 */
      var d2='';
      for(i=0;i<=160;i++){
        var t=TMAX*i/160, c=1-Math.exp(-R*t);
        d2+=(i?'L':'M')+cx(t).toFixed(1)+' '+cy(0.25+0.75*c).toFixed(1);
      }
      g.appendChild(el('path',{d:d2,fill:'none',stroke:'#6ba3f0','stroke-width':1.8,'stroke-opacity':0.85}));
      g.appendChild(el('text',{x:X1p+4,y:cy(1.0)+3,fill:'#6ba3f0','font-size':9},'100%'));
      g.appendChild(el('text',{x:X1p+4,y:cy(0.25)+3,fill:'#6ba3f0','font-size':9},'0%'));
      /* fidelity decay */
      var d1='';
      for(i=0;i<=200;i++){
        var t2=TMAX*i/200;
        d1+=(i?'L':'M')+cx(t2).toFixed(1)+' '+cy(Fof(t2,a,tau)).toFixed(1);
      }
      g.appendChild(el('path',{d:d1,fill:'none',stroke:'#e2603f','stroke-width':2.4,'stroke-linecap':'round'}));
      /* mean wait marker */
      if(mean<TMAX){
        g.appendChild(el('line',{x1:cx(mean),y1:YT,x2:cx(mean),y2:YB,stroke:'#4a4a4a','stroke-width':1,'stroke-dasharray':'4 4'}));
        g.appendChild(el('text',{x:cx(mean)+4,y:YT+10,fill:'#8f8f8f','font-size':9},'average wait'));
        g.appendChild(el('circle',{cx:cx(mean),cy:cy(Fof(mean,a,tau)),r:4,fill:'#fff',stroke:'#e2603f','stroke-width':2}));
      }
      /* expected fidelity at swap time */
      var EF=wFid(a,tau);
      g.appendChild(el('line',{x1:X0p,y1:cy(EF),x2:X1p,y2:cy(EF),stroke:'#32c29e','stroke-width':1.6}));
      g.appendChild(el('text',{x:X1p,y:cy(EF)+(EF>0.62?15:-6),'text-anchor':'end',fill:'#32c29e','font-size':9.5},'expected fidelity at swap time, '+EF.toFixed(3)));
      return EF;
    }
    function upd(){
      var a=+sa.value, tau=+st.value;
      var R=rate(a), gam=1000/tau, eta=R/gam, th=thresh(a);
      var EF=plot(a,tau), fresh=F0(a), sw=swapF(EF,fresh);
      H.querySelector('#qwr-av').textContent=a.toFixed(1)+'%';
      H.querySelector('#qwr-tv').textContent=ms(tau);
      H.querySelector('#qwr-r').textContent=R.toFixed(3)+' Hz';
      H.querySelector('#qwr-w').textContent=ms(1000/R);
      H.querySelector('#qwr-e').textContent=eta.toFixed(2);
      H.querySelector('#qwr-th').textContent=th.toFixed(2);
      H.querySelector('#qwr-f').textContent=EF.toFixed(3);
      var v=H.querySelector('#qwr-v'), txt, col;
      if(eta<th){ txt='losing the race'; col='#e2603f'; }
      else if(sw<0.5){ txt='wins the race, fails the swap'; col='#f2a03d'; }
      else { txt='a repeater segment'; col='#32c29e'; }
      v.textContent=txt; v.style.color=col;
      H.querySelector('#qwr-note').innerHTML=
        'At <b>&alpha; = '+a.toFixed(1)+'%</b> the link heralds every <b>'+ms(1000/R)+'</b> on average and the pair it made starts at fidelity <b>'+
        fresh.toFixed(3)+'</b>. Waiting the random interval to the next success leaves it at <b>'+EF.toFixed(3)+'</b>. Link efficiency <b>'+eta.toFixed(2)+
        '</b> against a threshold of <b>'+th.toFixed(2)+'</b>, and that threshold is not a constant. It is <i>x</i>/(1&minus;<i>x</i>) with '+
        '<i>x</i> = 0.25/(F<sub>0</sub>&minus;0.25), so turning &alpha; up moves the bar as well as the jump. '+
        'Swapping this stored pair against a fresh one gives <b>'+sw.toFixed(3)+'</b> under the standard depolarizing composition'+
        (sw<0.5?', which is below the separability boundary.':', which clears it.')+
        ' The paper&rsquo;s own point is &alpha; = 17%: 2.226 Hz, 550 ms, expected fidelity 0.578.';
    }
    sa.addEventListener('input',function(){ offGroup(H,'ps'); upd(); });
    st.addEventListener('input',function(){ offGroup(H,'ps'); upd(); });
    btnGroup(H,'ps',function(v){ sa.value=v; st.value=550; upd(); });
    upd();
  })();

  /* ---------- 2. the alpha knob and the error budget ---------- */
  (function(){
    var H=document.getElementById('qw-alpha');
    if(!H) return;
    /* Infidelity budget. The protocol term is 0.9a, which is the slope of the
       Fig. 3b fit. The ion, photon and magnetic terms are read off Fig. 3c and
       rescaled so the total matches the paper's stated fidelities: 0.923 at
       10 km and alpha = 2.5%, and just above 0.90 at 101 km and alpha = 5%.  */
    var ION=0.030, PHO=0.017, DL=8e-5, MAG=2.5e-3;
    var sa=H.querySelector('#qwa-a'), sl=H.querySelector('#qwa-l'), echo=1;
    var gS=H.querySelector('#qwa-stack'), gC=H.querySelector('#qwa-curve');
    function parts(a,L,e){
      return [0.009*a, ION+DL*L*0.45, PHO+DL*L*0.55, e?0:MAG*L];
    }
    function fid(a,L,e){
      var p=parts(a,L,e);
      return Math.max(0.25,1-(p[0]+p[1]+p[2]+p[3]));
    }
    function stack(a,L,e){
      clear(gS);
      var X0p=36,X1p=306,YB=150,YT=16,LMAX=120,HI=e?0.30:0.45;
      var COL=['#e2603f','#f2a03d','#6ba3f0','#8f8f8f'];
      function cx(v){ return X0p+v/LMAX*(X1p-X0p); }
      function cy(v){ return YB-v/HI*(YB-YT); }
      [0,0.1,0.2,0.3,0.4].forEach(function(v){
        if(v>HI) return;
        gS.appendChild(el('line',{x1:X0p,y1:cy(v),x2:X1p,y2:cy(v),stroke:'#2a2a2a','stroke-width':1}));
        gS.appendChild(el('text',{x:X0p-5,y:cy(v)+3.5,'text-anchor':'end',fill:'#7d7d7d','font-size':9},v.toFixed(1)));
      });
      var cum=[0,0,0,0], k, i, base=[];
      for(k=0;k<4;k++){
        var d='', dback='';
        for(i=0;i<=60;i++){
          var x=LMAX*i/60, p=parts(a,x,e), lo=0;
          for(var j=0;j<k;j++) lo+=p[j];
          var hiv=lo+p[k];
          d+=(i?'L':'M')+cx(x).toFixed(1)+' '+cy(Math.min(hiv,HI)).toFixed(1);
        }
        for(i=60;i>=0;i--){
          var x2=LMAX*i/60, p2=parts(a,x2,e), lo2=0;
          for(var j2=0;j2<k;j2++) lo2+=p2[j2];
          d+='L'+cx(x2).toFixed(1)+' '+cy(Math.min(lo2,HI)).toFixed(1);
        }
        gS.appendChild(el('path',{d:d+'Z',fill:COL[k],'fill-opacity':0.62,stroke:COL[k],'stroke-width':0.8}));
      }
      [0,50,100].forEach(function(t){
        gS.appendChild(el('text',{x:cx(t),y:YB+14,'text-anchor':'middle',fill:'#7d7d7d','font-size':9},t+' km'));
      });
      gS.appendChild(el('text',{x:(X0p+X1p)/2,y:YB+30,'text-anchor':'middle',fill:'#8f8f8f','font-size':10},'link length'));
      gS.appendChild(el('text',{x:X0p-5,y:YT-5,'text-anchor':'end',fill:'#8f8f8f','font-size':9},'1 − F'));
      var tot=1-fid(a,L,e);
      gS.appendChild(el('line',{x1:cx(L),y1:YT,x2:cx(L),y2:YB,stroke:'#fff','stroke-width':1,'stroke-opacity':0.35}));
      gS.appendChild(el('circle',{cx:cx(L),cy:cy(Math.min(tot,HI)),r:4,fill:'#fff','fill-opacity':0.92}));
    }
    function curve(a,L,e){
      clear(gC);
      var X0p=40,X1p=306,YB=150,YT=16;
      function cx(f){ return X0p+(f-0.72)/0.26*(X1p-X0p); }
      function cy(r){ return YB-r/3.0*(YB-YT); }
      [0,1,2,3].forEach(function(v){
        gC.appendChild(el('line',{x1:X0p,y1:cy(v),x2:X1p,y2:cy(v),stroke:'#2a2a2a','stroke-width':1}));
        gC.appendChild(el('text',{x:X0p-5,y:cy(v)+3.5,'text-anchor':'end',fill:'#7d7d7d','font-size':9},v));
      });
      [0.75,0.85,0.95].forEach(function(v){
        gC.appendChild(el('text',{x:cx(v),y:YB+14,'text-anchor':'middle',fill:'#7d7d7d','font-size':9},v.toFixed(2)));
      });
      gC.appendChild(el('text',{x:(X0p+X1p)/2,y:YB+30,'text-anchor':'middle',fill:'#8f8f8f','font-size':10},'fresh Bell-state fidelity'));
      gC.appendChild(el('text',{x:X0p-5,y:YT-5,'text-anchor':'end',fill:'#8f8f8f','font-size':9},'Hz'));
      var d='', i;
      for(i=10;i<=220;i++){
        var av=i/10, f=fid(av,L,e);
        if(f<0.72||f>0.98) continue;
        d+=(d?'L':'M')+cx(f).toFixed(1)+' '+cy(rate(av)).toFixed(1);
      }
      gC.appendChild(el('path',{d:d,fill:'none',stroke:'#32c29e','stroke-width':2.4,'stroke-linecap':'round'}));
      /* the decoherence rate line: above it, the link wins the race */
      gC.appendChild(el('line',{x1:X0p,y1:cy(1.8),x2:X1p,y2:cy(1.8),stroke:'#f2a03d','stroke-width':1.2,'stroke-dasharray':'5 4'}));
      gC.appendChild(el('text',{x:X1p,y:cy(1.8)+13,'text-anchor':'end',fill:'#f2a03d','font-size':9},'decoherence rate, 1.8 Hz'));
      [[2.5,'2.5%'],[17,'17%']].forEach(function(m){
        var f=fid(m[0],L,e);
        if(f<0.72||f>0.98) return;
        gC.appendChild(el('circle',{cx:cx(f),cy:cy(rate(m[0])),r:3,fill:'#8f8f8f'}));
        gC.appendChild(el('text',{x:cx(f)+6,y:cy(rate(m[0]))+3,fill:'#8f8f8f','font-size':9},m[1]));
      });
      var fc=fid(a,L,e);
      if(fc>=0.72&&fc<=0.98)
        gC.appendChild(el('circle',{cx:cx(fc),cy:cy(rate(a)),r:4.5,fill:'#fff',stroke:'#32c29e','stroke-width':2}));
    }
    function upd(){
      var a=+sa.value, L=+sl.value, f=fid(a,L,echo), p=parts(a,L,echo);
      var share=p[0]/(p[0]+p[1]+p[2]+p[3]);
      H.querySelector('#qwa-av').textContent=a.toFixed(1)+'%';
      H.querySelector('#qwa-lv').textContent=L+' km';
      H.querySelector('#qwa-f').textContent=f.toFixed(3);
      H.querySelector('#qwa-r').textContent=rate(a).toFixed(3)+' Hz';
      H.querySelector('#qwa-s').textContent=(100*share).toFixed(0)+'%';
      var v=H.querySelector('#qwa-v'), txt, col;
      if(f>=0.90){ txt='everything, device-independent protocols included'; col='#32c29e'; }
      else if(f>=0.78){ txt='violates a Bell inequality, no margin'; col='#d8c257'; }
      else if(f>0.5){ txt='entangled, but no Bell violation'; col='#f2a03d'; }
      else { txt='separable'; col='#e2603f'; }
      v.textContent=txt; v.style.color=col;
      stack(a,L,echo); curve(a,L,echo);
      H.querySelector('#qwa-note').innerHTML=
        'At <b>&alpha; = '+a.toFixed(1)+'%</b> over <b>'+L+' km</b> the fresh pair sits at <b>'+f.toFixed(3)+'</b> and the link heralds at <b>'+
        rate(a).toFixed(3)+' Hz</b>. The protocol&rsquo;s own double-excitation term is <b>'+(100*share).toFixed(0)+
        '% of the infidelity</b>, and it is the only contributor that moves when you change the rate. '+
        (echo? 'With spin-echo decoupling on, going from 10 to 100 km costs about seven thousandths of fidelity: <b>distance is nearly free</b>.'
              : 'With the decoupling off, magnetic-field noise dominates everything past about 20 km and the 100 km link falls to roughly 0.65. <b>This is what the decoupling is buying.</b>')+
        ' The budget is the paper&rsquo;s Fig. 3c, rescaled so the totals match the fidelities it states: 0.923 at 10 km and &alpha; = 2.5%, and just above 0.90 at 101 km and &alpha; = 5%.';
    }
    sa.addEventListener('input',function(){ offGroup(H,'ps'); upd(); });
    sl.addEventListener('input',function(){ offGroup(H,'ps'); upd(); });
    btnGroup(H,'se',function(v){ echo=+v; upd(); });
    btnGroup(H,'ps',function(v){
      var p=v.split('|'); sa.value=p[0]; sl.value=p[1];
      echo=1; setGroup(H,'se','1'); upd();
    });
    upd();
  })();

  /* ---------- 3. the finite-key cliff ---------- */
  (function(){
    var H=document.getElementById('qw-key');
    if(!H) return;
    /* Asymptotic device-independent bound, r = 1 - h(Q) - h((1+sqrt(S^2/4-1))/2),
       which reproduces both of the paper's stated numbers: 0.326 against 0.325
       at 10 km, 0.0989 against 0.0974 at 101 km. The two finite-size penalties
       are single-constant 1/sqrt(N) forms pinned to the paper's own landmarks:
       1,917 bits from 405,145 rounds for the complementarity analysis, and a
       zero crossing at the 1.5e6 rounds it says entropy accumulation needs.  */
    var EPS=Math.log(1e5)/Math.LN2, C_COMP=50.175, C_EAT=97.96;
    var LINK={ '10':{S:2.5758,Q:0.0360,lbl:'10 km'}, '101':{S:2.504,Q:0.069,lbl:'101 km'} };
    var link='10', sn=H.querySelector('#qwk-n'), g=H.querySelector('#qwk-plot'), curN=405145;
    var R_DI=0.291, DUTY=0.264;
    function h2(x){ if(x<=0||x>=1) return 0; return -x*log2(x)-(1-x)*log2(1-x); }
    function rInf(L){ return 1-h2(L.Q)-h2((1+Math.sqrt(Math.max(L.S*L.S/4-1,0)))/2); }
    function rFin(L,N,C){ return Math.max(0,rInf(L)-C*Math.sqrt(EPS/N)); }
    function zero(L,C){ var v=C*C*EPS/(rInf(L)*rInf(L)); return v; }
    function plot(N){
      clear(g);
      var L=LINK[link], X0p=54,X1p=596,YB=196,YT=20, LO=-4, HI=0;
      function cx(l){ return X0p+(l-4)/4*(X1p-X0p); }
      function cy(r){ var l=Math.log(Math.max(r,1e-5))/Math.LN10; return YB-(l-LO)/(HI-LO)*(YB-YT); }
      [0,-1,-2,-3,-4].forEach(function(e){
        g.appendChild(el('line',{x1:X0p,y1:cy(Math.pow(10,e)),x2:X1p,y2:cy(Math.pow(10,e)),stroke:'#2a2a2a','stroke-width':1}));
        var t=g.appendChild(el('text',{x:X0p-6,y:cy(Math.pow(10,e))+3.5,'text-anchor':'end',fill:'#7d7d7d','font-size':9}));
        t.appendChild(document.createTextNode('10'));
        var s=el('tspan',{dy:'-4','font-size':7}); s.appendChild(document.createTextNode(String(e))); t.appendChild(s);
      });
      [4,5,6,7,8].forEach(function(e){
        g.appendChild(el('line',{x1:cx(e),y1:YT,x2:cx(e),y2:YB,stroke:'#232323','stroke-width':1}));
        var t=g.appendChild(el('text',{x:cx(e),y:YB+15,'text-anchor':'middle',fill:'#7d7d7d','font-size':9}));
        t.appendChild(document.createTextNode('10'));
        var s=el('tspan',{dy:'-4','font-size':7}); s.appendChild(document.createTextNode(String(e))); t.appendChild(s);
      });
      g.appendChild(el('text',{x:(X0p+X1p)/2,y:YB+31,'text-anchor':'middle',fill:'#8f8f8f','font-size':10},'rounds collected'));
      g.appendChild(el('text',{x:X0p-6,y:YT-6,'text-anchor':'end',fill:'#8f8f8f','font-size':9},'bits/round'));
      var ri=rInf(L);
      g.appendChild(el('line',{x1:X0p,y1:cy(ri),x2:X1p,y2:cy(ri),stroke:'#32c29e','stroke-width':2}));
      g.appendChild(el('text',{x:X1p,y:cy(ri)-6,'text-anchor':'end',fill:'#32c29e','font-size':9.5},'asymptotic, '+ri.toFixed(3)));
      [[C_COMP,'#f2a03d'],[C_EAT,'#6ba3f0']].forEach(function(c){
        var d='', started=false;
        for(var i=0;i<=240;i++){
          var lg=4+4*i/240, r=rFin(L,Math.pow(10,lg),c[0]);
          if(r<1e-5){ started=false; continue; }
          d+=(started?'L':'M')+cx(lg).toFixed(1)+' '+cy(r).toFixed(1); started=true;
        }
        if(d) g.appendChild(el('path',{d:d,fill:'none',stroke:c[1],'stroke-width':2.2,'stroke-linecap':'round'}));
      });
      /* landmarks */
      var marks=[[405145,'their run',rFin(L,405145,C_COMP),'#f2a03d']];
      if(link==='10') marks.push([1.2e6,'Science, single atoms',0.112,'#8f8f8f']);
      marks.forEach(function(m){
        if(m[2]<1e-5) return;
        var lg=Math.log(m[0])/Math.LN10;
        g.appendChild(el('circle',{cx:cx(lg),cy:cy(m[2]),r:3.6,fill:'#fff',stroke:m[3],'stroke-width':1.6}));
        g.appendChild(el('text',{x:cx(lg),y:cy(m[2])+17,'text-anchor':'middle',fill:m[3],'font-size':9},m[1]));
      });
      var lgn=Math.log(N)/Math.LN10;
      g.appendChild(el('line',{x1:cx(lgn),y1:YT,x2:cx(lgn),y2:YB,stroke:'#fff','stroke-width':1,'stroke-opacity':0.3}));
      var rc=rFin(L,N,C_COMP);
      if(rc>1e-5) g.appendChild(el('circle',{cx:cx(lgn),cy:cy(rc),r:4.5,fill:'#fff',stroke:'#f2a03d','stroke-width':2}));
    }
    function upd(){
      var N=curN, L=LINK[link];
      var ri=rInf(L), rc=rFin(L,N,C_COMP), bits=N*rc;
      var hval=N/R_DI/3600;
      H.querySelector('#qwk-nv').textContent=N.toLocaleString();
      H.querySelector('#qwk-ra').textContent=ri.toFixed(3);
      H.querySelector('#qwk-rc').textContent=rc>0?(rc<0.01?rc.toExponential(1):rc.toFixed(3)):'no key';
      H.querySelector('#qwk-b').textContent=bits>=1?Math.round(bits).toLocaleString():'0';
      H.querySelector('#qwk-t').textContent=dur(hval/DUTY);
      var v=H.querySelector('#qwk-v'), txt, col;
      if(rc<=0){ txt='not enough rounds for any key at all'; col='#e2603f'; }
      else if(rc<0.05*ri){ txt='a key, and 95% of it eaten by statistics'; col='#f2a03d'; }
      else if(rc<0.5*ri){ txt='the penalty is finally reasonable'; col='#d8c257'; }
      else { txt='converged, statistics no longer the problem'; col='#32c29e'; }
      v.textContent=txt; v.style.color=col;
      plot(N);
      var zc=zero(L,C_COMP), ze=zero(L,C_EAT);
      H.querySelector('#qwk-note').innerHTML=
        'Over <b>'+L.lbl+'</b> with S = '+L.S+' and QBER '+(100*L.Q).toFixed(2)+'%, the asymptotic rate is <b>'+ri.toFixed(3)+
        ' bits per round</b>. At <b>'+N.toLocaleString()+' rounds</b> the complementarity analysis leaves <b>'+
        (rc>0?(rc<0.01?rc.toExponential(1):rc.toFixed(3))+' bits per round</b>, or '+Math.round(bits).toLocaleString()+' secret bits'
             :'nothing at all</b>')+
        '. The first positive key appears at about <b>'+Math.round(zc/1000).toLocaleString()+
        ',000 rounds</b> for this analysis and <b>'+(ze/1e6).toFixed(1)+' million</b> for entropy accumulation. '+
        'At the measured 0.291 rounds per second and the ~26% duty cycle their two-month run implies, that is <b>'+
        dur(zc/R_DI/3600/DUTY)+'</b> and <b>'+dur(ze/R_DI/3600/DUTY)+'</b> of wall clock respectively.'+
        (link==='101'?' <b>Nobody has run this experiment.</b> The 101 km link has 2,799 rounds and no finite-size analysis in the paper.':'');
    }
    sn.addEventListener('input',function(){ curN=Math.round(Math.pow(10,(+sn.value)/100)); offGroup(H,'ps'); upd(); });
    btnGroup(H,'ps',function(v){ curN=+v; sn.value=Math.round(100*Math.log(curN)/Math.LN10); upd(); });
    btnGroup(H,'ln',function(v){ link=v; upd(); });
    upd();
  })();
})();
</script>{% endraw %}