---
layout: post
section-type: post
title: "A quantum LAN on deployed fiber : what leaving the optical table costs"
category: 'networking'
tags: [ 'networking', 'fiber', 'quantum' ]
---

[the flex grid post]({% post_url 2026-04-04-Flex-grid-for-entanglement %}) was about a box. one wavelength selective switch in the middle of an entanglement network, replacing a tree of passive filters, so that who-talks-to-whom becomes a config push instead of a trip to the rack.

everything in that paper was in one room. one optical table, one time-to-digital converter, one clock, four users who were four fiber pigtails coiled a meter apart.

this post is about the paper that took the same architecture outside. [Alshowkan et al., "Reconfigurable Quantum Local Area Network Over Deployed Fiber," *PRX Quantum* 2, 040304 (2021)](https://doi.org/10.1103/PRXQuantum.2.040304). three buildings on the Oak Ridge campus, 250 m and 1200 m of already-installed fiber, a GPS antenna, and eight reconfigurable channels of polarization entanglement.

the physics is unchanged. same PPLN waveguide, same |Ψ⁺⟩, same kind of switch. what changed is that four things a tabletop hands you for free stopped being free:

- **a clock.** on a table every detector shares one. across a campus they do not.
- **the fiber.** on a table it is a patch cord you chose this morning. across a campus it is whatever got pulled years ago, through however many communication rooms.
- **the detectors.** you buy three identical ones for a table. across an organization you inherit whatever each group already owns.
- **a number that means something.** coincidence rate is fine when every link has the same budget, and stops being comparable the moment they do not.

the paper reports enough to price all four. and one of them turns out to dominate so completely that you can take the paper's own two tables, do about six lines of arithmetic, and predict the results of the follow-up paper published a year later.

below: the network, what each of those four cost, and then the critique and what i think is worth doing next.

## the network

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Node</th><th>Where</th><th>Fiber to source</th><th>Detector</th><th>Efficiency</th></tr>
</thead>
<tbody>
<tr><td>Alice</td><td>with the source</td><td>0 m</td><td>SNSPD</td><td>&ge; 80%</td></tr>
<tr><td>Bob</td><td>second building</td><td>~250 m, 1.8 dB panel to panel</td><td>InGaAs APD, gated</td><td>~20%</td></tr>
<tr><td>Charlie</td><td>third building</td><td>~1200 m, 3.3 dB panel to panel</td><td>SNSPD</td><td>&ge; 80%</td></tr>
</tbody>
</table>
</div>

the source is a 10 mm type-II PPLN ridge waveguide pumped by a CW diode at 779.4 nm, giving pairs across roughly 310 GHz (2.5 nm FWHM) centered on 192.3125 THz. a wavelength selective switch carves that into eight pairs of frequency-correlated 25 GHz channels sitting on the ITU 25 GHz grid, ITU 21.25 through 25.00, occupying about 3 nm from 1557.3 to 1560.5 nm.

the addressing rule is the one from the last post, and it is the whole reason a switch works here:

```
    ω_signal  +  ω_idler  =  ω_pump
```

hand two users energy-matched slices and they share entanglement. hand them mismatched slices and they share nothing. channel *n* sits at `ω₀ + Δω(n − ½)` on the signal side and the same distance below on the idler side, so channel 1 is closest to degeneracy and channel 8 is furthest out in the wings.

that last detail matters more than it looks. the pair spectrum has a sinc-squared envelope, and the paper puts the 50% point inside channel 7. so **the eight channels are not eight equal resources.** channel 1 carries essentially the full flux and channel 8 carries about a third of it. the allocator is dividing a hill, not a rectangle.

measured locally at the source, before any of it goes outside, the eight channel pairs have fidelities to |Ψ⁺⟩ between 0.935 and 0.952, and a coincidence-to-accidental ratio of 11.4.

hold onto that 0.94. it is the number the network starts with.

## the clock

every node timestamps its own detections. a detection on its own means nothing, because dark counts and stray light click too. what you want is a coincidence: Alice clicked and Bob clicked close enough together in time that they probably came from the same pair.

"close enough together" is a window you have to choose, and choosing it requires the two nodes to agree on what time it is.

on a table this is trivial. one TDC, one clock, one cable. across three buildings it is a distributed systems problem with a physics deadline. the paper's numbers make the deadline concrete: the FPGA-based TDCs bin at 5 ns using a 200 MHz internal clock, and **the detector-limited coincidence peaks are less than 1 ns wide.** so a photon pair is a sub-nanosecond event, and everything above that in your window is a pure noise tax.

the paper's solution is GPS. a Trimble Thunderbolt E receiver at each node, disciplining a 10 MHz clock and a pulse-per-second. only one building had an antenna already, so the GPS RF is distributed to the other two over dedicated strands of the deployed fiber with commercial RF-over-fiber transmitters and receivers, which is a nice piece of pragmatism.

then they measured what that buys. histograms of the relative delay between pairs of receivers, recorded over several hours:

- Alice to Bob: **σ = 12.1 ns**
- Charlie to Alice: **σ = 14.7 ns**

that is the wander of the agreed-upon zero. the coincidence peak is under a nanosecond wide, and its *position* drifts by more than ten. so the window has to be wide enough to keep a narrow peak from walking out of it, which the paper reasons out as "coincidence windows up to approximately 30 ns would be reasonable," roughly 2σ.

they chose 10 ns.

that is a deliberate under-window. they gave up some correlated detections to avoid admitting three times as much noise. and it sets everything downstream of it, because accidental coincidences scale linearly with the window while real ones do not scale at all.

### how much noise that let in

the paper does something unusually honest here. it reports every result twice: once raw, and once with accidentals subtracted before tomography, where the accidentals are taken from a 10 ns window time-shifted off the correlated peak.

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Allocation</th><th>Link</th><th>Ch.</th><th>Fidelity, raw</th><th>Fidelity, accidentals removed</th></tr>
</thead>
<tbody>
<tr><td>1</td><td>A&ndash;B</td><td>1</td><td>0.75 &plusmn; 0.03</td><td>0.960 &plusmn; 0.005</td></tr>
<tr><td>1</td><td>B&ndash;C</td><td>2&ndash;7</td><td>0.55 &plusmn; 0.06</td><td>0.925 &plusmn; 0.006</td></tr>
<tr><td>1</td><td>C&ndash;A</td><td>8</td><td>0.90 &plusmn; 0.01</td><td>0.985 &plusmn; 0.001</td></tr>
<tr><td>2</td><td>A&ndash;B</td><td>3</td><td>0.75 &plusmn; 0.03</td><td>0.957 &plusmn; 0.004</td></tr>
<tr><td>2</td><td>B&ndash;C</td><td>1&ndash;2</td><td>0.69 &plusmn; 0.04</td><td>0.959 &plusmn; 0.005</td></tr>
<tr><td>2</td><td>C&ndash;A</td><td>4</td><td>0.84 &plusmn; 0.02</td><td>0.968 &plusmn; 0.001</td></tr>
</tbody>
</table>
</div>

the two columns differ by up to 37 points of fidelity. that gap is the deployment tax, and almost all of it is the clock.

you can put a number on it. accidentals are unpolarized, so as far as tomography is concerned they are a maximally mixed background, which contributes 0.25 to the fidelity. if `a` is the number of accidentals per true coincidence, then

```
    F_raw  =  ( F_true  +  0.25 · a )  /  ( 1 + a )
```

which rearranges to `a = (F_true − F_raw) / (F_raw − 0.25)`. run the six rows through it:

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Allocation</th><th>Link</th><th>Ch.</th><th>Accidentals per true pair, at 10 ns</th></tr>
</thead>
<tbody>
<tr><td>1</td><td>A&ndash;B</td><td>1</td><td>0.42</td></tr>
<tr><td>1</td><td>B&ndash;C</td><td>2&ndash;7</td><td><b>1.25</b></td></tr>
<tr><td>1</td><td>C&ndash;A</td><td>8</td><td>0.13</td></tr>
<tr><td>2</td><td>A&ndash;B</td><td>3</td><td>0.41</td></tr>
<tr><td>2</td><td>B&ndash;C</td><td>1&ndash;2</td><td>0.61</td></tr>
<tr><td>2</td><td>C&ndash;A</td><td>4</td><td>0.22</td></tr>
</tbody>
</table>
</div>

on the worst link in allocation 1, **there is more noise than signal**. fewer than half of what Bob and Charlie call a coincidence is one. that link still has non-zero entanglement, which the paper is careful to point out is three standard deviations clear of zero, but it is doing it on 44% of its own traffic.

and every one of those numbers is proportional to the window. halve the window, halve the noise.

<div class="qw" id="qw-win" data-qw><div class="qw-hd"><span class="qw-t">the coincidence window is a noise budget</span><span class="qw-s">accidentals scale with the window. real pairs do not. the only thing stopping you from closing it is how well your two nodes agree on what time it is.</span><span class="qw-lg"><i style="background:#32c29e"></i>true coincidence peak<i style="background:#f2a03d"></i>accidental floor<i style="background:#5b8def"></i>window you keep</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">clock</span><span class="qw-b" data-grp="clk"><button type="button" data-v="12.1" class="on" aria-pressed="true">GPS across the campus, 12.1 ns</button><button type="button" data-v="1.21">GPS, both receivers on one bench, 1.21 ns</button><button type="button" data-v="0.0129">White Rabbit, 12.9 ps</button><button type="button" data-v="0.003">mode-locked laser sync, 3 ps</button></span></div></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">link, as allocated in the paper</span><span class="qw-b" data-grp="lnk"><button type="button" data-v="0">A&ndash;B, alloc 2, ch 3</button><button type="button" data-v="1">B&ndash;C, alloc 2, ch 1&ndash;2</button><button type="button" data-v="2">C&ndash;A, alloc 2, ch 4</button><button type="button" data-v="3" class="on" aria-pressed="true">B&ndash;C, alloc 1, ch 2&ndash;7</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">coincidence window <span class="qw-gk">&tau;</span> <b id="qww-tv">10.0 ns</b></span><input type="range" id="qww-t" min="0" max="90" step="0.5" value="75"></label></div><div class="qw-two"><div class="qw-pane"><span class="qw-pl">what lands inside the window</span><svg class="qw-svg" viewBox="0 0 320 170" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Coincidence histogram with the accepted window drawn on it"><g id="qww-hist"></g></svg></div><div class="qw-pane"><span class="qw-pl">measured fidelity vs window</span><svg class="qw-svg" viewBox="0 0 320 170" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Measured fidelity as a function of coincidence window width"><g id="qww-curve"></g></svg></div></div><div class="qw-out"><div class="qw-oi"><span class="qw-ok">accidentals per true pair</span><span class="qw-ov" id="qww-a">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">fidelity you would measure</span><span class="qw-ov" id="qww-f">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">share of counts that are real</span><span class="qw-ov" id="qww-p">&mdash;</span></div><div class="qw-oi qw-grow"><span class="qw-ok">verdict</span><span class="qw-ov qw-txt" id="qww-v">&mdash;</span></div></div><p class="qw-n" id="qww-note"></p><noscript><p class="qw-n">At the paper&rsquo;s 10 ns window the B&ndash;C link of allocation 1 carries 1.25 accidentals per true pair, and the measured fidelity is 0.55 against an underlying 0.925. Accidentals fall linearly with the window, so a 1 ns window on the same link would give roughly 0.85. The window cannot usefully go below the clock&rsquo;s own wander, which is 12.1 ns for campus GPS and 12.9 ps for White Rabbit.</p></noscript></div>

### the useful thing you can do with that

if the accidentals are proportional to the window, and the underlying state is what it is, then you can ask what these exact links would have measured with a better clock. take the accidentals-subtracted fidelity, scale `a` by the window ratio, and put it back through the mixing formula.

for allocation 2 at a 1 ns window:

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Link</th><th>Measured, 10 ns, GPS</th><th>Predicted at 1 ns</th><th>Measured a year later with White Rabbit</th></tr>
</thead>
<tbody>
<tr><td>A&ndash;B</td><td>0.75</td><td>0.93</td><td>0.938 &plusmn; 0.008</td></tr>
<tr><td>B&ndash;C</td><td>0.69</td><td>0.92</td><td>0.91 &plusmn; 0.01</td></tr>
<tr><td>C&ndash;A</td><td>0.84</td><td>0.95</td><td>0.971 &plusmn; 0.004</td></tr>
</tbody>
</table>
</div>

the right-hand column is [Alshowkan et al., "Advanced architectures for high-performance quantum networking," *JOCN* 14, 493 (2022)](https://opg.optica.org/jocn/abstract.cfm?uri=jocn-14-6-493) ([arXiv](https://arxiv.org/abs/2111.15547)), the same group, same three buildings, same source, GPS replaced by White Rabbit switches and the window closed from 10 ns to 1 ns. their measured timing jitter with White Rabbit was **12.9 ps**, against 1.21 ns for the GPS receivers in a bench comparison, and they quote "an approximately 10-fold reduction in accidental coincidences."

so a one-line noise model, calibrated on nothing but this paper's own two tables, gets the follow-up paper's fidelities to within a couple of points on every link.

i find that worth stating plainly: **the 2021 network was not limited by its source, its fiber, its switch or its detectors. it was limited by GPS.** everything else in the paper is downstream of a decision about what time it is.

two footnotes on that, because the story is less tidy than it sounds.

the 1.21 ns GPS figure in the follow-up was measured with both receivers sitting in the same laboratory. the 12.1 and 14.7 ns figures here were measured between buildings. they are not the same experiment, and quoting them as a before-and-after would be wrong. the honest version is that campus-deployed GPS gave 12 to 15 ns, and a bench shootout of the two technologies gave 1.21 ns against 12.9 ps.

and the ebit rates in the follow-up went *down*, from 57, 26 and 320 to 47, 19.6 and 145. the paper attributes that to extra loss from re-optimizing the setup by hand rather than to anything about the clock. so the White Rabbit upgrade bought fidelity and paid for it in rate, which is a trade the paper does not frame as a trade.

## the fiber

the fiber is the part the paper says least about, which turns out to be the interesting thing.

each user is connected to the WSS by a single strand of deployed campus fiber. it leaves the source lab, hits a patch panel, traverses several communication rooms, and comes out on a panel in the destination building. panel to panel it costs 1.8 dB to Bob and 3.3 dB to Charlie. for 250 m and 1200 m of glass that is almost all connectors and splices, not attenuation.

polarization is the thing that should worry you. the state is encoded in polarization, and single-mode fiber applies an arbitrary and time-varying unitary to it. random birefringence from stress, temperature and handling will rotate |Ψ⁺⟩ into something else on the way, and unlike loss you cannot budget for it once.

the paper's handling of this is one manual fiber polarization controller per user, set by an alignment procedure before each experiment. and this sentence:

> Given the fact that all fibers are located either indoors or underground, we do not find it necessary to perform active polarization tracking; the polarization state is typically stable for hours at a time.

that is a true statement about that campus. it is also the assumption that the following five years spent the most effort demolishing.

the compensation procedure itself is worth reading because it is so hands-on. insert a polarizer after the source so only H gets through, adjust each user's controller to minimize counts in the V setting, which pins H and V up to an unknown relative phase. that leaves the state as `|HV⟩ + e^{iφ}|VH⟩`. then define effective D/A axes by setting each quarter-wave plate to 45 degrees, and tune one half-wave plate to compensate φ. the free parameter shows up again in the measurement settings: there are three of them, which the paper notes is exactly enough to optimize the wave-plate settings for all pairs of nodes at once, though in this experiment they optimize one analyzer per pair.

three parameters for a whole network is a real result buried in a methods paragraph. it means the network-wide polarization compensation problem is small and, in principle, solvable in real time.

nobody did that in 2021. plenty of people have since.

## the detectors you did not choose

Alice and Charlie have superconducting nanowire detectors at 80% or better. Bob has an InGaAs avalanche photodiode at about 20%, with 100 μs of dead time and a 33.5 ns gate running at 15 MHz.

the efficiency ratio is the number people quote, and it understates the problem. that gate is 33.5 ns wide at 15 MHz, so Bob's detector is only listening about half the time. between efficiency, gating, dead time and 1.8 dB of fiber, Bob's singles on the brightest channel come in at 5.0 × 10³ per second against Alice's 1.2 × 10⁵ on the conjugate channel. a factor of 24.

the paper's framing of this is the right one:

> The heterogeneous detectors result in widely varying link efficiencies and reflect the types of variability in quantum resources that should be expected in larger quantum networks.

that is a networking statement, not a physics one. the reason Bob has an APD is not that anyone chose an APD for the experiment. it is that Bob's building had an APD. one 2026 review puts an SNSPD system at around €100,000 and notes that InGaAs SPADs, being room temperature, come with five to ten times the timing jitter and therefore need roughly fourteen times the measurement time for the same timing precision, so an efficiency mismatch quietly becomes a scheduling mismatch.

so the allocator is doing more than dividing spectrum. it is compensating for a purchasing decision made by a different department, and every link touching Bob is worse than the others no matter how the spectrum is cut.

allocation 1 makes this vivid. B–C is the link between the 20% detector and the node 1200 m away, and it is the one that gets six channels out of eight. six channels does not fix it. it makes the fidelity worse.

## a metric that survives the trip

the flex grid paper rated links by coincidence rate, and said in its own conclusion that it wanted something better. this paper is where the something better shows up.

a fidelity number says nothing about how fast, and a rate number says nothing about whether the pairs are any good. neither one tells you whether the link can carry a protocol, which is the question a user has.

what you want is distillable entanglement: how many clean Bell pairs you could squeeze out of the noisy ones, per second. that quantity is the right answer and it is famously hard to compute even when you know the state exactly. so the paper uses **log-negativity**, which upper-bounds it, is easy to compute from a density matrix, and for two qubits is greater than zero exactly when the state is entangled at all. multiply it by the coincidence rate and you get a link rated in **entangled bits per second**:

```
    R_E  =  E_N  ×  (coincidence rate)
```

that is a real unit. it folds quality and quantity into one number that stays comparable across links with completely different budgets.

here is what the two allocations delivered.

<div class="qw" id="qw-alloc" data-qw><div class="qw-hd"><span class="qw-t">two ways to cut the same eight channels</span><span class="qw-s">the pair spectrum is a hill, not a rectangle. channel 1 sits at degeneracy and carries full flux; channel 8 is out at the 35% point. picking channels is picking flux.</span><span class="qw-lg"><i style="background:#32c29e"></i>A&ndash;B<i style="background:#f2a03d"></i>B&ndash;C<i style="background:#5b8def"></i>C&ndash;A<i style="background:#4a4a4a"></i>held in reserve</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">allocation</span><span class="qw-b" data-grp="al"><button type="button" data-v="1" class="on" aria-pressed="true">1 &mdash; balance the rates</button><button type="button" data-v="2">2 &mdash; rescue the worst link</button></span></div><div class="qw-g"><span class="qw-l">numbers</span><span class="qw-b" data-grp="sb"><button type="button" data-v="0" class="on" aria-pressed="true">as measured</button><button type="button" data-v="1">accidentals removed</button></span></div></div><div class="qw-two"><div class="qw-pane"><span class="qw-pl">the eight channel pairs, and who got them</span><svg class="qw-svg" viewBox="0 0 320 175" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Spectrum of eight channel pairs coloured by which link each was assigned to"><g id="qwa-spec"></g></svg></div><div class="qw-pane"><span class="qw-pl">fidelity is not a rate</span><svg class="qw-svg" viewBox="0 0 320 175" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Log-negativity against fidelity for all six measured links"><g id="qwa-plane"></g></svg></div></div><div class="qw-out"><div class="qw-oi qw-grow"><span class="qw-ok">A&ndash;B</span><span class="qw-ov qw-txt" id="qwa-ab">&mdash;</span></div><div class="qw-oi qw-grow"><span class="qw-ok">B&ndash;C</span><span class="qw-ov qw-txt" id="qwa-bc">&mdash;</span></div><div class="qw-oi qw-grow"><span class="qw-ok">C&ndash;A</span><span class="qw-ov qw-txt" id="qwa-ca">&mdash;</span></div></div><p class="qw-n" id="qwa-note"></p><noscript><p class="qw-n">Allocation 1 gives channel 1 to A&ndash;B, channels 2&ndash;7 to B&ndash;C and channel 8 to C&ndash;A, measuring fidelities of 0.75, 0.55 and 0.90 and rates of 56, 30 and 206 ebits/s. Allocation 2 gives channels 1&ndash;2 to B&ndash;C, channel 3 to A&ndash;B and channel 4 to C&ndash;A, leaving four channels unassigned, and measures 0.75, 0.69 and 0.84 at 57, 26 and 320 ebits/s. Giving B&ndash;C three times the spectrum bought about 15% more ebits per second and cost 14 points of fidelity.</p></noscript></div>

the comparison is the point of the paper, and it does not come out tidy.

**B–C.** allocation 1 gives it six channels, 150 GHz. allocation 2 gives it two, 50 GHz. three times the spectrum bought 30 ebits/s against 26, about 15% more, and cost 14 points of fidelity. more bandwidth is more flux, more flux is more multipair emission, and desired coincidences grow linearly with flux while accidentals grow quadratically. so past some point you are buying noise.

**C–A.** allocation 1 puts it on channel 8, out in the wings at about a third of peak flux, and gets fidelity 0.90 at 206 ebits/s. allocation 2 moves it to channel 4 at 81% flux and gets 0.84 at **320 ebits/s**. lower fidelity, 55% more entanglement per second. the paper calls this "the exact opposite behavior" to B–C and leaves it there.

which is the conclusion: **there is no ordering.** you cannot say allocation 2 is better than allocation 1. you can only say which one is better for a stated objective, and the paper says exactly that: "the optimal allocation may depend on a given objective."

that sentence is where quantum networking becomes a service provisioning problem rather than an optics problem.

### the identity the paper noticed and did not explain

there is a smaller thing in there worth pulling out. compare the ebit rates raw and with accidentals subtracted, and they are **equal within error on every link**. the paper flags it:

> While we are unaware of any theoretical requirement for this relationship, intuitively it makes sense: subtracting accidentals eliminates noise from the data, but it does not increase the throughput of entangled pairs.

the intuition is right and the arithmetic is available. `R_E = E_N × C`. subtracting accidentals divides the coincidence rate by `(1 + a)`. so the rates come out equal exactly when log-negativity itself falls off as `1/(1 + a)` under that background. check it on the six links: the measured ratio of raw to subtracted log-negativity is 0.729, 0.421, 0.902, 0.722, 0.612, 0.837, and `1/(1+a)` is 0.704, 0.444, 0.884, 0.707, 0.621, 0.822.

so it is not a coincidence and it is not a deep theorem. in this regime log-negativity is close to linear in the fraction of your counts that are real, which is a small, useful fact: **the ebit rate is insensitive to how you account for accidentals, and that is a good property for a service metric to have.** it means an operator and a customer will agree on the number without arguing about noise subtraction.

## remote state preparation

the last section of the paper runs an actual protocol over the links, which is how you tell a network from a measurement.

remote state preparation is teleportation's cheaper cousin. in teleportation Alice holds an unknown state and has to do a two-photon Bell measurement to send it. in RSP Alice already knows the state she wants Bob to have, so she can just measure her half of the entangled pair in a chosen basis and Bob's half collapses into the state she wanted. one single-photon measurement instead of a Bell measurement, and less classical traffic.

it is not deterministic. sometimes you get the orthogonal outcome, and you either apply a correction or, as here, postselect. but it does the same job as teleportation whenever the sender knows what they are sending, and it is the primitive that feeds blind quantum computing, where you want to load inputs onto an untrusted server without telling it what they are.

the mechanics fall straight out of |Ψ⁺⟩ being maximally correlated in all three bases. project your photon onto |R⟩ and your partner's collapses to |R⟩. so:

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Link</th><th>Prepared</th><th>Fidelity to target</th></tr>
</thead>
<tbody>
<tr><td>Alice &rarr; Bob</td><td>|R&rang;</td><td>0.821 &plusmn; 0.008</td></tr>
<tr><td>Charlie &rarr; Bob</td><td>|V&rang;</td><td>0.80 &plusmn; 0.01</td></tr>
<tr><td>Alice &rarr; Charlie</td><td>|D&rang;</td><td>0.939 &plusmn; 0.003</td></tr>
</tbody>
</table>
</div>

the number that matters is not in that table. it is this: the prepared states agree with what you would predict from the measured density matrices to a fidelity of about **0.99 in all three cases**. so the imperfections are the link's, not the protocol's. RSP is not adding error, it is faithfully reporting the entanglement it was handed.

that is what "the network stack works" looks like when you write it as a measurement. and note the ordering is the ordering of the links: the two hops touching Bob's APD land at 0.80 and 0.82, the SNSPD-to-SNSPD hop lands at 0.94.

## the critique

this is a good paper and the criticisms are mostly about what it claims rather than what it did.

**the headline results are 10 ns wide.** the abstract quantifies link quality in ebits/s and the tables report fidelities of 0.55 to 0.90. those numbers describe a GPS receiver at least as much as they describe a quantum network, and the paper knows it, because the accidentals-subtracted table sits right there showing 0.925 to 0.985. i would rather the paper had led with the noise decomposition instead of putting it in the discussion. the interesting claim is not "we distributed entanglement at fidelity 0.55," it is "we distributed entanglement at fidelity 0.925 and our clock took 37 points off it."

**two allocations is not an exploration.** the flexibility argument is that a WSS lets you provision on demand, and the evidence for it is two hand-picked configurations. neither was produced by any procedure. there is no sweep of channels-per-link against fidelity and rate, which is the one measurement that would turn the observations into a design rule. the multipair scaling argument is stated analytically and then demonstrated at exactly two points.

**the polarization claim is scoped to that campus and does not travel.** "indoors or underground" and "stable for hours" is fine for buildings you own. it is not a property of deployed fiber, it is a property of *those* fibers. every serious deployment since has needed active compensation, and the manual FPC alignment before each experiment is the part of this setup that cannot survive contact with a real service.

**nothing here is running unattended.** wave plates are set by hand or by motion controllers under an alignment procedure, the analyzers are optimized per pair, and the reconfiguration is a human deciding to try allocation 2. the paper describes a control plane for data transfer and instrument control, but the actual decisions are all people. the reconfiguration time, which is the number an operator would ask for first, is never stated.

**tomography is incomplete and the paper says so.** measurements are in H/V and D/A only. that is not tomographically complete, and they rely on Bayesian estimation plus the strong correlations of the input state to get away with it. the error bars are small and the approach is defensible, but circular basis data is missing and the |Ψ⁺⟩ assumption is doing work.

**the metric did not catch on.** ebits/s is the most transferable idea in the paper and five years later it is still mostly an ORNL house metric. Qunnect, Delft, Harvard and the Bristol group all report fidelity, pair rate and CHSH *S* instead. i think ebits/s is the better number and i also think the paper did not do enough to make it adoptable: there is no reference procedure, no statement of how long you integrate, no discussion of how it behaves when the state is not close to Bell diagonal.

and one thing that is not a criticism but reads like one. eight channels across 310 GHz is a small network. the paper is upfront that a broadband source and the full 192-channel WSS would reach nearly 200 nodes. the demonstration is three users because three buildings is what they had, and the fair read is that this is an architecture paper with an existence proof attached.

## what i would do next

the five years since have answered some of this and sharpened the rest.

**1. the clock problem got solved, and then became a security problem.** White Rabbit took the same network from 10 ns to 1 ns and lifted fidelities into the 0.91 to 0.97 range. if you want the argument for why the clock is the hard part on a shared fiber, it is in [the coexistence post]({% post_url 2026-03-23-coexistence-running-a-quantum-channel-and-its-clock-down-the-same-fiber %}). long-haul followed: [unrepeated White Rabbit over 300 km](https://arxiv.org/abs/2511.23254) now holds 4 ps time deviation at 400 s averaging. and you can skip external timing entirely, using the pair correlations themselves as the reference, which [held two clocks within 12 ps over 48 hours on a 50 km link with a 120 ps coincidence window](https://arxiv.org/abs/2501.16796).

but the reason this paper used GPS was convenience, and the reason to move off GPS was jitter. neither is the reason that matters now. GPS is spoofable, and White Rabbit is not obviously better: its authentication rests on computational assumptions, and PTP-family protocols authenticate content without authenticating propagation delay. a group at NUDT has [demonstrated a tunable asymmetric delay attack on quantum clock synchronization over 10 km of fiber](https://arxiv.org/abs/2510.21101), producing sub-nanosecond offsets on demand, and concluded that threshold-based detection cannot stand alone. a follow-up in [Optics Letters](https://opg.optica.org/ol/abstract.cfm?uri=ol-51-14-4028) hides a slow persistent version inside normal timing variation and evades both threshold alarms and sliding-window anomaly detection.

the paper saw this coming. it says GPS "will suffer from spoofing vulnerabilities" and asks what asymmetric path attacks mean for White Rabbit. that question is now answered and the answer is unwelcome. **swapping the clock fixed the jitter and did not fix the trust.** if a coincidence window is a security boundary, and it is, then somebody who can move your window can move your noise floor, and there is no protocol on any of these networks that would notice. a [critical assessment of quantum timing protocols](https://arxiv.org/abs/2604.10243) is the current survey and its conclusion is that quantum time transfer is not replacing classical methods soon, which leaves the attack surface where it is.

**2. polarization went from a footnote to the binding constraint.** the moment jitter drops to picoseconds, the thing that limits your fidelity is a fiber that reconfigures itself. Qunnect's [automated compensation over 34 km of deployed New York fiber](https://doi.org/10.1103/PRXQuantum.5.030330) ran 15 days at 99.84% uptime, with a correction fired roughly every 20 seconds. ORNL's own [dim-reference heterodyne stabilization on the EPB Chattanooga network](https://arxiv.org/abs/2411.15135) held 0.94 average fidelity for 30 hours at 100% uptime, using references at −50 dBm that add no counts above dark noise.

and the timescale depends on where the glass is. on a [62 km partially aerial link](https://arxiv.org/abs/2601.11753), polarization is stable for tens of minutes at night and drifts by tens of degrees in seconds during the day, with fidelity falling below 95% in under 20 seconds. [eleven months of drift statistics on a hybrid aerial-inground link](https://arxiv.org/abs/2607.07629) show a clean diurnal cycle and a seasonal one on top of it, with wind speed dominating in winter.

so "typically stable for hours at a time" was a statement about underground fiber on a campus in a mild month. the thing i would want next is the measurement nobody has published cleanly: **an aerial and a buried fiber, same route, same instrument, same year.** the aerial-inground paper deliberately treats its link as one system. until someone separates them, the drift penalty for hanging fiber on poles is folklore with good supporting evidence rather than a number you can put in a link budget.

**3. close the loop, and put pump power in it.** allocation in this paper is open-loop: pick channels, align by hand, measure, publish. the pieces for a controller now exist. [Rate-Fidelity Control for Wide-Area Quantum Links](https://arxiv.org/abs/2608.07163) is the first work i have seen that treats **SPDC pump power as a runtime actuator** rather than a design-time choice, jointly with polarization compensation scheduling, maximizing entanglement rate under an application-specified minimum fidelity. trace-driven on 24 hours from a 64 km deployed fiber, it reports about 14% more mean entanglement rate than optimized static policies, without offline tuning.

that is the right shape. and it is exactly the knob this paper needed: the B–C link in allocation 1 is a link with too much flux for its fidelity target, and no amount of spectrum reshuffling fixes that. turning the pump down would have.

**4. give the allocator the accidentals term.** every routing and spectrum allocation solver in this line, from the [2022 genetic algorithm](https://arxiv.org/abs/2204.06642) through the [NP-hardness result](https://arxiv.org/abs/2404.08744) to [the 2026 Yen plus APOPT plus CP-SAT pipeline](https://arxiv.org/abs/2607.15465), optimizes rate under a fidelity constraint where fidelity is a per-channel property. this paper shows it is not. fidelity is a function of how much bandwidth you gave the link, because accidentals grow quadratically with flux while coincidences grow linearly, and it is a function of your coincidence window, which is a function of your clock. that is one analytic term, and it is the term that separates "the solver says this allocation is fine" from "this allocation delivers a stated service class."

**5. publish the reconfiguration time.** i said this in the flex grid post and it is still true, and this paper makes it worse: here, changing the allocation also means re-running the polarization alignment at every affected node. the switch moves in milliseconds. the compensation loop settles in [30 to 1000 ms per cycle](https://doi.org/10.1103/PRXQuantum.5.030330) when it is automated, and in an afternoon when it is a person with a fiber paddle. the sum is the service-change latency of a flex-grid quantum network. it is an afternoon's measurement and nobody has done it.

**6. work out what a network does about Bob.** the detector heterogeneity observation in this paper is sharp and then dropped. every deployed network since has the same shape: the [Qunnect and Cisco NYC network](https://arxiv.org/abs/2602.15653) runs a 65% SPAD alongside SNSPDs at 80% and 93% in the same topology. nobody has written down the allocation problem where node capability is a first-class input, and the interesting version is not "give the weak node more spectrum," because this paper shows that makes it worse. it is admission control: tell Bob at request time which service classes his hardware can actually hold, and let him buy a longer integration time instead of a worse link.

**7. give ebits/s a reference procedure, or stop using it.** the metric deserves to win and it is losing. what is missing is boring and fixable: a stated integration time, a stated basis set, a stated position on accidental subtraction (which, per the identity above, barely matters, and saying so in a spec would settle arguments), and a companion number for reliability. ORNL is circling this with [work on standardized quantum network metrics](https://arxiv.org/abs/2607.05642) that puts temperature, humidity and vibration alongside fidelity and rate as first-class quantities, which is the right instinct given where the polarization results landed. there is still no accepted network-level analogue of quantum volume. the closest thing, [a randomized-benchmarking procedure for network links](https://www.nature.com/articles/s41534-022-00628-x), measures link transmission quality rather than what a network can compose.

**8. do something with RSP.** this remains, as far as i can find, the first and effectively the last remote state preparation over deployed fiber. the field moved to teleportation over urban fiber, [14.4 km in Saarland at 80% process fidelity with a trapped ion](https://www.nature.com/articles/s41534-024-00886-x), and to entanglement swapping, [17.6 km between Brooklyn and Manhattan](https://arxiv.org/abs/2602.15653). blind quantum computing, which is the application RSP was pitched for, has been [demonstrated on a two-node solid-state network](https://www.science.org/doi/10.1126/science.adu6894) and on [a photonic Qline](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.134.200603), both in the lab. **nobody has run blind quantum computing over deployed fiber.** this paper built the primitive and put it outside five years ago. that gap is the obvious thing to close, and the network that closed it first would have the most convincing demonstration in the field.

## the short version

- the flex grid architecture works outside. three buildings, deployed campus fiber, eight reconfigurable 25 GHz channel pairs, entanglement on all three links.
- a tabletop gives you a shared clock, a fiber you chose, matched detectors and a metric that does not have to travel. deployment takes all four away.
- **the clock cost the most.** GPS wander of 12 to 15 ns forced a 10 ns coincidence window, and accidentals scale with the window.
- the paper reports every result twice, raw and accidentals-subtracted. the gap is up to 37 points of fidelity, and on the worst link there is more noise than signal, 1.25 accidentals per real pair.
- take those two tables, scale the accidentals down 10x, and you predict the follow-up paper's White Rabbit fidelities to within a couple of points on every link. the 2021 network was limited by GPS, not by physics.
- more spectrum does not mean a better link. B–C got three times the bandwidth and gained 15% in rate while losing 14 points of fidelity, because coincidences grow linearly with flux and accidentals grow quadratically.
- the two allocations do not order. C–A trades 6 points of fidelity for 55% more ebits/s going the other way. "the optimal allocation may depend on a given objective" is the real result.
- **log-negativity times coincidence rate, in ebits/s, is the paper's best idea.** it is insensitive to how you account for accidentals, because log-negativity falls off as `1/(1+a)` and the coincidence rate rises by the same factor.
- remote state preparation ran on all three links at 0.80 to 0.94, agreeing with the measured density matrices to 0.99. the protocol was not the limitation.
- since 2021: White Rabbit closed the window to 1 ns, polarization replaced timing as the hard problem, allocation became a solved-ish RSA problem, and timing turned into an attack surface with demonstrated sub-nanosecond delay attacks.
- what is missing: a clock you can trust rather than just a precise one, an aerial-versus-buried drift number, pump power in the control loop, the accidentals term in the allocator, a published reconfiguration time, admission control for weak nodes, a reference procedure for ebits/s, and blind quantum computing on a real fiber.

the thing i keep coming back to is that the biggest single number in this paper is not about entanglement at all. it is 12.1 nanoseconds, the disagreement between two GPS receivers in two buildings, and it set the noise floor of every link in the network. the quantum part was fine. the network was waiting on a clock.

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
  function erf(x){
    var s=x<0?-1:1; x=Math.abs(x);
    var t=1/(1+0.3275911*x);
    var y=1-(((((1.061405429*t-1.453152027)*t)+1.421413741)*t-0.284496736)*t+0.254829592)*t*Math.exp(-x*x);
    return s*y;
  }
  function log2(x){ return Math.log(x)/Math.LN2; }
  function fmt(x,d){ return x.toFixed(d===undefined?2:d); }

  var GREEN='#32c29e', AMBER='#f2a03d', BLUE='#5b8def', GREY='#4a4a4a',
      AXIS='#3a3a3a', TICK='#7d7d7d', INK='#e6e6e6';

  /* =======================================================================
     WIDGET 1 - the coincidence window as a noise budget

     Accidental coincidences are uncorrelated, so their count grows linearly
     with the window width, while true pairs sit in a sub-nanosecond peak and
     do not.  Accidentals are unpolarised, so tomography sees them as a
     maximally mixed background contributing 0.25 to the fidelity:

         F_measured = ( F_true + 0.25 a ) / ( 1 + a )

     with a = accidentals per true pair.  Inverting that on the paper's own
     Table II (raw) and Table III (accidentals subtracted) gives a at the
     10 ns window used throughout, which is the only calibration here.
     ==================================================================== */
  (function(){
    var root=document.getElementById('qw-win'); if(!root) return;
    var LINKS=[
      {n:'A–B, allocation 2, ch 3',      Fs:0.957, a10:0.414, wr:0.938},
      {n:'B–C, allocation 2, ch 1–2', Fs:0.959, a10:0.611, wr:0.910},
      {n:'C–A, allocation 2, ch 4',      Fs:0.968, a10:0.217, wr:0.971},
      {n:'B–C, allocation 1, ch 2–7', Fs:0.925, a10:1.250, wr:null}
    ];
    var CLK={'12.1':'GPS between buildings','1.21':'GPS on one bench','0.0129':'White Rabbit','0.003':'mode-locked laser sync'};
    var li=3, sig=12.1;
    var sl=document.getElementById('qww-t');
    var LO=-2, HI=1.6;                                   /* 0.01 ns .. 39.8 ns, log; v=75 is exactly 10 ns */
    function tau(){ return Math.pow(10, LO+(HI-LO)*(+sl.value)/90); }

    function acc(t,L){ return L.a10*t/10; }
    function fid(t,L){ var a=acc(t,L); return (L.Fs+0.25*a)/(1+a); }

    var hist=document.getElementById('qww-hist'), curve=document.getElementById('qww-curve');

    function drawHist(t,L){
      clear(hist);
      var W=320,H=170,ml=6,mr=6,mb=22,mt=26, span=40;   /* +/- 40 ns */
      var x=function(v){ return ml+(v+span)/(2*span)*(W-ml-mr); };
      /* bars, 1 ns bins */
      var sp=0.35, bins=[], max=0;
      for(var i=-span;i<span;i++){
        var c=i+0.5;
        var pk=0.5*(erf((c+0.5)/(sp*Math.SQRT2))-erf((c-0.5)/(sp*Math.SQRT2)));
        var fl=L.a10/10;
        bins.push({c:c,pk:pk,fl:fl});
        if(pk+fl>max) max=pk+fl;
      }
      var y0=H-mb, hMax=y0-mt;
      var hOf=function(v){ return hMax*Math.pow(v/max,0.42); };
      /* the accepted window */
      var wx0=x(-t/2), wx1=x(t/2);
      hist.appendChild(el('rect',{x:Math.min(wx0,wx1)-(wx1-wx0<3?1.5:0),y:mt-4,width:Math.max(3,wx1-wx0),height:hMax+4,
        fill:BLUE,'fill-opacity':'0.13',stroke:BLUE,'stroke-width':'1.2','stroke-opacity':'0.75'}));
      bins.forEach(function(b){
        var bw=(W-ml-mr)/(2*span), inw=Math.abs(b.c)<=t/2;
        var hf=hOf(b.fl), hp=hOf(b.pk+b.fl)-hf;
        hist.appendChild(el('rect',{x:x(b.c)-bw/2,y:y0-hf,width:Math.max(0.8,bw-0.4),height:hf,fill:AMBER,'fill-opacity':inw?'0.9':'0.2'}));
        if(hp>0.4) hist.appendChild(el('rect',{x:x(b.c)-bw/2,y:y0-hf-hp,width:Math.max(0.8,bw-0.4),height:hp,fill:GREEN,'fill-opacity':inw?'1':'0.25'}));
      });
      hist.appendChild(el('line',{x1:ml,y1:y0,x2:W-mr,y2:y0,stroke:AXIS,'stroke-width':'1'}));
      /* where the peak may sit, given the clock */
      var s=Math.min(sig,span*0.95);
      hist.appendChild(el('line',{x1:x(-s),y1:mt-13,x2:x(s),y2:mt-13,stroke:TICK,'stroke-width':'1'}));
      [-s,s].forEach(function(v){ hist.appendChild(el('line',{x1:x(v),y1:mt-17,x2:x(v),y2:mt-9,stroke:TICK,'stroke-width':'1'})); });
      hist.appendChild(el('text',{x:W/2,y:mt-19,fill:TICK,'font-size':'8.5','text-anchor':'middle'},
        sig>=0.05?('the peak wanders ±'+fmt(sig,1)+' ns'):'clock wander, too small to see'));
      [-40,-20,0,20,40].forEach(function(v){
        hist.appendChild(el('text',{x:x(v),y:y0+13,fill:TICK,'font-size':'9','text-anchor':'middle'},v+''));
      });
      hist.appendChild(el('text',{x:W/2,y:H-2,fill:TICK,'font-size':'9','text-anchor':'middle'},'delay between the two detections, ns'));
      hist.appendChild(el('text',{x:ml+1,y:mt+6,fill:TICK,'font-size':'8.5'},'counts'));
    }

    function drawCurve(t,L){
      clear(curve);
      var W=320,H=170,ml=30,mr=8,mb=32,mt=12;
      var x=function(v){ return ml+(Math.log(v)/Math.LN10-LO)/(HI-LO)*(W-ml-mr); };
      var y=function(v){ return H-mb-(v-0.25)/0.78*(H-mb-mt); };
      [0.25,0.4,0.6,0.8,1.0].forEach(function(v){
        curve.appendChild(el('line',{x1:ml,y1:y(v),x2:W-mr,y2:y(v),stroke:AXIS,'stroke-width':'0.5'}));
        curve.appendChild(el('text',{x:ml-4,y:y(v)+3,fill:TICK,'font-size':'9','text-anchor':'end'},v.toFixed(2)));
      });
      [0.01,0.1,1,10].forEach(function(v){
        curve.appendChild(el('text',{x:x(v),y:H-mb+11,fill:TICK,'font-size':'9','text-anchor':'middle'},v<1?String(v):v+' ns'));
      });
      /* a window narrower than the clock's own wander cannot reliably hold the peak */
      var lim=Math.min(Math.max(sig,Math.pow(10,LO)),39.8);
      curve.appendChild(el('rect',{x:ml,y:mt,width:Math.max(0,x(lim)-ml),height:H-mb-mt,fill:'#c98a3a','fill-opacity':'0.06'}));
      curve.appendChild(el('line',{x1:x(lim),y1:mt,x2:x(lim),y2:H-mb,stroke:'#c9a178','stroke-width':'1','stroke-dasharray':'3 2'}));
      if(x(lim)-ml>58) curve.appendChild(el('text',{x:x(lim)-4,y:mt+9,fill:'#c9a178','font-size':'8','text-anchor':'end'},'clock wander'));
      curve.appendChild(el('line',{x1:ml,y1:y(L.Fs),x2:W-mr,y2:y(L.Fs),stroke:GREEN,'stroke-width':'1','stroke-dasharray':'3 3','stroke-opacity':'0.6'}));
      curve.appendChild(el('text',{x:ml+3,y:y(L.Fs)-4,fill:GREEN,'font-size':'8','fill-opacity':'0.8'},'the state the fiber delivered'));
      var d='';
      for(var i=0;i<=160;i++){
        var tv=Math.pow(10,LO+(HI-LO)*i/160);
        d+=(i?'L':'M')+x(tv).toFixed(1)+' '+y(fid(tv,L)).toFixed(1);
      }
      curve.appendChild(el('path',{d:d,fill:'none',stroke:BLUE,'stroke-width':'2'}));
      [{v:10,l:'this paper',dy:-14},{v:1,l:'the 2022 upgrade',dy:-4}].forEach(function(m){
        curve.appendChild(el('line',{x1:x(m.v),y1:mt+2,x2:x(m.v),y2:H-mb,stroke:TICK,'stroke-width':'0.7','stroke-dasharray':'2 3'}));
        curve.appendChild(el('text',{x:x(m.v)-3,y:H-mb+m.dy,fill:TICK,'font-size':'8','text-anchor':'end'},m.l));
      });
      curve.appendChild(el('circle',{cx:x(t),cy:y(fid(t,L)),r:4,fill:BLUE,stroke:'#191919','stroke-width':'1.5'}));
      if(L.wr) curve.appendChild(el('circle',{cx:x(1),cy:y(L.wr),r:3.2,fill:'none',stroke:GREEN,'stroke-width':'1.6'}));
      curve.appendChild(el('text',{x:W/2,y:H-2,fill:TICK,'font-size':'8.5','text-anchor':'middle'},'coincidence window'));
    }

    function render(){
      var t=tau(), L=LINKS[li], a=acc(t,L), F=fid(t,L), p=1/(1+a);
      document.getElementById('qww-tv').textContent = t<0.1?(t*1000).toFixed(0)+' ps':fmt(t,t<1?2:1)+' ns';
      document.getElementById('qww-a').textContent  = a<0.01?a.toExponential(1):fmt(a,2);
      document.getElementById('qww-f').textContent  = fmt(F,3);
      document.getElementById('qww-p').textContent  = (100*p).toFixed(1)+'%';
      var v,note;
      if(a>1){
        v='more noise than signal';
        note='At '+fmt(t,1)+' ns this link carries <b>'+fmt(a,2)+' accidentals for every real pair</b>. Only '+(100*p).toFixed(0)+
             '% of what the two nodes call a coincidence is one. Tomography reads the rest as a maximally mixed background, which is why the measured fidelity is '+
             fmt(F,2)+' against an underlying '+fmt(L.Fs,3)+'. This is the worst link in the paper: six channels of flux into a 20% detector.';
      } else if(t<sig){
        v='narrower than the clock wanders';
        note='A window of '+(t<0.1?(t*1000).toFixed(0)+' ps':fmt(t,2)+' ns')+' is narrower than the wander of '+CLK[String(sig)]+
             ' (σ = '+fmt(sig,sig<0.1?4:2)+' ns), so the coincidence peak can drift out of it between alignments and you start losing real pairs rather than only noise. '+
             'The paper reasons from the same numbers that <b>windows up to about 30 ns would be reasonable</b> and picks 10 ns anyway, giving up some pairs to avoid three times the accidentals. That is the trade, stated once and never revisited.';
      } else if(a>0.1){
        v='paying for the clock';
        note='Accidentals are '+fmt(a,2)+' per real pair, costing '+fmt(L.Fs-F,3)+' of fidelity. Every nanosecond of window is a proportional dose of noise, so this is a clock bill, not a source or a fiber bill.';
      } else {
        v='clean';
        note='Accidentals are down to '+(a<0.01?a.toExponential(1):fmt(a,3))+' per real pair and the measured fidelity is within '+fmt(L.Fs-F,3)+
             ' of the underlying state. '+(L.wr?'The green ring marks what this link actually measured a year later with White Rabbit and a 1 ns window: <b>'+fmt(L.wr,3)+'</b>.':'');
      }
      document.getElementById('qww-v').textContent=v;
      document.getElementById('qww-note').innerHTML=note;
      drawHist(t,L); drawCurve(t,L);
    }
    btnGroup(root,'clk',function(v){ sig=+v; render(); });
    btnGroup(root,'lnk',function(v){ li=+v; render(); });
    sl.addEventListener('input',render);
    sl.value=75; render();
  })();

  /* =======================================================================
     WIDGET 2 - the two allocations

     Channel n sits at (n - 1/2) x 25 GHz either side of degeneracy, and the
     pair spectrum is a sinc-squared with a 310 GHz FWHM, which puts the 50%
     point inside channel 7.  All fidelity, log-negativity and ebit-rate
     values are the paper's, Table II raw and Table III with accidentals
     subtracted.  The curve on the right is the log-negativity a Werner state
     of a given fidelity would have, log2(2F), which is a floor: every
     measured point sits above it.
     ==================================================================== */
  (function(){
    var root=document.getElementById('qw-alloc'); if(!root) return;
    var C={AB:GREEN,BC:AMBER,CA:BLUE,'':GREY};
    var A={
      '1':{own:['AB','BC','BC','BC','BC','BC','BC','CA'],
           links:{AB:{ch:'1',    raw:[0.75,0.70,56],  sub:[0.960,0.96, 52.4]},
                  BC:{ch:'2–7',  raw:[0.55,0.40,30],  sub:[0.925,0.95, 33.0]},
                  CA:{ch:'8',    raw:[0.90,0.89,206], sub:[0.985,0.987,207]}}},
      '2':{own:['BC','BC','AB','CA','','','',''],
           links:{AB:{ch:'3',    raw:[0.75,0.70,57],  sub:[0.957,0.97, 53.3]},
                  BC:{ch:'1–2',  raw:[0.69,0.60,26],  sub:[0.959,0.98, 24.8]},
                  CA:{ch:'4',    raw:[0.84,0.82,320], sub:[0.968,0.98, 320]}}}
    };
    var al='1', sub=0;
    var spec=document.getElementById('qwa-spec'), plane=document.getElementById('qwa-plane');
    function flux(n){ var f=(n-0.5)*25, x=1.39156*f/155; return Math.pow(Math.sin(x)/x,2); }

    function drawSpec(){
      clear(spec);
      var W=320,H=175,ml=8,mr=8,mb=30,mt=16, span=205, own=A[al].own;
      var x=function(v){ return ml+(v+span)/(2*span)*(W-ml-mr); };
      var y0=H-mb, hMax=y0-mt;
      var d='';
      for(var i=0;i<=200;i++){
        var f=-span+2*span*i/200, u=1.39156*Math.abs(f)/155;
        var v=u<1e-6?1:Math.pow(Math.sin(u)/u,2);
        d+=(i?'L':'M')+x(f).toFixed(1)+' '+(y0-v*hMax).toFixed(1);
      }
      spec.appendChild(el('path',{d:d,fill:'none',stroke:'#555','stroke-width':'1','stroke-dasharray':'3 3'}));
      spec.appendChild(el('line',{x1:x(0),y1:mt-6,x2:x(0),y2:y0,stroke:'#666','stroke-width':'0.8'}));
      spec.appendChild(el('text',{x:x(0),y:mt-9,fill:TICK,'font-size':'8.5','text-anchor':'middle'},'degeneracy'));
      var bw=(W-ml-mr)/(2*span)*23;
      for(var n=1;n<=8;n++){
        var h=flux(n)*hMax, c=C[own[n-1]], f=(n-0.5)*25;
        [f,-f].forEach(function(pos){
          spec.appendChild(el('rect',{x:x(pos)-bw/2,y:y0-h,width:bw,height:h,fill:c,
            'fill-opacity':own[n-1]?'0.9':'0.5',rx:'1'}));
        });
        spec.appendChild(el('text',{x:x(f),y:y0+11,fill:TICK,'font-size':'8','text-anchor':'middle'},n+''));
        spec.appendChild(el('text',{x:x(-f),y:y0+11,fill:TICK,'font-size':'8','text-anchor':'middle'},n+''));
      }
      spec.appendChild(el('line',{x1:ml,y1:y0,x2:W-mr,y2:y0,stroke:AXIS,'stroke-width':'1'}));
      spec.appendChild(el('text',{x:ml+1,y:mt+2,fill:TICK,'font-size':'8.5'},'pair flux'));
      spec.appendChild(el('text',{x:W/2,y:H-4,fill:TICK,'font-size':'8.5','text-anchor':'middle'},
        'idler side  ·  channel number  ·  signal side'));
    }

    function drawPlane(){
      clear(plane);
      var W=320,H=175,ml=32,mr=12,mb=26,mt=14;
      var V=sub?{x0:0.90,x1:1.0,y0:0.86,y1:1.02,xt:[0.90,0.94,0.98],yt:[0.90,0.95,1.00],lab:0.955}
               :{x0:0.45,x1:1.02,y0:0,   y1:1.05,xt:[0.5,0.6,0.7,0.8,0.9,1.0],yt:[0,0.25,0.5,0.75,1],lab:0.62};
      var x=function(v){ return ml+(v-V.x0)/(V.x1-V.x0)*(W-ml-mr); };
      var y=function(v){ return H-mb-(v-V.y0)/(V.y1-V.y0)*(H-mb-mt); };
      V.yt.forEach(function(v){
        plane.appendChild(el('line',{x1:ml,y1:y(v),x2:W-mr,y2:y(v),stroke:AXIS,'stroke-width':'0.5'}));
        plane.appendChild(el('text',{x:ml-4,y:y(v)+3,fill:TICK,'font-size':'9','text-anchor':'end'},v.toFixed(2)));
      });
      V.xt.forEach(function(v){
        plane.appendChild(el('text',{x:x(v),y:H-mb+12,fill:TICK,'font-size':'9','text-anchor':'middle'},v.toFixed(2)));
      });
      var d='';
      for(var i=0;i<=100;i++){
        var F=Math.max(0.5,V.x0)+(V.x1-Math.max(0.5,V.x0))*i/100;
        d+=(i?'L':'M')+x(F).toFixed(1)+' '+y(log2(2*F)).toFixed(1);
      }
      plane.appendChild(el('path',{d:d,fill:'none',stroke:'#666','stroke-width':'1','stroke-dasharray':'4 3'}));
      plane.appendChild(el('text',{x:x(V.lab),y:y(log2(2*V.lab))+16,fill:'#8a8a8a','font-size':'8.5','text-anchor':'middle'},'Werner floor'));
      ['1','2'].forEach(function(k){
        ['AB','BC','CA'].forEach(function(lk){
          var m=A[k].links[lk][sub?'sub':'raw'], cur=(k===al);
          plane.appendChild(el('circle',{cx:x(m[0]),cy:y(m[1]),r:cur?5:3,fill:cur?C[lk]:'none',
            stroke:C[lk],'stroke-width':'1.4','fill-opacity':cur?'0.95':'0','stroke-opacity':cur?'1':'0.4'}));
          if(cur){
            var rt=x(m[0])>W-mr-34, dy=({AB:11,BC:-7,CA:3.5})[lk];
            plane.appendChild(el('text',{x:x(m[0])+(rt?-7:7),y:y(m[1])+dy,fill:C[lk],'font-size':'9','text-anchor':rt?'end':'start'},lk[0]+'–'+lk[1]));
          }
        });
      });
      plane.appendChild(el('text',{x:W/2,y:H-3,fill:TICK,'font-size':'8.5','text-anchor':'middle'},'fidelity to |Ψ⁺⟩'));
      plane.appendChild(el('text',{x:3,y:mt-2,fill:TICK,'font-size':'8.5'},'ebits'));
    }

    function render(){
      ['AB','BC','CA'].forEach(function(lk){
        var L=A[al].links[lk], m=L[sub?'sub':'raw'];
        document.getElementById('qwa-'+lk.toLowerCase()).textContent =
          'ch '+L.ch+'  ·  F '+m[0].toFixed(m[0]>=0.9?3:2)+'  ·  '+m[1].toFixed(2)+' ebits  ·  '+m[2]+' ebits/s';
      });
      var n;
      if(al==='1'){
        n='Allocation 1 spends every channel and tries to <b>even out the rates</b>: the dimmest slice, channel 8, goes to the pair with the best detectors and the most fiber, and the six middle channels go to B–C to compensate for Bob&rsquo;s APD. It does not work. Six channels is 150 GHz of flux into a 20% detector, and the extra multipair emission drags B–C down to a measured fidelity of 0.55 while buying only 30 ebits/s.';
      } else {
        n='Allocation 2 <b>rescues the worst link by giving it less</b>: B–C drops from six channels to two and its fidelity climbs 14 points, at a cost of 4 ebits/s. Four channels are left unassigned, held for nodes that do not exist yet. C–A moves inward from channel 8 to channel 4, trading 6 points of fidelity for 55% more entanglement per second, which is the opposite trade in the same reconfiguration.';
      }
      n+=sub?' Numbers shown are with accidentals subtracted before tomography, which is the state the fiber and the source actually delivered. Note the ebit rates barely move.'
             :' Numbers shown are as measured through a 10 ns coincidence window.';
      document.getElementById('qwa-note').innerHTML=n;
      drawSpec(); drawPlane();
    }
    btnGroup(root,'al',function(v){ al=v; render(); });
    btnGroup(root,'sb',function(v){ sub=+v; render(); });
    render();
  })();
})();
</script>{% endraw %}