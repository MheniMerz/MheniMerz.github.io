---
layout: post
section-type: post
title: "Multihop entanglement : what a quantum network does when a fiber breaks"
category: 'networking'
tags: [ 'networking', 'fiber', 'quantum' ]
---

every transport network has an answer to the backhoe. sonet had automatic protection switching and a 50 ms budget for it. sdh had the same. optical mesh networks have pre-computed restoration paths, shared spare capacity and a whole literature on how to place it. the assumption underneath all of it is that fiber gets cut, and the network is supposed to notice and keep going.

quantum networks have had no answer at all. every entanglement distribution experiment before this one ran on exactly one lightpath between the source and each user. cut it and the service stops until somebody walks to the patch panel.

this paper is the first one to fix that. [Alshowkan, Lukens, Lu and Peters, "Resilient Entanglement Distribution in a Multihop Quantum Network," *Journal of Lightwave Technology* 43(19), 9016 (2025)](https://doi.org/10.1109/JLT.2025.3581220) ([arXiv](https://arxiv.org/abs/2407.20443)). six users, three buildings on the Oak Ridge campus, one entangled photon source, and a controller that watches for a dead link and reroutes the spectrum through a different building.

it works. it also takes five seconds, costs about 10 dB per hop, and in one of the two recovery scenarios the restored links come back at a fidelity of 0.76. that last number is the one i want to spend time on, because the paper reports it accurately and then moves on, and i think it is the most important thing in the paper.

if you've read [the flex grid post]({% post_url 2026-04-04-Flex-grid-for-entanglement %}), this is the same group's network four years later, with a control plane bolted on. if you haven't, the short version is below.

## what came before

one box makes entangled photon pairs. every user has a fiber to that box. the pairs come out over a wide band of wavelengths at once, and because the two photons of a pair always add up to the pump energy, wavelength is the address:

```
    ω_signal  +  ω_idler  =  ω_pump
```

hand two users energy-matched slices of that spectrum and they share entanglement. hand them mismatched slices and they share nothing. so provisioning who-talks-to-whom is a spectrum allocation problem, and a wavelength selective switch, the same liquid-crystal-on-silicon part that lives in every ROADM, is the thing that does the allocating.

that was the 2021 result. this paper takes that architecture and asks what happens when the fiber underneath it breaks.

## the network

three buildings on the ORNL campus, one subnetwork each.

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Building</th><th>Users</th><th>Detectors</th><th>Fiber to A</th></tr>
</thead>
<tbody>
<tr><td>A</td><td>Alice (A1), Avery (A2)</td><td>SNSPD, &gt;81%</td><td>the source is here</td></tr>
<tr><td>B</td><td>Bob (B1), Bailey (B2)</td><td>free-running APD, 20%</td><td>250 m</td></tr>
<tr><td>C</td><td>Charlie (C1), Casey (C2)</td><td>SNSPD, &gt;81%</td><td>1.2 km</td></tr>
</tbody>
</table>
</div>

plus a 930 m fiber between B and C. three buildings, three fibers, a full mesh, which at N=3 is also the minimum for any protection at all.

the source in building A is a continuous-wave laser at 779.4 nm putting about 25 mW into a type-II PPLN ridge waveguide, giving polarization-entangled pairs in the state `|Ψ+⟩ ∝ |HV⟩ + |VH⟩` across roughly 310 GHz. WSS1 carves that into eight pairs of frequency-correlated 25 GHz channels centered on 192.3125 THz, which puts them on the standard ITU grid, channels 21.25 to 25.00.

two of WSS1's outputs go to the local users. two more go to the patch panel and out to B and C, where WSS2 and WSS3 act as local hubs and split the arriving spectrum among their own users.

three things make it a network rather than an experiment:

**a clock.** a rubidium frequency standard disciplines a White Rabbit switch, which feeds a White Rabbit node in each building over 1310/1490 nm SFPs. those nodes hand out 10 MHz (doubled to 20) and a pulse per second to the time-to-digital converters that timestamp every detection. the coincidence window is about 1 ns. if you want the argument for why the clock is the hard part, it's in [the coexistence post]({% post_url 2026-03-23-coexistence-running-a-quantum-channel-and-its-clock-down-the-same-fiber %}).

**a control plane.** an SDN controller programs the WSSs and a set of MEMS switches. the quantum data plane is all-optical: no optical-to-electrical conversion anywhere, so nothing in the path measures or copies the photons.

**redundant fiber and MEMS switches.** building B has a 2×1 MEMS switch and a circulator; building C has a 2×2. these are what let the controller pick which physical fiber feeds the local WSS, and they are the entire protection mechanism.

one thing worth flagging early: the classical and quantum signals run on **separate fibers**. that sidesteps Raman scattering and filter crosstalk entirely, and the paper says so. it is also a luxury that a real deployment on leased fiber will not have.

## what a hop costs

in normal operation, the total loss from the input of WSS1 to a user's detector is 7.25 dB (A1) and 8.51 dB (A2) for the local users, and 16.6 to 17.6 dB for everyone in B and C.

when a link fails and traffic is rerouted through the other building, C1 goes from 17.2 dB to 27.5 dB. B2 goes from 16.6 dB to 28.0 dB. the paper's summary: about 10 dB per extra hop.

almost none of that is fiber. 250 m of SMF-28 is 0.05 dB. the 10 dB is the insertion loss of a second WSS, plus the MEMS switch, plus connectors. on a campus this size the network is entirely component-limited, and the fiber is free.

that matters because of how a coincidence rate is built. a link needs *both* photons to arrive. so the rate scales as the product of the two arms' transmission, and whether one photon or two take the detour changes the answer by an order of magnitude.

<div class="qw" id="qw-hop" data-qw><div class="qw-hd"><span class="qw-t">what an extra hop costs a link</span><span class="qw-s">a coincidence needs both photons, so the rate scales as the product of the two arms. drag the hops and watch which arm is carrying them.</span><span class="qw-lg"><i style="background:#32c29e"></i>direct path, both arms<i style="background:#f2a03d"></i>after rerouting</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">the paper&rsquo;s measured cases</span><span class="qw-b" data-grp="ps"><button type="button" data-v="direct">direct, no detour</button><button type="button" data-v="one" class="on" aria-pressed="true">A1&ndash;C1 rerouted via B</button><button type="button" data-v="both">C1&ndash;C2 rerouted via B</button></span></div><div class="qw-g"><span class="qw-l">who takes the detour</span><span class="qw-b" data-grp="ar"><button type="button" data-v="1" class="on" aria-pressed="true">one photon</button><button type="button" data-v="2">both photons</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">extra hops <b id="qwp-hv">1</b></span><input type="range" id="qwp-h" min="0" max="4" step="1" value="1"></label><label class="qw-sr"><span class="qw-l">insertion loss per hop <b id="qwp-lv">10.0 dB</b></span><input type="range" id="qwp-l" min="1" max="20" step="0.5" value="10"></label><label class="qw-sr"><span class="qw-l">fiber between nodes <b id="qwp-dv">1.2 km</b></span><input type="range" id="qwp-d" min="0" max="80" step="0.5" value="1.2"></label></div><div class="qw-pane"><span class="qw-pl">the two arms of one link, and where the loss lands</span><svg class="qw-svg" viewBox="0 0 640 210" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Loss budget of the two arms of an entangled link"><g id="qwp-diag"></g></svg></div><div class="qw-out"><div class="qw-oi"><span class="qw-ok">added loss, arm 1</span><span class="qw-ov" id="qwp-a1">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">added loss, arm 2</span><span class="qw-ov" id="qwp-a2">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">coincidence rate</span><span class="qw-ov" id="qwp-r">&mdash;</span></div><div class="qw-oi qw-grow"><span class="qw-ok">one full tomography, 36 settings</span><span class="qw-ov qw-txt" id="qwp-t">&mdash;</span></div></div><p class="qw-n" id="qwp-note"></p><noscript><p class="qw-n">Rerouting adds about 10 dB per hop, almost all of it component insertion loss rather than fiber. When only one photon of a pair takes the detour the coincidence rate falls by roughly 10x; when both photons take it the rate falls by roughly 100x. The paper measured 48.1 to 0.33 coincidences per second on the C1&ndash;C2 link, a factor of 146.</p></noscript></div>

the C1&ndash;C2 link is the one to look at. both Charlie and Casey sit in building C, so when C's feed is rerouted through B, **both** photons of every pair take the extra hop. their singles rates drop by 10.9x and 11.2x, and the coincidence rate goes from 48.1 to 0.33 per second, a factor of 146 where the product of the two singles reductions predicts about 123.

for comparison, A1&ndash;C1 has only one photon on the detour and loses a factor of 10.

that asymmetry is a routing fact, not a physics curiosity. **an intra-building link pays every hop twice.** a controller choosing a restoration path should know that the users whose service degrades worst are the ones on the far side, talking to each other.

## the baseline

before breaking anything, they characterize the star topology: WSS1 in the middle, direct fibers to B and C. six links, tomography on all of them at once, 36 polarization projections each, Bayesian state estimation.

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Link</th><th>Type</th><th>Fidelity to |&Psi;<sup>+</sup>&rang;</th><th>Coincidences/s</th></tr>
</thead>
<tbody>
<tr><td>A1&ndash;A2</td><td>intra, building A</td><td>0.9504(8)</td><td>866.5(7)</td></tr>
<tr><td>B1&ndash;B2</td><td>intra, building B</td><td>0.887(8)</td><td>9.16(7)</td></tr>
<tr><td>C1&ndash;C2</td><td>intra, building C</td><td>0.932(3)</td><td>48.1(1)</td></tr>
<tr><td>A1&ndash;B1</td><td>inter</td><td>0.965(8)</td><td>43.4(4)</td></tr>
<tr><td>A2&ndash;C1</td><td>inter</td><td>0.978(2)</td><td>246.6(1)</td></tr>
<tr><td>B1&ndash;C1</td><td>inter</td><td>0.884(2)</td><td>9.8(2)</td></tr>
</tbody>
</table>
</div>

nothing surprising. the links with an APD user in building B are the slow ones, which is a detector problem, not a network problem. the good links are up around 0.97.

note the rates, though. the fastest link on this network moves 866 pairs per second. the slowest moves 9. an ethernet frame is 12,000 bits. these are not the same units as anything a network engineer is used to holding in their head, and it is worth recalibrating before reading the recovery numbers.

## breaking it

the failure model is a single building-to-building fiber going away. with three buildings in a triangle, any one of the three can be lost and the other two still reach each other the long way round.

detection is a threshold. the SDN application layer sets a minimum singles count for each detector, chosen from the dark count and background level. when a detector's singles fall below it, the controller assumes the upstream fiber is gone and starts the reroute: WSS1 is told to send building C's slots to building B instead, WSS2 is told to pass the extra spectrum through to its output fiber, and the MEMS switches flip to put the right fiber on the right WSS input.

<div class="qw" id="qw-net" data-qw><div class="qw-hd"><span class="qw-t">cut a fiber and watch the network reroute</span><span class="qw-s">click a fiber in the diagram, or use the buttons. every fidelity and rate below is measured in the paper, not modelled.</span><span class="qw-lg"><i style="background:#e2603f"></i>carrying entanglement<i style="background:#4a4a4a"></i>idle or cut</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">network state</span><span class="qw-b" data-grp="st"><button type="button" data-v="ok" class="on" aria-pressed="true">all links up</button><button type="button" data-v="ac">A&rarr;C cut</button><button type="button" data-v="ab">A&rarr;B cut</button><button type="button" data-v="bc">B&harr;C cut</button></span></div></div><div class="qw-two"><div class="qw-pane"><span class="qw-pl">three buildings, one source, three fibers</span><svg class="qw-svg" viewBox="0 0 320 240" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Three building quantum network topology with clickable fibers"><g id="qwn-topo"></g></svg></div><div class="qw-pane"><span class="qw-pl">the links that changed, before and after</span><svg class="qw-svg" viewBox="0 0 320 240" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Coincidence rate before and after rerouting"><g id="qwn-bars"></g></svg></div></div><div class="qw-out"><div class="qw-oi"><span class="qw-ok">path in use</span><span class="qw-ov" id="qwn-path">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">worst rate hit</span><span class="qw-ov" id="qwn-hit">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">lowest fidelity</span><span class="qw-ov" id="qwn-fid">&mdash;</span></div><div class="qw-oi qw-grow"><span class="qw-ok">recovery time</span><span class="qw-ov qw-txt" id="qwn-t">&mdash;</span></div></div><p class="qw-n" id="qwn-note"></p><noscript><p class="qw-n">With all links up, the six links run at 9.2 to 866 coincidences per second with fidelities of 0.88 to 0.98. Losing A&rarr;C reroutes C&rsquo;s spectrum through building B and costs about 10 dB, dropping C1&ndash;C2 from 48.1 to 0.33 coincidences per second while fidelities stay within a few points. Losing A&rarr;B reroutes B&rsquo;s spectrum through building C, and because the recovered links are given the whole band to make up the loss, fidelity falls to about 0.76. Recovery takes roughly 5 seconds in both cases.</p></noscript></div>

### losing A&rarr;C

the controller sends building C's spectrum to building B, and building B passes it on. C's users are now two hops from the source.

fidelities come back at 0.947, 0.897 and 0.909, within a few points of their direct-path equivalents. rates are the casualty: A1&ndash;C1 lands at 23.9/s against 246.6/s for the comparable direct link, B2&ndash;C2 at 2.94/s against 9.8/s, and C1&ndash;C2 at 0.33/s against 48.1/s.

this is the good case, and it is worth being clear about why. the loss went up 10 dB and the fidelity barely moved, because **loss is the one impairment that does not touch the quantum state**. you lose photons, you don't corrupt the ones that survive. attenuation costs you rate and nothing else.

that is a genuinely nice property, and it is the reason a transparent optical reroute is a reasonable thing to do at all. it also runs out.

### losing A&rarr;B

the other direction is where it gets interesting.

building B's users are the ones with the 20% APDs. rerouting them through C would put an already-slow link 10 dB further down, and the paper decided that was not survivable at the normal allocation. so instead of the balanced eight-channel plan, they used what they call priority allocations: give the entire band to one link at a time.

two configurations, mutually exclusive. all eight channels to A1&ndash;B2, then all eight to B2&ndash;C2. rates come back at 17.5/s and 4.92/s, which is within a factor of two or three of the direct links. the loss got compensated.

fidelity is 0.765(8) and 0.76(2).

## the trade they made

the paper explains the fidelity drop in one sentence: the larger bandwidth increases multipair emission noise inside the coincidence window.

that is worth unpacking, because it is the most useful thing in the paper and it is easy to read past.

a continuous-wave pumped SPDC source does not emit one pair at a time on request. it emits pairs at random, and occasionally two pairs land inside the same 1 ns coincidence window. when that happens, the photon Alice detects and the photon Bob detects came from different pairs, and they are not entangled with each other. they are an accidental coincidence dressed up as a real one.

the scaling is unforgiving. allocate a link `n` channels instead of one and the true coincidence rate goes up like `n`, because you are collecting more of the same pairs. but the accidentals go up like `n²`, because an accidental needs one photon from each of two independent singles streams, and both singles rates scaled with `n`:

```
    true coincidences        ∝  n
    accidental coincidences  ∝  n²
    so the ratio of garbage  ∝  n
```

so bandwidth buys rate linearly and buys noise quadratically. **the paper traded fidelity for rate, and the exchange rate is set by the pump power and the coincidence window, not by anything in the network.**

they were right to make the trade in that situation. the alternative was leaving building B dark. but it was made by hand, as a configuration choice, and nothing in the control loop knows the exchange rate exists.

### where 0.76 sits

the paper reports log-negativity of 0.62(2) and 0.61(3) ebits for the two recovered links and notes, correctly, that the states are still entangled and still useful.

model the noise as depolarizing, which for this data is a good enough approximation, and a state with fidelity `F` to a Bell state has

```
    p    =  (4F − 1) / 3          depolarizing parameter
    E_N  =  log₂( (3p + 1) / 2 )  log-negativity, in ebits
```

at `F = 0.765` that gives `E_N = 0.614`. the paper measured 0.62(2). the model is calibrated.

so run it the other way and ask where the thresholds are. the same model says a Bell inequality is violated only when `p > 1/√2`, which is

```
    F  >  0.780
```

**both recovered links land just under it.** 0.765 and 0.76.

that is not a catastrophe. log-negativity is positive, the entanglement is distillable, and plenty of protocols work fine below the CHSH line. but device-independent protocols do not, and any user who was relying on a Bell violation as their security argument just silently lost it. the network came back up. the *service* did not come back the same.

and the network cannot tell. the health signal in this system is a singles count threshold. singles were fine. they were better than fine, because the recovery deliberately widened the band to push them up.

<div class="qw" id="qw-band" data-qw><div class="qw-hd"><span class="qw-t">what widening the band actually buys</span><span class="qw-s">rate rises linearly, multipair noise rises quadratically. dots are the paper&rsquo;s measured points; the curves are a depolarizing model fitted to them.</span><span class="qw-lg"><i style="background:#32c29e"></i>coincidence rate<i style="background:#6ba3f0"></i>ebits per second<i style="background:#e2603f"></i>fidelity</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">jump to</span><span class="qw-b" data-grp="ps"><button type="button" data-v="2">balanced plan, 2 ch</button><button type="button" data-v="8" class="on" aria-pressed="true">recovery plan, all 8 ch</button><button type="button" data-v="16">max ebits/s</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">25 GHz channel pairs given to one link <b id="qwb-nv">8</b></span><input type="range" id="qwb-n" min="1" max="24" step="1" value="8"></label></div><div class="qw-two"><div class="qw-pane"><span class="qw-pl">fidelity, and the thresholds it crosses</span><svg class="qw-svg" viewBox="0 0 320 190" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Fidelity versus allocated bandwidth"><g id="qwb-fid"></g></svg></div><div class="qw-pane"><span class="qw-pl">raw rate against useful rate</span><svg class="qw-svg" viewBox="0 0 320 190" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Coincidence rate and ebits per second versus allocated bandwidth"><g id="qwb-rate"></g></svg></div></div><div class="qw-out"><div class="qw-oi"><span class="qw-ok">fidelity</span><span class="qw-ov" id="qwb-f">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">ebits per pair</span><span class="qw-ov" id="qwb-e">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">ebits per second</span><span class="qw-ov" id="qwb-es">&mdash;</span></div><div class="qw-oi qw-grow"><span class="qw-ok">what a user can still do with it</span><span class="qw-ov qw-txt" id="qwb-v">&mdash;</span></div></div><p class="qw-n" id="qwb-note"></p><noscript><p class="qw-n">Allocating one link the whole 8-channel band raises its coincidence rate roughly eightfold but drops fidelity from about 0.94 to the measured 0.765, which is just below the 0.780 needed for a Bell violation under a depolarizing noise model. Log-negativity per pair falls from about 0.90 to 0.61 ebits, so the useful rate rises far more slowly than the raw rate and peaks at around 17 channels before multipair noise wins.</p></noscript></div>

look at the shape of that second panel. **raw coincidence rate is a straight line and ebits per second is not.** the paper's own metric of choice from earlier work was ebits/s, and on that metric the last few channels are nearly free of benefit while costing all of the fidelity margin.

## five seconds

the recovery time, end to end, including detection and the sequential switching of the WSS and MEMS devices, is approximately 5 seconds.

for context, the number a transport network has been engineered around since the 1980s is 50 ms. that figure comes from how long a voice circuit can be interrupted before it drops, and it survived into SDH, into optical mesh restoration, and into every carrier SLA written since. five seconds is two orders of magnitude past it.

the paper doesn't break the five seconds down, and that's the single most useful missing number in it. the components have very different speeds:

- MEMS switches are sub-millisecond to a few milliseconds. not the problem.
- LCoS wavelength selective switches take somewhere between a hundred milliseconds and a few seconds to load a new profile, because the liquid crystal has to physically settle. very likely a large share.
- detection needs enough integration time on a singles counter to distinguish a real failure from a statistical dip. on the APD users in building B, sitting at a few thousand counts per second, a confident threshold crossing is not instant.
- polarization re-settling after the photons take a physically different fiber. the paper does not say whether the analyzers needed re-optimizing, and if they did, that could dominate everything else.

each of those has a different fix, and you cannot pick one without knowing which term dominates. an oscilloscope, a counter and a long afternoon would settle it.

## what's missing

the paper is a first, and firsts get graded generously. these are the things i'd want before calling this a protection scheme rather than a demonstration of one.

**the failure detector measures the wrong thing.** a singles-count threshold is a loss-of-light alarm. it catches a cut fiber. it does not catch a link that is passing photons and no longer passing entanglement, which is exactly what a polarization drift, a source degradation, or a widened allocation produces. the A&rarr;B recovery is the proof: the network restored a link to 0.76 fidelity and had no mechanism that could have noticed. **the health metric and the service metric are different quantities, and only one of them is being watched.**

**and it cannot localize.** a threshold crossing at C1 says something upstream is broken. it does not say what. classical transport spent thirty years building an alarm hierarchy (loss of signal, loss of frame, alarm indication signal, remote defect indication) precisely so that one fault raises one alarm at the right place instead of every downstream node reporting a failure simultaneously. a quantum network with more than three buildings will need the same discipline, and there is nothing here yet.

**the protection is 1:1 and dedicated.** every building pair has its own fiber, standing by. that is fine for a triangle. it is `N(N−1)/2` fibers for a full mesh, and the entire history of optical network design is the story of moving away from dedicated spare capacity toward shared pools. the quantum version has a shared resource classical networks don't: spectrum. the spare capacity to pool is unallocated channels on the switch, not dark fiber. nobody has written that problem down.

**shared risk is unaccounted for.** the paper notes the fibers "pass through several telecommunication rooms" before reaching their destination. if the A&rarr;B and A&rarr;C fibers leave building A through the same duct, they are one fiber wearing two names, and the protection is decorative. classical planning calls this a shared risk link group and treats an SRLG-diverse path as the minimum bar for calling something protected. establishing it needs a duct map and, realistically, [OTDR traces]({% post_url 2025-09-17-Fiber-and-OTDR-testing-choosing-the-right-settings %}) to confirm the physical routes actually diverge.

**the source is a single point of failure.** one PPLN waveguide in building A feeds the entire network. every fiber is protected and the thing generating the entanglement is not. in classical terms they hardened the links and left the head end alone.

**degraded-mode policy is manual.** in the A&rarr;B recovery, the two priority allocations are mutually exclusive: the network can serve A1&ndash;B2 or B2&ndash;C2, not both, and which one is a human decision made at configuration time. a real network needs this as policy, with committed rates, preemptable classes and an admission decision that comes with a reason attached, rather than as a pair of profiles someone loads by hand.

**one fault, and only one.** the triangle survives any single cut and no double cut. that is a fine place to start, and it is worth stating explicitly rather than leaving "resilient" to do the work.

**"multihop" is doing double duty.** in most of the quantum networking literature, a hop implies a repeater: a node with a memory that does an entanglement swap, and whose whole purpose is that the loss budget starts over on the other side. nothing here swaps. the photon physically transits building B's WSS on its way to C, and pays 10 dB for the privilege. this is an optical bypass through a ROADM, which is a completely legitimate and useful thing, but it is a different thing. the paper's discussion section is careful about this. it says the term refers to architectural capability rather than a fixed number of hops. the title is not so careful.

## what i'd build next

**1. publish the five-second breakdown, then attack the biggest term.** detection, WSS settling, MEMS, polarization re-lock. four numbers.

**2. pre-load the restoration profile so the slow device is out of the failure path.** if the WSS is the bottleneck, don't reprogram it during an outage. provision the restoration allocation on a spare WSS output at commissioning time and leave it loaded, so that recovery is a MEMS flip and nothing else. you pay in spectrum, because the standby allocation occupies channels that carry nothing until they're needed. that is precisely the 1+1 versus 1:1 trade from classical protection, translated into the spectral domain, and as far as i can tell nobody has posed it that way. **the question "how much spectrum should a network hold in reserve for restoration" has a clean formulation and no answer in the literature.**

**3. alarm on entanglement, not on photons.** full tomography is 36 settings and hours, so it cannot be the health signal. but you don't need it. a two-basis correlation check, or a coincidence-to-accidental ratio monitor, is cheap and continuous and would have caught the 0.76. better still, the [in situ process tomography work](https://arxiv.org/abs/2510.26034) gives a live quantum map of a channel with a sliding window, including noise the classical polarization tracker cannot see. that is a control-plane input, and it is sitting unused. the ORNL group has also started on [what a standardized quantum network metrics set should even contain](https://arxiv.org/abs/2607.05642), which is the other half of the same problem.

**4. put the multipair term in the objective function.** the allocator maximizes rate under a fidelity constraint. but as shown above, the fidelity of a link is a function of *how much bandwidth you gave it*, and that coupling isn't in any of the routing-and-spectrum-allocation formulations. it is one analytic term. adding it turns "restore the link" into "restore the link to a stated service class", which is the difference between the two.

**5. formulate shared restoration in spectrum.** classical spare capacity allocation is a well-studied problem with decades of heuristics. the quantum version has a different shared resource and an extra constraint, and it is a straightforward port for somebody who knows both literatures.

**6. protect the source.** two sources, in different buildings, and a controller that treats source diversity as another dimension of the allocation. the [2026 routing and spectrum allocation work](https://arxiv.org/abs/2607.15465) already handles multiple sources and non-colliding frequency bin assignment, so the solver side exists. what's missing is the failover.

**7. put the coexistence noise back in.** this network keeps quantum and classical on separate fibers, which is what makes the reroute clean. share the fiber and the noise floor on a restoration path depends on which classical channels are lit on the glass you just rerouted onto, because spontaneous Raman scattering dumps noise across tens of THz. **the restoration decision then depends on the classical traffic matrix**, which is a coupling nobody has written down. the [same group's coexistence work](https://doi.org/10.1016/j.pquantelec.2025.100586) has the noise model.

**8. and eventually, hops that actually reset the budget.** the 10 dB per hop is a consequence of transparency. a node with a quantum memory that performs an entanglement swap breaks the multiplication instead of compounding it, and the routing problem changes character completely: you would route toward nodes that have memories rather than away from extra hops. that used to be a lab-only capability. in february 2026 [Qunnect and Cisco ran entanglement swapping over 17.6 km of deployed metro fiber](https://www.qunnect.inc/press-releases/2026-02-18) between Brooklyn and Manhattan at 5,400 swapped pairs per hour with polarization fidelity above 99%, using independent sources rather than a shared master laser. 5,400 an hour is 1.5 a second, against 9.8 to 866 a second for the direct links here, so a swap is still two to three orders of magnitude slower than just sending the photon through. transparent bypass is the right answer today. it will not be forever, and the control plane built here is the part that carries over.

## the short version

- this is the first quantum network that reroutes entanglement around a broken fiber instead of waiting for someone with a screwdriver. six users, three buildings, one source, an SDN controller driving wavelength selective switches and MEMS switches.
- a rerouted hop costs about **10 dB**, almost all of it component insertion loss rather than fiber. at campus scale the glass is free and the switches are not.
- because a coincidence needs both photons, **one photon on the detour costs about 10x and two photons cost well over 100x**. the C1&ndash;C2 link fell from 48.1 to 0.33 coincidences per second, a factor of 146.
- loss costs rate and not fidelity, which is why a transparent optical bypass works at all. attenuation removes photons; it does not corrupt the survivors.
- in the harder recovery they gave one link the whole band to compensate for the loss. rate came back within a factor of two to three. **fidelity fell to 0.76**, because bandwidth buys true pairs linearly and accidental pairs quadratically.
- 0.76 is just below the ~0.780 needed for a Bell violation under a depolarizing model. still entangled, still distillable at ~0.62 ebits, but a device-independent user silently lost their security argument and the network had no way to know.
- **recovery takes about 5 seconds**, against the 50 ms that transport networks have been engineered around for forty years. the paper doesn't say which component owns the five seconds.
- the failure detector is a singles-count threshold. it catches a cut fiber and cannot catch a link that passes photons but no longer passes entanglement, which is exactly what the recovery produced.
- protection is dedicated 1:1 on pre-built fiber, single-fault only, with no shared risk analysis, no source redundancy, and degraded-mode priorities chosen by hand.
- "multihop" here means transparent optical bypass, not repeater hops. nothing swaps, so nothing resets the loss budget. the paper's discussion says this; the title doesn't.

the thing i keep coming back to is that this paper's real contribution isn't the reroute. it's that it forces the question of what "up" means on a quantum link. on a classical link, up is unambiguous: light is arriving, frames are checksumming, done. here the network restored service, reported healthy, and delivered a state that a whole class of protocols can no longer use. *carrying photons* and *carrying entanglement* are two different conditions, and this is the first paper that puts a measured number on both of them at the same time. everything an operations team would want next, the alarms, the thresholds, the service classes, the escalation, follows from taking that distinction seriously.

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
.qw-fib{cursor:pointer}
.qw-fib:hover .qw-fh{stroke-opacity:.35}
.qw-node{pointer-events:none}
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
  function setGroup(root, grp, v){
    Array.prototype.forEach.call(root.querySelectorAll('[data-grp="'+grp+'"] button'), function(b){
      var on = b.getAttribute('data-v') === String(v);
      b.classList.toggle('on', on);
      if (on) b.setAttribute('aria-pressed','true'); else b.removeAttribute('aria-pressed');
    });
  }
  function offGroup(root, grp){
    Array.prototype.forEach.call(root.querySelectorAll('[data-grp="'+grp+'"] button'), function(b){
      b.classList.remove('on'); b.removeAttribute('aria-pressed');
    });
  }
  function dur(s){
    if (s < 90) return s.toFixed(0) + ' s';
    if (s < 5400) return (s/60).toFixed(s<600?1:0) + ' min';
    if (s < 172800) return (s/3600).toFixed(s<36000?1:0) + ' h';
    return (s/86400).toFixed(1) + ' days';
  }
  function rt(x){ return x >= 100 ? Math.round(x).toLocaleString() : (x >= 10 ? x.toFixed(1) : x.toFixed(2)); }

  /* ---------- 1. what an extra hop costs ---------- */
  (function(){
    var H = document.getElementById('qw-hop');
    if (!H) return;
    var BASE = 48.1, PERKM = 0.2, TARGET = 1000, SETTINGS = 36;
    var sh = H.querySelector('#qwp-h'), sl = H.querySelector('#qwp-l'), sd = H.querySelector('#qwp-d');
    var arms = 1, g = H.querySelector('#qwp-diag');
    var PRESET = { direct:{h:0,arms:1}, one:{h:1,arms:1}, both:{h:1,arms:2} };

    function diag(hops, per, km){
      clear(g);
      var CX = 320, CY = 96, SPAN = 250;
      g.appendChild(el('rect',{x:CX-46,y:CY-17,width:92,height:34,rx:4,fill:'#242424',stroke:'#5a5a5a','stroke-width':1}));
      g.appendChild(el('text',{x:CX,y:CY-2,'text-anchor':'middle',fill:'#e6e6e6','font-size':10.5},'source'));
      g.appendChild(el('text',{x:CX,y:CY+11,'text-anchor':'middle',fill:'#8f8f8f','font-size':9.5},'WSS1, building A'));
      [-1,1].forEach(function(dir,i){
        var n = (i === 0 || arms === 2) ? hops : 0;
        var col = n ? '#f2a03d' : '#32c29e';
        var x0 = CX + dir*46, x1 = CX + dir*SPAN, y = CY + dir*44;
        g.appendChild(el('line',{x1:x0,y1:CY,x2:x0+dir*18,y2:y,stroke:col,'stroke-width':2.2}));
        g.appendChild(el('line',{x1:x0+dir*18,y1:y,x2:x1,y2:y,stroke:col,'stroke-width':2.2}));
        var seg = (x1 - (x0+dir*18));
        for (var k=1;k<=n;k++){
          var hx = x0 + dir*18 + seg*(k/(n+1));
          g.appendChild(el('rect',{x:hx-13,y:y-11,width:26,height:22,rx:3,fill:'#2a2118',stroke:'#f2a03d','stroke-width':1.2}));
          g.appendChild(el('text',{x:hx,y:y+4,'text-anchor':'middle',fill:'#f2a03d','font-size':9},'WSS'));
        }
        g.appendChild(el('circle',{cx:x1,cy:y,r:9,fill:'#212121',stroke:col,'stroke-width':2}));
        g.appendChild(el('text',{x:x1,y:y+dir*26,'text-anchor':'middle',fill:'#cfcfcf','font-size':10.5,'class':'qw-node'}, i===0?'photon 1':'photon 2'));
        var add = n*per + n*km*PERKM;
        g.appendChild(el('text',{x:x1,y:y+dir*39,'text-anchor':'middle',fill:col,'font-size':10,'class':'qw-node'},
          n ? '+' + add.toFixed(1) + ' dB' : 'no detour'));
      });
      g.appendChild(el('text',{x:CX,y:CY+72,'text-anchor':'middle',fill:'#8f8f8f','font-size':10},
        'a coincidence needs both circles to click inside the same 1 ns window'));
    }
    function upd(){
      var hops = +sh.value, per = +sl.value, km = +sd.value;
      var addOne = hops*per + hops*km*PERKM;
      var a1 = addOne, a2 = arms === 2 ? addOne : 0;
      var rate = BASE*Math.pow(10, -(a1+a2)/10);
      H.querySelector('#qwp-hv').textContent = hops;
      H.querySelector('#qwp-lv').textContent = per.toFixed(1) + ' dB';
      H.querySelector('#qwp-dv').textContent = km.toFixed(1) + ' km';
      H.querySelector('#qwp-a1').textContent = a1 ? '+' + a1.toFixed(1) + ' dB' : '0 dB';
      H.querySelector('#qwp-a2').textContent = a2 ? '+' + a2.toFixed(1) + ' dB' : '0 dB';
      H.querySelector('#qwp-r').textContent = rt(rate) + '/s';
      var tt = H.querySelector('#qwp-t');
      var secs = SETTINGS*TARGET/rate;
      tt.textContent = dur(secs);
      tt.style.color = secs < 3600 ? '#32c29e' : (secs < 86400 ? '#f2a03d' : '#e2603f');
      diag(hops, per, km);
      var drop = Math.pow(10,(a1+a2)/10);
      var note;
      if (!hops) note = 'No detour. This is the direct C1&ndash;C2 link at its measured <b>48.1 coincidences per second</b>, the reference every other case is scaled from.';
      else if (arms === 1) note = 'One photon takes <b>' + hops + ' extra hop' + (hops>1?'s':'') + '</b> and the other goes direct, so the link loses a factor of <b>' + rt(drop) + '</b>. The paper&rsquo;s A1&ndash;C1 case is one hop: measured 246.6/s down to 23.9/s, a factor of 10.3.';
      else note = 'Both photons take <b>' + hops + ' extra hop' + (hops>1?'s':'') + '</b>, so the loss is paid twice and the rate falls by a factor of <b>' + rt(drop) + '</b>. This is the C1&ndash;C2 case: two users in the same building, both fed through the detour, measured at 48.1/s down to <b>0.33/s</b>.';
      H.querySelector('#qwp-note').innerHTML = note +
        ' Time assumes you want <b>1,000 coincidences in each of 36 tomography settings</b>, and fiber at 0.2 dB/km.';
    }
    sh.addEventListener('input', function(){ offGroup(H,'ps'); upd(); });
    sl.addEventListener('input', function(){ offGroup(H,'ps'); upd(); });
    sd.addEventListener('input', function(){ offGroup(H,'ps'); upd(); });
    btnGroup(H,'ar', function(v){ arms = +v; offGroup(H,'ps'); upd(); });
    btnGroup(H,'ps', function(v){
      var p = PRESET[v];
      sh.value = p.h; arms = p.arms; setGroup(H,'ar',arms); upd();
    });
    upd();
  })();

  /* ---------- 2. cut a fiber, watch it reroute ---------- */
  (function(){
    var H = document.getElementById('qw-net');
    if (!H) return;
    var NODE = { A:{x:160,y:52,lbl:'A',sub:'source, A1 A2'},
                 B:{x:52,y:186,lbl:'B',sub:'B1 B2, APDs'},
                 C:{x:268,y:186,lbl:'C',sub:'C1 C2'} };
    var FIB = [ {id:'ab',a:'A',b:'B',km:'250 m'}, {id:'ac',a:'A',b:'C',km:'1.2 km'}, {id:'bc',a:'B',b:'C',km:'930 m'} ];
    var BASE = { 'A1-A2':[0.9504,866.5], 'B1-B2':[0.887,9.16], 'C1-C2':[0.932,48.1],
                 'A1-B1':[0.965,43.4], 'A2-C1':[0.978,246.6], 'B1-C1':[0.884,9.8] };
    var CASE = {
      ok: { cut:null, active:['ab','ac','bc'], path:'direct, star',
            rows:[] },
      ac: { cut:'ac', active:['ab','bc'], path:'A &rarr; B &rarr; C',
            rows:[['A1–C1','A2–C1',0.978,246.6,0.947,23.9],
                  ['B2–C2','B1–C1',0.884,9.8,0.897,2.94],
                  ['C1–C2','C1–C2',0.932,48.1,0.909,0.33]] },
      ab: { cut:'ab', active:['ac','bc'], path:'A &rarr; C &rarr; B',
            rows:[['A1–B2','A1–B1',0.965,43.4,0.765,17.5],
                  ['B2–C2','B1–C1',0.884,9.8,0.760,4.92]] },
      bc: { cut:'bc', active:['ab','ac'], path:'direct, no spare',
            rows:[['B1–C1','B1–C1',0.884,9.8,0,0]] }
    };
    var st = 'ok', gT = H.querySelector('#qwn-topo'), gB = H.querySelector('#qwn-bars');

    function topo(){
      clear(gT);
      var c = CASE[st];
      FIB.forEach(function(f){
        var A = NODE[f.a], B = NODE[f.b], on = c.active.indexOf(f.id) >= 0;
        var grp = el('g',{'class':'qw-fib',role:'button',tabindex:0});
        grp.appendChild(el('line',{x1:A.x,y1:A.y,x2:B.x,y2:B.y,stroke:'#32c29e','stroke-width':14,
          'stroke-opacity':0,'stroke-linecap':'round','class':'qw-fh'}));
        grp.appendChild(el('line',{x1:A.x,y1:A.y,x2:B.x,y2:B.y,stroke:on?'#e2603f':'#4a4a4a','stroke-width':on?3:2,
          'stroke-dasharray':on?'':'5 5'}));
        var mx = (A.x+B.x)/2, my = (A.y+B.y)/2;
        if (!on){
          grp.appendChild(el('line',{x1:mx-9,y1:my-9,x2:mx+9,y2:my+9,stroke:'#e2603f','stroke-width':2.6}));
          grp.appendChild(el('line',{x1:mx-9,y1:my+9,x2:mx+9,y2:my-9,stroke:'#e2603f','stroke-width':2.6}));
        }
        var tx = mx + (f.id==='bc'?0:(f.id==='ab'?-30:30)), ty = my + (f.id==='bc'?22:4);
        grp.appendChild(el('text',{x:tx,y:ty,'text-anchor':'middle',fill:'#8f8f8f','font-size':9.5,'class':'qw-node'}, f.km));
        grp.appendChild(el('title',{}, on ? 'click to cut the ' + f.a + '–' + f.b + ' fiber' : 'click to restore'));
        function toggle(){ st = (st === f.id) ? 'ok' : f.id; setGroup(H,'st',st); draw(); }
        grp.addEventListener('click', toggle);
        grp.addEventListener('keydown', function(e){ if (e.key==='Enter'||e.key===' '){ e.preventDefault(); toggle(); } });
        gT.appendChild(grp);
      });
      if (st !== 'ok' && st !== 'bc'){
        var v = st === 'ac' ? ['A','B','C'] : ['A','C','B'];
        var d = 'M' + NODE[v[0]].x + ' ' + NODE[v[0]].y + 'L' + NODE[v[1]].x + ' ' + NODE[v[1]].y
              + 'L' + NODE[v[2]].x + ' ' + NODE[v[2]].y;
        gT.appendChild(el('path',{d:d,fill:'none',stroke:'#f2a03d','stroke-width':2,'stroke-dasharray':'2 6',
          'stroke-linecap':'round','stroke-linejoin':'round'}));
      }
      ['A','B','C'].forEach(function(k){
        var N = NODE[k];
        gT.appendChild(el('circle',{cx:N.x,cy:N.y,r:22,fill:'#212121',stroke:k==='A'?'#32c29e':'#5a5a5a','stroke-width':2,'class':'qw-node'}));
        gT.appendChild(el('text',{x:N.x,y:N.y+5,'text-anchor':'middle',fill:'#fff','font-size':15,'font-weight':600,'class':'qw-node'}, N.lbl));
        gT.appendChild(el('text',{x:N.x,y:N.y+(k==='A'?-30:37),'text-anchor':'middle',fill:'#8f8f8f','font-size':9.5,'class':'qw-node'}, N.sub));
      });
    }
    function bars(){
      clear(gB);
      var c = CASE[st];
      if (!c.rows.length){
        var keys = Object.keys(BASE), X0 = 62, YB = 196, YT = 22, bw = (300-X0)/keys.length;
        [1,10,100,1000].forEach(function(v){
          var y = YB - Math.log(v)/Math.LN10/3*(YB-YT);
          gB.appendChild(el('line',{x1:X0,y1:y,x2:304,y2:y,stroke:'#2a2a2a','stroke-width':1}));
          gB.appendChild(el('text',{x:X0-6,y:y+4,'text-anchor':'end',fill:'#7d7d7d','font-size':9}, v));
        });
        keys.forEach(function(k,i){
          var r = BASE[k][1], h = Math.max(3, Math.log(r)/Math.LN10/3*(YB-YT));
          var x = X0 + i*bw + bw*0.2;
          gB.appendChild(el('rect',{x:x,y:YB-h,width:bw*0.6,height:h,fill:'#32c29e','fill-opacity':0.75,stroke:'#32c29e','stroke-width':1,rx:2}));
          gB.appendChild(el('text',{x:x+bw*0.3,y:YB+14,'text-anchor':'middle',fill:'#9a9a9a','font-size':8.5,
            transform:'rotate(-40 '+(x+bw*0.3)+' '+(YB+14)+')'}, k));
        });
        gB.appendChild(el('text',{x:X0-6,y:YT-6,'text-anchor':'end',fill:'#8f8f8f','font-size':9},'pairs/s'));
        gB.appendChild(el('text',{x:184,y:YT-6,'text-anchor':'middle',fill:'#8f8f8f','font-size':10},'all six links, healthy'));
        return;
      }
      var X0 = 70, YB = 180, YT = 26, rh = (YB-YT)/c.rows.length;
      function bx(r){ return X0 + Math.max(0, Math.log(Math.max(r,0.05)/0.05)/Math.log(1000/0.05))*(300-X0); }
      c.rows.forEach(function(r,i){
        var y = YT + i*rh + 6;
        gB.appendChild(el('text',{x:X0-6,y:y+8,'text-anchor':'end',fill:'#cfcfcf','font-size':10}, r[0]));
        if (r[1] !== r[0]) gB.appendChild(el('text',{x:X0-6,y:y+20,'text-anchor':'end',fill:'#7d7d7d','font-size':8.5}, 'vs ' + r[1]));
        gB.appendChild(el('rect',{x:X0,y:y,width:Math.max(2,bx(r[3])-X0),height:9,fill:'#32c29e','fill-opacity':0.8,rx:2}));
        gB.appendChild(el('text',{x:bx(r[3])+5,y:y+8,fill:'#32c29e','font-size':9}, rt(r[3])+'/s'));
        if (r[5] > 0){
          gB.appendChild(el('rect',{x:X0,y:y+13,width:Math.max(2,bx(r[5])-X0),height:9,fill:'#f2a03d','fill-opacity':0.85,rx:2}));
          gB.appendChild(el('text',{x:bx(r[5])+5,y:y+21,fill:'#f2a03d','font-size':9}, rt(r[5])+'/s'));
        } else {
          gB.appendChild(el('text',{x:X0+4,y:y+21,fill:'#e2603f','font-size':9},'dark, no restoration path'));
        }
      });
      gB.appendChild(el('text',{x:X0,y:YT-8,fill:'#32c29e','font-size':9.5},'before'));
      gB.appendChild(el('text',{x:X0+42,y:YT-8,fill:'#f2a03d','font-size':9.5},'after rerouting'));
      gB.appendChild(el('text',{x:184,y:YB+18,'text-anchor':'middle',fill:'#8f8f8f','font-size':10},'coincidences per second, log scale'));
    }
    function draw(){
      var c = CASE[st];
      topo(); bars();
      H.querySelector('#qwn-path').innerHTML = c.path;
      var hit = '—', fid = '—', tt = H.querySelector('#qwn-t'), note;
      if (st === 'ok'){
        hit = 'none'; fid = '0.884';
        tt.textContent = 'nothing to recover'; tt.style.color = '#32c29e';
        note = 'The star topology, WSS1 in building A feeding WSS2 and WSS3 directly. Six links running from <b>9.2 to 866 coincidences per second</b>, fidelities <b>0.884 to 0.978</b>. Click any fiber to cut it.';
      } else if (st === 'bc'){
        hit = 'B1–C1 dark'; fid = 'n/a';
        tt.textContent = 'no path exists'; tt.style.color = '#e2603f';
        note = 'Cutting B&harr;C is the case the paper does not test. The links from the source to A, B and C all survive, but <b>every B&harr;C link goes dark</b> and there is no third route to restore them. It also removes the spare path that both other recoveries depend on, so from here a second cut takes a whole building offline. The triangle protects the source-side links and nothing else.';
      } else {
        var worst = c.rows.reduce(function(m,r){ return (r[3]/r[5] > m[3]/m[5]) ? r : m; });
        hit = rt(worst[3]/worst[5]) + 'x on ' + worst[0];
        fid = Math.min.apply(null, c.rows.map(function(r){ return r[4]; })).toFixed(3);
        tt.textContent = '~5 seconds'; tt.style.color = '#f2a03d';
        if (st === 'ac'){
          note = 'Building C&rsquo;s spectrum is rerouted through building B, adding about <b>10 dB</b>. Fidelities hold up, within a few points of the direct links, because loss removes photons without corrupting the ones that arrive. Rates do not: C1&ndash;C2 falls from <b>48.1 to 0.33 per second</b>, because both of its photons take the detour.';
        } else {
          note = 'Building B&rsquo;s spectrum is rerouted through building C. Because B&rsquo;s users have the slow APDs, the paper compensates by giving <b>the entire band to one link at a time</b> instead of the balanced plan. Rates come back within a factor of two or three. Fidelity falls to <b>0.765 and 0.760</b>, just under the ~0.780 a Bell violation needs, and the two configurations are mutually exclusive: the network serves one of these links or the other.';
        }
      }
      H.querySelector('#qwn-hit').innerHTML = hit;
      H.querySelector('#qwn-fid').textContent = fid;
      H.querySelector('#qwn-note').innerHTML = note;
    }
    btnGroup(H,'st', function(v){ st = v; draw(); });
    draw();
  })();

  /* ---------- 3. what widening the band buys ---------- */
  (function(){
    var H = document.getElementById('qw-band');
    if (!H) return;
    var F0 = 0.97, K = 0.04976, R1 = 17.5/8, NMAX = 24, CHSH = 0.7803;
    var sn = H.querySelector('#qwb-n'), gF = H.querySelector('#qwb-fid'), gR = H.querySelector('#qwb-rate');
    function fid(n){ var e = K*n/(1+K*n); return F0 - e*(F0 - 0.25); }
    function eN(F){ var p = (4*F-1)/3; return p <= 1/3 ? 0 : Math.log((3*p+1)/2)/Math.LN2; }
    function fidPane(n){
      clear(gF);
      var X0 = 34, X1 = 306, Y0 = 16, Y1 = 142;
      function cx(v){ return X0 + (v-1)/(NMAX-1)*(X1-X0); }
      function cy(v){ return Y1 - (v-0.4)/0.62*(Y1-Y0); }
      [0.5,0.6,0.7,0.8,0.9,1.0].forEach(function(v){
        gF.appendChild(el('line',{x1:X0,y1:cy(v),x2:X1,y2:cy(v),stroke:'#2a2a2a','stroke-width':1}));
        gF.appendChild(el('text',{x:X0-6,y:cy(v)+3.5,'text-anchor':'end',fill:'#7d7d7d','font-size':9}, v.toFixed(1)));
      });
      [[CHSH,'#d8c257','Bell violation, 0.780'],[0.5,'#e2603f','separable below 0.5']].forEach(function(t){
        gF.appendChild(el('line',{x1:X0,y1:cy(t[0]),x2:X1,y2:cy(t[0]),stroke:t[1],'stroke-width':1.2,'stroke-dasharray':'5 4'}));
        gF.appendChild(el('text',{x:X1,y:cy(t[0])-4,'text-anchor':'end',fill:t[1],'font-size':9}, t[2]));
      });
      [1,8,16,24].forEach(function(t){
        gF.appendChild(el('text',{x:cx(t),y:Y1+15,'text-anchor':'middle',fill:'#7d7d7d','font-size':9}, t));
      });
      gF.appendChild(el('text',{x:(X0+X1)/2,y:Y1+31,'text-anchor':'middle',fill:'#8f8f8f','font-size':10},'25 GHz channel pairs allocated'));
      var d = '';
      for (var v=1;v<=NMAX;v+=0.25) d += (v===1?'M':'L') + cx(v).toFixed(1) + ' ' + cy(fid(v)).toFixed(1);
      gF.appendChild(el('path',{d:d,fill:'none',stroke:'#e2603f','stroke-width':2.2,'stroke-linecap':'round'}));
      [[8,0.765],[8,0.760]].forEach(function(m){
        gF.appendChild(el('circle',{cx:cx(m[0]),cy:cy(m[1]),r:3.4,fill:'#fff','fill-opacity':0.9}));
      });
      gF.appendChild(el('circle',{cx:cx(n),cy:cy(fid(n)),r:4.5,fill:'#fff',stroke:'#e2603f','stroke-width':2}));
    }
    function ratePane(n){
      clear(gR);
      var X0 = 34, X1 = 306, Y0 = 16, Y1 = 142;
      var RMAX = R1*NMAX;
      function cx(v){ return X0 + (v-1)/(NMAX-1)*(X1-X0); }
      function cy(v){ return Y1 - v/RMAX*(Y1-Y0); }
      [0,0.25,0.5,0.75,1].forEach(function(fr){
        gR.appendChild(el('line',{x1:X0,y1:cy(fr*RMAX),x2:X1,y2:cy(fr*RMAX),stroke:'#2a2a2a','stroke-width':1}));
        gR.appendChild(el('text',{x:X0-6,y:cy(fr*RMAX)+3.5,'text-anchor':'end',fill:'#7d7d7d','font-size':9}, Math.round(fr*RMAX)));
      });
      [1,8,16,24].forEach(function(t){
        gR.appendChild(el('text',{x:cx(t),y:Y1+15,'text-anchor':'middle',fill:'#7d7d7d','font-size':9}, t));
      });
      gR.appendChild(el('text',{x:(X0+X1)/2,y:Y1+31,'text-anchor':'middle',fill:'#8f8f8f','font-size':10},'25 GHz channel pairs allocated'));
      var d1 = '', d2 = '', best = 1, bv = 0;
      for (var v=1;v<=NMAX;v+=0.25){
        var r = R1*v, e = r*eN(fid(v));
        if (e > bv){ bv = e; best = v; }
        d1 += (v===1?'M':'L') + cx(v).toFixed(1) + ' ' + cy(r).toFixed(1);
        d2 += (v===1?'M':'L') + cx(v).toFixed(1) + ' ' + cy(e).toFixed(1);
      }
      gR.appendChild(el('path',{d:d1,fill:'none',stroke:'#32c29e','stroke-width':2.2,'stroke-linecap':'round'}));
      gR.appendChild(el('path',{d:d2,fill:'none',stroke:'#6ba3f0','stroke-width':2.2,'stroke-linecap':'round'}));
      gR.appendChild(el('line',{x1:cx(best),y1:Y0,x2:cx(best),y2:Y1,stroke:'#4a4a4a','stroke-width':1,'stroke-dasharray':'4 4'}));
      gR.appendChild(el('text',{x:cx(best)-4,y:Y0+10,'text-anchor':'end',fill:'#8f8f8f','font-size':9},'peak ebits/s'));
      gR.appendChild(el('text',{x:X0+4,y:Y0+11,fill:'#32c29e','font-size':9},'pairs/s'));
      gR.appendChild(el('circle',{cx:cx(n),cy:cy(R1*n),r:4,fill:'#fff',stroke:'#32c29e','stroke-width':2}));
      gR.appendChild(el('circle',{cx:cx(n),cy:cy(R1*n*eN(fid(n))),r:4,fill:'#fff',stroke:'#6ba3f0','stroke-width':2}));
      return best;
    }
    function verdict(F){
      if (F >= 0.90) return ['everything, including device-independent protocols','#32c29e'];
      if (F >= CHSH) return ['still violates a Bell inequality, with no margin','#d8c257'];
      if (F >= 0.60) return ['entangled and distillable, but no Bell violation','#f2a03d'];
      if (F > 0.50) return ['barely entangled, purification first','#e2603f'];
      return ['separable, this is not a quantum link','#e2603f'];
    }
    function upd(){
      var n = +sn.value, F = fid(n), e = eN(F), r = R1*n;
      H.querySelector('#qwb-nv').textContent = n;
      H.querySelector('#qwb-f').textContent = F.toFixed(3);
      H.querySelector('#qwb-e').textContent = e.toFixed(3);
      H.querySelector('#qwb-es').textContent = (r*e).toFixed(1) + '/s';
      var vd = verdict(F), ve = H.querySelector('#qwb-v');
      ve.textContent = vd[0]; ve.style.color = vd[1];
      fidPane(n); var best = ratePane(n);
      H.querySelector('#qwb-note').innerHTML =
        'At <b>' + n + ' channel pair' + (n>1?'s':'') + '</b> the link carries <b>' + r.toFixed(1) + ' pairs/s</b> at fidelity <b>' + F.toFixed(3) +
        '</b>, worth <b>' + e.toFixed(2) + ' ebits</b> each and <b>' + (r*e).toFixed(1) + ' ebits/s</b> in total. ' +
        (F < CHSH ? 'That is <b>below the 0.780</b> a Bell violation needs. ' : 'That still clears the 0.780 Bell threshold. ') +
        'The model is depolarizing noise growing linearly with allocated bandwidth, fitted to the paper&rsquo;s measured endpoint (8 channels, F = 0.765, 0.62 ebits) and checked against it: the fit predicts 0.614 ebits where the paper measured 0.62(2). Raw rate is a straight line; useful rate is not, and it peaks near <b>' + Math.round(best) + ' channels</b> before multipair noise takes over.';
    }
    sn.addEventListener('input', function(){ offGroup(H,'ps'); upd(); });
    btnGroup(H,'ps', function(v){ sn.value = v; upd(); });
    upd();
  })();
})();
</script>{% endraw %}