---
layout: post
section-type: post
title: "holding entanglement together on a wire in the wind"
category: 'networking'
tags: [ 'networking', 'fiber', 'quantum' ]
---

i have now cited this paper twice in passing, once in [the fiber types post]({% post_url 2026-01-03-five-kinds-of-fiber-and-when-to-use-which %}) as the reason polarization drift is a solved problem, and once in [the eight-users post]({% post_url 2026-05-29-eight-users-on-one-chip-maximizing-the-source-side-of-a-flex-grid-quanntum-network %}) under "what happened since." it is out properly now, so it is time to write it up.

disclosure before anything else: i am on it. [Shi et al., "Entanglement distribution over a polarization-stabilized aerial fiber," *J. Opt. Commun. Netw.* 18, 813 (2026)](https://doi.org/10.1364/JOCN.592521), also on [arXiv](https://arxiv.org/abs/2601.11753). that changes what this post can be. i am not going to pretend to review it from outside, and i am going to be more specific than a reviewer could be about which numbers were decisions and which were things the fiber did to us. what it does not change is the format: there is still a "what i'd push on next" section, and some of it is aimed at my own paper.

the one-line version is that we sent one half of an entangled photon pair 62 km from NIST in Gaithersburg to the University of Maryland in College Park, over a fiber that is about 70% strung between poles, and it worked for a day. Oliver Slattery, one of the co-authors, described the link to the [NIST press office](https://www.nist.gov/news-events/news/2026/08/spooky-particles-transit-dc-suburbs-step-toward-quantum-network) as "about as bad a connection as you can possibly have," which is roughly the spirit in which it was chosen.

this post is the long version. the first section is background for people who do not do quantum. if you know what a Bell violation is, skip to *what the wind does*.

## the quantum part, for people who don't do quantum

### polarization entanglement, in one paragraph

a source makes two photons at once. the pair is prepared in the state

```
    |Φ+⟩ = (|HH⟩ + |VV⟩) / √2
```

which says: if you measure both photons in the horizontal/vertical basis you will always find them the same, and, the part that matters, that also holds in every other basis. measure both at 45°, still the same. this second half is not something a pair of classically correlated photons can do, and the whole apparatus of quantum key distribution and entanglement swapping rests on it.

we send one photon of each pair down the fiber. the other stays home.

### why polarization, and why it is the fragile choice

there are other places to hide a qubit in a photon. time-bin encoding survives fiber beautifully, which is why long-distance records tend to use it. frequency-bin is having a moment. polarization is the one that is trivial to make, trivial to measure with a waveplate and a beamsplitter, and trivial to feed into any of the standard protocols. it is also the one the fiber attacks directly.

a perfect fiber would be a perfect cylinder of perfectly isotropic glass and would leave polarization alone. real fiber has an elliptical core, sits in a duct under load, gets bent around a pole, and heats unevenly. all of that is birefringence, which means the two polarization components travel at slightly different speeds and pick up a relative phase. i covered the mechanism in [the fiber types post]({% post_url 2026-01-03-five-kinds-of-fiber-and-when-to-use-which %}); the short version is that a length of deployed fiber applies some arbitrary rotation to the polarization state of everything passing through it.

if that rotation were fixed, this would be a non-problem. you would measure it once, put a compensating rotation at the far end, and never think about it again. it is not fixed. it moves with temperature, with stress, with anything that touches the cable.

### buried fiber is a different animal

this is the part worth internalizing if you come from the classical side, where the aerial-versus-buried decision is about cost, permits and backhoes.

buried fiber sits in a duct at a stable temperature with nothing touching it. its polarization transformation drifts on a scale of hours, sometimes longer. Wengerowsky and co-authors distributed polarization entanglement over 192 km of deployed fiber with *no* active stabilization at all, which tells you how gentle a buried link can be.

aerial fiber is hanging in the air. the sun heats one side of the span. wind moves it. a truck goes past and shakes the pole. birds land on it, which is a sentence that appears in the NIST press release and is not a joke. the same transformation that took hours to move underground now moves in seconds.

so the interesting question is not "can you send entanglement over fiber," which was settled years ago. it is whether the fraction of the installed base that is strung between poles is usable at all, because in most metro areas that fraction is not small and you do not get to choose.

## what the wind does

the link is 62 km of deployed fiber between NIST Gaithersburg and UMD College Park, roughly 70% aerial. end to end loss is about 18 dB. of that, roughly 12.4 dB is the fiber doing what fiber does at 0.2 dB/km, and the other 5.6 dB is splices, patch panels and switches, which is a very ordinary ratio for a real link and worth remembering the next time somebody quotes you a length-times-attenuation budget.

before touching any of the quantum hardware we did the boring characterization: inject polarized probe light, watch the Stokes parameters come out the far end, and see how long the state stays put.

the answer depends on what time it is. at night the output polarization is stable for tens of minutes. during the day it walks tens of degrees away in a matter of seconds, and the fidelity of the transmitted state falls below 95% in under 20 seconds.

<div class="qw" id="qw-drift" data-qw><div class="qw-hd"><span class="qw-t">how fast the fiber runs away from you</span><span class="qw-s">a random-walk model pinned to the two numbers the paper prints: fidelity below 95% in under 20 s during the day, and stability over tens of minutes at night.</span><span class="qw-lg"><i style="background:#32c29e"></i>state fidelity<i style="background:#f2a03d"></i>the APC trigger threshold<i style="background:#6ba3f0"></i>the 3 s quantum window</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">what is happening to the cable</span><span class="qw-b" data-grp="cond"><button type="button" data-v="night">2 a.m., still air</button><button type="button" data-v="evening">evening, traffic dying down</button><button type="button" data-v="day" class="on" aria-pressed="true">midday, sun and trucks</button><button type="button" data-v="gust">a wind gust on the span</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">seconds since the last compensation <b id="qwr-tv">3.0 s</b></span><input type="range" id="qwr-t" min="0" max="60" step="0.2" value="3"></label><label class="qw-sr"><span class="qw-l">fidelity threshold that triggers the APC <b id="qwr-hv">98%</b></span><input type="range" id="qwr-h" min="90" max="99.5" step="0.1" value="98"></label></div><div class="qw-two"><div class="qw-pane"><span class="qw-pl">fidelity of the received state vs time</span><svg class="qw-svg" viewBox="0 0 340 170" preserveAspectRatio="xMidYMid meet" role="img" aria-label="State fidelity decaying with time since the last compensation"><g id="qwr-grid"></g><g id="qwr-curve"></g><g id="qwr-mark"></g></svg></div><div class="qw-pane"><span class="qw-pl">where the state has wandered to</span><svg class="qw-svg" viewBox="0 0 340 170" preserveAspectRatio="xMidYMid meet" role="img" aria-label="The polarization state drifting away from its starting point on the Poincare sphere"><g id="qwr-sph"></g></svg></div></div><div class="qw-out"><div class="qw-oi"><span class="qw-ok">fidelity now</span><span class="qw-ov" id="qwr-f">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">how far it moved</span><span class="qw-ov" id="qwr-a">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">time to hit the threshold</span><span class="qw-ov" id="qwr-x">&mdash;</span></div><div class="qw-oi qw-grow"><span class="qw-ok">verdict</span><span class="qw-ov qw-txt" id="qwr-v">&mdash;</span></div></div><p class="qw-n" id="qwr-note"></p><noscript><p class="qw-n">At midday the model crosses the 98% trigger threshold about 11 seconds after a compensation finishes, and 95% at about 20 seconds. At night the same two thresholds take roughly 11 and 20 minutes, a factor of 60 slower. Under a wind gust the 98% crossing comes at about 2 seconds, which is inside the 3 s window.</p></noscript></div>

the factor between day and night falls straight out of those two published numbers, and it is about 60.

the daytime average is not the number that matters, though. at midday the state takes about 11 seconds to cross the 98% threshold that triggers the compensator, so the 3 second window we leave open for entanglement has a comfortable 3.5x of margin. under a gust it crosses in about 2 seconds, which is *inside* the window. the link is not marginal on a typical afternoon. it is marginal specifically during the events that are also making the compensator time out, and that pattern comes back twice more in this post.

## a pilot tone and an inverse

the fix is the one you would guess. send a classical reference signal through the same fiber, measure what the fiber did to it, and drive an inline polarization controller until the damage is undone.

the specific arrangement here is a pair of automated polarization compensation modules built by Qunnect: an injector at the NIST end and a compensator at the UMD end.

- the injector launches about 0.5 mW of laser light into the fiber, cycling its polarization through a sequence of six designated reference states.
- the compensator measures the six states as they arrive, on a polarimeter, and computes the fidelity of each against what was sent.
- the minimum of those six fidelities becomes the error signal. a gradient descent drives the polarization controller to push it back toward one.

six reference states is more than the job strictly requires. two non-orthogonal states are mathematically sufficient to pin down the transformation. more states means a better-conditioned estimate and a slower modulation cycle, and six is where that trade bottomed out for this hardware. this is the kind of thing that reads as arbitrary in a paper and was in fact a couple of afternoons.

the loop is a threshold machine, not a continuous servo:

- check the fidelity. above 98%, do nothing, release the fiber immediately. that check takes about 50 ms.
- below 98%, run the descent until fidelity exceeds 99%, or until 55 seconds have gone by, whichever happens first.
- either way, hand the fiber back for a 3 second window, then check again.

the 55 second timeout is there so that a session cannot swallow the link forever during a bad stretch. it also, as we will get to, ends up being the most interesting number in the experiment.

## why the pilot has to sit on top of the photons

this is the constraint that shapes the whole design, and the one i have the most to say about.

the pilot tone is only useful if the fiber does the same thing to it that the fiber does to the single photons. birefringence is wavelength dependent, so that stops being true as you move them apart. we measured it: six polarization basis states across the 62 km at wavelengths from 1545 to 1555 nm, extract the transformation matrix at each, and compare.

the fidelity between transformations falls off quickly with detuning. so the pilot is generated at 1549.32 nm, the same ITU channel as the idler photons it is protecting.

<div class="qw" id="qw-pilot" data-qw><div class="qw-hd"><span class="qw-t">how far can the pilot tone sit from the photons</span><span class="qw-s">the fidelity curve is fitted to the paper&rsquo;s Fig. 3(b). turning that into an S-value is my model, not the paper&rsquo;s, and it assumes the leftover error is a rotation about an unknown axis.</span><span class="qw-lg"><i style="background:#32c29e"></i>transformation fidelity, left axis<i style="background:#c8c8c8"></i>resulting S, right axis<i style="background:#e2603f"></i>the classical bound</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">places you might put the pilot</span><span class="qw-b" data-grp="pl"><button type="button" data-v="0" class="on" aria-pressed="true">same channel, 1549.32 nm</button><button type="button" data-v="0.8">one ITU channel away</button><button type="button" data-v="1">the paper&rsquo;s 1 nm rule</button><button type="button" data-v="4">a C-band pilot at 1545</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">pilot detuning from the idler <b id="qwp-dv">0.00 nm</b></span><input type="range" id="qwp-d" min="0" max="5" step="0.02" value="0"></label></div><div class="qw-pane"><span class="qw-pl">transformation fidelity vs detuning, and what it leaves in the state</span><svg class="qw-svg" viewBox="0 0 640 230" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Polarization transformation fidelity and resulting CHSH S-value versus pilot detuning"><g id="qwp-grid"></g><g id="qwp-curves"></g><g id="qwp-mark"></g></svg></div><div class="qw-out"><div class="qw-oi"><span class="qw-ok">transformation fidelity</span><span class="qw-ov" id="qwp-f">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">error left behind</span><span class="qw-ov" id="qwp-e">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">visibility</span><span class="qw-ov" id="qwp-vis">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">S</span><span class="qw-ov" id="qwp-s">&mdash;</span></div><div class="qw-oi qw-grow"><span class="qw-ok">verdict</span><span class="qw-ov qw-txt" id="qwp-v">&mdash;</span></div></div><p class="qw-n" id="qwp-note"></p><noscript><p class="qw-n">Fitted to Fig. 3(b), the transformation fidelity is 99.7% one ITU channel away and 99.5% at 1 nm, which costs about 0.01 of an S-value. It takes roughly 4 nm of detuning before the leftover rotation on its own pushes S through the classical bound of 2.</p></noscript></div>

that is not quite what the paper says, and the difference is the part i want to argue with.

the paper's stated rule is that the reference should sit within 1 nm of the photons. that is sound advice. but a nanometer of detuning costs about 0.005 in transformation fidelity, and when you push that through to a measured S-value it is worth roughly 0.01. you have to get out to about 4 nm before the leftover rotation alone is enough to drag S through the classical bound. one ITU channel away, 0.8 nm, is essentially free.

so the wavelength argument does not on its own force the pilot into the same channel as the photons. and that matters, because being in the same channel is exactly what forces the time multiplexing, and the time multiplexing is what costs 7.2% of the link.

what actually forces it is power. the injector is putting 0.5 mW into the fiber. spontaneous Raman scattering off that pump spreads a noise floor across a wide swath of spectrum around it, and 0.5 mW against a detector counting single photons is not a contest. i went through the arithmetic of this in [the coexistence post]({% post_url 2026-03-23-coexistence-running-a-quantum-channel-and-its-clock-down-the-same-fiber %}); the summary is that you can run a classical signal alongside a quantum channel in the same fiber, but only after you have thought hard about launch power and spectral distance, and half a milliwatt one channel over is nowhere near that regime.

which reframes the problem. **the time multiplexing is a power problem wearing a wavelength problem's clothes.** the fix is not a faster switch, it is a dimmer pilot. and that work exists: [continuous stabilization from a dim coexisting reference](https://arxiv.org/abs/2411.15135) does the measurement with heterodyne detection instead of a polarimeter, which buys back enough sensitivity to run the reference at a power the quantum channel can tolerate. put that on this link and the 3 second window and the 55 second timeout both stop existing as concepts.

one honest counterargument, which is the reason we did not do that here. the fidelity curve in Fig. 3(b) is a snapshot. wavelength-dependent birefringence is itself a property of the fiber that drifts, so the offset between what the pilot experiences and what the photons experience is not a fixed number you can calibrate away once. it wanders too, on the same aerial fiber, for the same reasons. at zero detuning that entire class of error is identically zero, and on a link this unstable that is worth paying for. how much it actually wanders on aerial fiber is, as far as i know, not measured anywhere, and somebody should measure it.

## the duty cycle

so: pilot and photons share a channel, therefore they cannot share the fiber at the same time, therefore a pair of optical switches alternates the link between compensating and distributing.

that gives the link a duty cycle, and the duty cycle is the headline number. over 24 hours the compensation sessions consumed 7.2% of the link, leaving 92.8% uptime for entanglement.

<div class="qw" id="qw-duty" data-qw><div class="qw-hd"><span class="qw-t">what the link is actually doing all day</span><span class="qw-s">the 3 s quantum window against the compensation session. the paper reports 92.8% uptime; everything else here is backed out of that number and the 55 s timeout.</span><span class="qw-lg"><i style="background:#32c29e"></i>entanglement distribution<i style="background:#6ba3f0"></i>time tagging<i style="background:#f2a03d"></i>compensating</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">conditions</span><span class="qw-b" data-grp="dp"><button type="button" data-v="paper" class="on" aria-pressed="true">the paper&rsquo;s 24 h average</button><button type="button" data-v="night">a quiet night</button><button type="button" data-v="bad">a bad afternoon</button><button type="button" data-v="storm">every session times out</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">quantum window <b id="qwu-wv">3.0 s</b></span><input type="range" id="qwu-w" min="0.5" max="15" step="0.1" value="3"></label><label class="qw-sr"><span class="qw-l">a normal compensation session <b id="qwu-cv">100 ms</b></span><input type="range" id="qwu-c" min="20" max="2000" step="10" value="100"></label><label class="qw-sr"><span class="qw-l">sessions that hit the 55 s timeout <b id="qwu-pv">0.24%</b></span><input type="range" id="qwu-p" min="0" max="300" step="1" value="24"></label></div><div class="qw-pane"><span class="qw-pl">one minute of link time</span><svg class="qw-svg" viewBox="0 0 640 74" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Timeline of alternating compensation and entanglement distribution periods"><g id="qwu-tl"></g></svg></div><div class="qw-pane"><span class="qw-pl">where the day goes</span><svg class="qw-svg" viewBox="0 0 640 54" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Breakdown of 24 hours of link time"><g id="qwu-bar"></g></svg></div><div class="qw-out"><div class="qw-oi"><span class="qw-ok">link uptime</span><span class="qw-ov" id="qwu-up">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">measurement duty cycle</span><span class="qw-ov" id="qwu-md">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">effective pair rate</span><span class="qw-ov" id="qwu-pr">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">timeouts per day</span><span class="qw-ov" id="qwu-to">&mdash;</span></div><div class="qw-oi qw-grow"><span class="qw-ok">downtime caused by those timeouts</span><span class="qw-ov qw-txt" id="qwu-sh">&mdash;</span></div></div><p class="qw-n" id="qwu-note"></p><noscript><p class="qw-n">With a 3 s window and 92.8% uptime, the average compensation session works out to about 233 ms. If normal sessions take 100 ms, then about 0.24% of them, roughly 65 events in 24 hours, are hitting the 55 s timeout, and those 65 events account for more than half of the day&rsquo;s total downtime.</p></noscript></div>

the most useful thing about that duty cycle is not in the paper, and it took me a while after publication to see it. start from what is published. the window is 3 seconds and the uptime is 92.8%. so

```
    3 / (3 + t_avg)  =  0.928     →     t_avg ≈ 233 ms
```

the average compensation session is a shade under a quarter of a second. that is a fine number. it is also completely unrepresentative, because Fig. 5(b) shows the distribution: most sessions sit on a floor around 100 ms, and a handful run to tens of seconds, with one at 41 s.

so take 100 ms as the normal case and ask what fraction of sessions have to be timing out at 55 s to drag the mean up to 233 ms. the answer is about a quarter of one percent. over 24 hours the link runs roughly 26,700 cycles, so that is about **64 events in a day**, and those 64 events account for **around 57% of all the downtime**.

that changes what the fix looks like. the paper's outlook section proposes two things: faster hardware, meaning lithium niobate polarization controllers instead of the current ones, and a different algorithm, specifically a reverse-operator scheme that computes the inverse transformation from a fixed number of measurements instead of descending toward it.

the first one improves the average, and the average is not the problem. cut the ordinary session by a factor of ten, from 100 ms to 10 ms, and uptime goes from 92.8% to 95.5%. worth having. but every bit of the remaining 4.5% is then timeouts, and you have spent a hardware upgrade to get there. go the other way, leave the controller alone and stop the sessions timing out, and you get 96.8%. do both and you get 99.7%.

the second one attacks the tail, and the tail is the entire problem. a gradient descent on a target that is moving faster than the loop can chase it does not converge slowly, it fails to converge, and then it times out. a scheme with a fixed measurement count cannot fail that way. it has a worst case equal to its best case. that is the property that matters here, and it is worth saying more loudly than we said it.

there is a preprint from a few weeks ago claiming exactly this direction, [exponential speedup of polarization stabilization for long distance DWDM quantum networks](https://arxiv.org/abs/2609.02841). i have not read it carefully enough to summarize honestly, but the title is aimed at the right thing.

one more number while we are here. the 92.8% is link uptime, and inside each 3 second window the time tagger only runs for 2 seconds, because you have to stay clear of the next session. so the *measurement* duty cycle is 2 / 3.233, or about 62%. if you were quoting a service level to somebody, 62% is the honest figure and 92.8% is the one about the fiber.

## what came out

with the stabilization running:

- roughly 1500 entangled pairs per second detected across the link, in a 1.6 ns coincidence window, with the polarization analyzers out of the path.
- coincidence-to-accidental ratio around 12.
- a 24 hour time-averaged CHSH parameter of **S = 2.34 ± 0.37**, against a classical bound of 2, with the violation holding for more than 20 of those hours.

and the control, which is the part i find more persuasive than the result: with the stabilization off, the reference fidelity collapsed within minutes and S fell below 2 after about three hours and never came back for the remaining thirteen.

there is one more number in there that i want to pull out, because it is the best evidence in the paper for something the paper does not quite say. throw away the data taken immediately after a timed-out session, and the 24 hour average becomes S = 2.39 ± 0.14. the mean barely moves, 2.34 to 2.39. **the error bar drops by a factor of 2.6.** the timeouts were not degrading the average result. they were producing the variance.

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>NIST analyzer basis</th><th>0&deg; (H)</th><th>45&deg; (D)</th><th>90&deg; (V)</th><th>135&deg; (A)</th></tr>
</thead>
<tbody>
<tr><td>Visibility, at the source</td><td>0.97</td><td>0.96</td><td>0.97</td><td>0.96</td></tr>
<tr><td>Visibility, after 62 km</td><td>0.87</td><td>0.78</td><td>0.76</td><td>0.79</td></tr>
<tr><td><i>S</i>, at the source</td><td colspan="4">2.69 &plusmn; 0.02</td></tr>
<tr><td><i>S</i>, after 62 km</td><td colspan="4">2.26 &plusmn; 0.04</td></tr>
</tbody>
</table>
</div>

the state went in at S = 2.69 and came out at 2.26. the interesting question is where the 0.43 went, and the answer is not the one you would expect from a paper about polarization.

## where the Bell violation actually leaked out

three things happen to the state between the source and the far analyzer, and they are worth separating because only one of them is about polarization.

**accidentals.** a coincidence window is a bucket. real pairs land in it, and so does anything uncorrelated that happens to arrive at the right moment. the bucket here is 1.6 ns wide, the CAR is about 12, and for a fringe sitting on a flat accidental background the visibility ceiling is

```
    V  =  CAR / (CAR + 2)  =  12 / 14  =  0.857
```

that alone takes 0.965 down to about 0.83, which is most of the loss.

**leftover polarization drift.** the compensator hands back a fiber at better than 99% fidelity and then the fiber starts moving again immediately, for the whole 3 seconds you are using it. that is a real effect and it is the one the experiment is nominally about. it is also, on this budget, the smaller term.

**the coincidence window itself.** that 1.6 ns is not set by anything quantum. the SNSPDs jitter by about 100 ps. chromatic dispersion over 62 km smears the idler by about 300 ps. those add to something like 320 ps. the remaining ~1.5 ns, which is to say nearly all of it, comes from the electrical-optical-electrical link: the NIST detector clicks get turned into laser pulses, sent to UMD down a parallel fiber, and turned back into electrical pulses, so that one time tagger can see both ends of the experiment.

<div class="qw" id="qw-chsh" data-qw><div class="qw-hd"><span class="qw-t">the coincidence window is a noise filter, and it is set by a classical link</span><span class="qw-s">calibrated to the paper: 1500 pairs/s and CAR 12 in a 1.6 ns window, local visibility 0.965, distributed S of 2.26. the accidental density stays fixed while the timing jitter moves.</span><span class="qw-lg"><i style="background:#32c29e"></i>coincidences kept<i style="background:#f2a03d"></i>accidentals<i style="background:#e2603f"></i>the classical bound</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">configurations</span><span class="qw-b" data-grp="cp"><button type="button" data-v="paper" class="on" aria-pressed="true">the link as built</button><button type="button" data-v="fix">fix the E-O-E jitter</button><button type="button" data-v="narrow">same link, narrower window</button><button type="button" data-v="out">the 41 s outlier</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">coincidence window <b id="qwc-wv">1.60 ns</b></span><input type="range" id="qwc-w" min="0.05" max="5" step="0.01" value="1.6"></label><label class="qw-sr"><span class="qw-l">timing jitter, FWHM <b id="qwc-jv">1.60 ns</b></span><input type="range" id="qwc-j" min="0.2" max="3" step="0.01" value="1.6"></label><label class="qw-sr"><span class="qw-l">polarization error left in the window <b id="qwc-ev">18&deg;</b></span><input type="range" id="qwc-e" min="0" max="45" step="0.5" value="18"></label></div><div class="qw-two"><div class="qw-pane"><span class="qw-pl">the coincidence peak and the window on it</span><svg class="qw-svg" viewBox="0 0 320 170" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Coincidence histogram with the acceptance window drawn on it"><g id="qwc-hist"></g></svg></div><div class="qw-pane"><span class="qw-pl">S vs window width</span><svg class="qw-svg" viewBox="0 0 320 170" preserveAspectRatio="xMidYMid meet" role="img" aria-label="CHSH S-value as a function of coincidence window width"><g id="qwc-curve"></g></svg></div></div><div class="qw-out"><div class="qw-oi"><span class="qw-ok">coincidences</span><span class="qw-ov" id="qwc-c">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">accidentals</span><span class="qw-ov" id="qwc-a">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">CAR</span><span class="qw-ov" id="qwc-car">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">visibility</span><span class="qw-ov" id="qwc-vis">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">S</span><span class="qw-ov" id="qwc-s">&mdash;</span></div><div class="qw-oi qw-grow"><span class="qw-ok">verdict</span><span class="qw-ov qw-txt" id="qwc-v">&mdash;</span></div></div><p class="qw-n" id="qwc-note"></p><noscript><p class="qw-n">Holding the accidental rate fixed and shrinking the timing jitter from 1.6 ns to the 320 ps that the detectors and dispersion alone would give lets the window shrink by the same factor, which cuts accidentals fivefold at no cost in real coincidences. CAR goes from about 12 to about 60 and the modelled S rises from 2.26 to roughly 2.55, without touching any quantum hardware.</p></noscript></div>

which leads to the sentence i would have argued for putting in the abstract: **the largest single lever on this link's Bell violation is a classical timing problem.**

you can see the shape of it by closing the window on the link as it is. narrower window, fewer accidentals, better S, and fewer pairs per second, because you are now cutting into the peak. that is a trade, and 1.6 ns is roughly where we stopped trading.

narrowing the *peak* is the same move without the bill. shrink the jitter from 1.6 ns to the ~320 ps the detectors and the dispersion would give you on their own, and the window shrinks with it while staying matched to the peak. real coincidences are untouched. accidentals fall by a factor of five, because they scale with window width and nothing else. CAR goes from 12 to about 60, and S goes from 2.26 to something around 2.55.

that is a bigger improvement than anything the polarization stabilization could deliver, and it comes from replacing an E-O-E timing hack with a real synchronization scheme. which is, satisfyingly, the exact subject of [the coexistence post]({% post_url 2026-03-23-coexistence-running-a-quantum-channel-and-its-clock-down-the-same-fiber %}) and of the [DC-QNet clock characterization work](https://pubs.aip.org/aip/apl/article/125/16/164004/3316979/Clock-synchronization-characterization-of-the) done down the hall. the pieces are in the same building. they were not in the same experiment.

in fairness to the E-O-E link: the two fibers are in the same bundle from the same TX/RX pair, and the relative delay between them moved only about 110 ps over the full 24 hours. as a *stability* solution it is excellent. it is the absolute jitter, not the drift, that costs.

## the 41 second outlier

there is a datapoint in Fig. 5(a) that sits visibly off its fringe, and Fig. 5(b) tells you why: the compensation session immediately before it ran for 41 seconds. excluding it lifts that basis from V = 0.76 to 0.82 and the whole measurement from S = 2.26 to 2.30.

we reported it both ways, which is the right thing to do, and the paper explains it as a temporary volatile change in the fiber environment. from inside, the honest version is: something happened to that cable and we do not know what. wind, a truck, someone's boot on a strand-mounted splice case. the same story explains the timeout clusters in Fig. 6(b), which fall in the middle of the day and around the evening rush and not at three in the morning.

this is the same story as the ±0.37 error bar above, at the scale of a single datapoint. the general form of it, and the reason it is worth more than an anecdote, is that a 41 second compensation session is not just 41 seconds of lost link. it is 41 seconds of the fiber moving *while the controller is failing to catch it*, and when the session finally times out it hands back a channel that is still moving. so the next 3 second window is bad too. the damage outlives the outage. that is why timeouts show up in the S-value trace and not only in the uptime column, and it is another argument for an algorithm that cannot fail to converge.

## what i'd push on next

**1. dim the pilot and stop time multiplexing.** the polarization penalty for moving one ITU channel away is about 0.01 in S. the barrier is the 0.5 mW launch, not the 0.8 nm. heterodyne detection of a dim reference is already demonstrated. putting it on this link deletes the duty cycle, the 55 s timeout and the 3 s window in one move, and turns a threshold machine into a continuous servo.

**2. measure how much the wavelength offset itself drifts.** this is the honest objection to point 1 and nobody has the number. Fig. 3(b) is one snapshot on one afternoon. if the pilot-to-photon transformation offset on aerial fiber is stable to a few tenths of a percent over hours, point 1 is free. if it wanders, point 1 needs a slow outer calibration loop and somebody should design it.

**3. fix the clock, not the controller.** 1.6 ns of coincidence window is buying five times more accidentals than the physics requires. an actual synchronization scheme in place of the E-O-E link is worth roughly 0.3 in S, which is more than doubling the source brightness would give you.

**4. report the tail, not the mean.** 7.2% average downtime is the number that makes the abstract, and the 0.37 error bar next to the headline S is what the tail did to it. the distribution behind it is 64 pathological events and 26,000 uneventful ones. any operator running a quantum link is going to care about the 64, because that is what a service level agreement gets written against. the field should be reporting compensation-time percentiles the way we report latency percentiles, and i would like p99 and p99.9 to become normal in these papers.

**5. put an admission-control decision on top of it.** this connects to the thing i keep saying about [flex-grid allocation]({% post_url 2026-04-04-Flex-grid-for-entanglement %}). if a controller can tell, from the reference fidelity trace alone, that the fiber has entered a regime where sessions are going to time out, the right response is not to keep trying. it is to tell the application that this link is unavailable for the next few minutes and let it fail over or wait. right now the link has exactly one behavior under stress, which is to keep grinding, and the cost of that shows up as bad data rather than as an honest outage.

**6. someone should do this on a worse link.** 70% aerial and 18 dB is a hard link. it is not the hardest one in anybody's metro plant. the interesting experiment is the one that finds where this approach actually breaks, and we did not find that here, because it did not break.

## the short version

- polarization entanglement is the easy kind to make and the fragile kind to move. buried fiber holds its polarization transformation for hours. **aerial fiber holds it for seconds.**
- the link is 62 km NIST to UMD, about 70% strung between poles, 18 dB end to end, of which 5.6 dB is connectors and splices rather than glass.
- unstabilized, the transmitted state falls below 95% fidelity in under 20 seconds during the day, and S drops through the classical bound after about 3 hours and stays there.
- the fix is a bright polarization reference in the same channel as the photons, six reference states, gradient descent on the worst of their fidelities, 98% to trigger and 99% to stop.
- same channel means no coexistence, which means **time multiplexing**: 3 s of entanglement, then a check, forever.
- result: about 1500 pairs/s, CAR 12, **S = 2.34 ± 0.37 averaged over 24 hours**, violation held for more than 20 of them, 92.8% uptime.
- drop the data taken right after a timed-out session and it becomes **S = 2.39 ± 0.14**. the mean barely moves and the error bar shrinks by 2.6x. the timeouts were making the variance, not the average.
- the honest uptime for *measurement* is 2 s in every 3.23, about 62%. the 92.8% is a fact about the fiber, not about the service.
- back out the numbers and the average compensation session is 233 ms, of which the typical case is ~100 ms. **about 64 timeout events in a day cause roughly 57% of the downtime.** the mean is not the problem, the tail is. ten times faster hardware buys 95.5% uptime; not timing out buys 96.8%; both together buy 99.7%.
- so faster hardware buys almost nothing, and an algorithm that cannot fail to converge buys everything. the paper says both; only the second one matters.
- **the biggest available gain is not quantum.** the 1.6 ns coincidence window is set by jitter in a classical timing link, not by the detectors or the dispersion. fixing it is worth about 0.3 in S, more than any plausible improvement to the stabilizer.
- the pilot could probably sit an ITU channel away for a penalty of about 0.01 in S. what keeps it in the same channel is 0.5 mW of launch power, and that is a solvable problem with an existing solution.

what i took away from a year of this is not really a quantum lesson. most of the hard part was ordinary network engineering in an unusual costume: a duty cycle, a tail latency problem, and a clock distribution hack that turned out to set the error budget. the entangled photons mostly did what they were supposed to. it was the cable in the wind and the timing link we bolted onto it that decided how the numbers came out, and i suspect that will keep being true for a while.

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
  function erf(x){
    var s=x<0?-1:1; x=Math.abs(x);
    var t=1/(1+0.3275911*x);
    var y=1-(((((1.061405429*t-1.453152027)*t)+1.421413741)*t-0.284496736)*t+0.254829592)*t*Math.exp(-x*x);
    return s*y;
  }
  function fmtT(s){
    if(s>=3600) return (s/3600).toFixed(1)+' h';
    if(s>=90) return (s/60).toFixed(1)+' min';
    if(s>=10) return s.toFixed(0)+' s';
    if(s>=1) return s.toFixed(1)+' s';
    return (s*1000).toFixed(0)+' ms';
  }
  var GRID='#2f2f2f', AXT='#8f8f8f', GRN='#32c29e', ORG='#f2a03d', BLU='#6ba3f0', RED='#e2603f';

  /* ---------- 1. how fast the fiber runs away ---------- */
  (function(){
    var H=document.getElementById('qw-drift'); if(!H) return;
    var RATE={night:0.0887, evening:0.45, day:2.35, gust:9.0};
    var LBL={night:'2 a.m., still air', evening:'evening', day:'midday', gust:'a wind gust'};
    var cond='day';
    var st=H.querySelector('#qwr-t'), sh=H.querySelector('#qwr-h');
    var gG=H.querySelector('#qwr-grid'), gC=H.querySelector('#qwr-curve'), gM=H.querySelector('#qwr-mark'), gS=H.querySelector('#qwr-sph');
    var X0=42,X1=330,Y0=18,Y1=134,TM=60;
    function ang(t){ return Math.min(90, RATE[cond]*Math.pow(Math.max(t,0),0.8)); }
    function fid(t){ var a=ang(t)*Math.PI/360; return Math.cos(a)*Math.cos(a); }
    function tOf(f){
      var a=2*Math.acos(Math.sqrt(f))*180/Math.PI;
      return Math.pow(a/RATE[cond],1.25);
    }
    function cx(t){ return X0+(t/TM)*(X1-X0); }
    function cy(f){ return Y1-((f-0.5)/0.5)*(Y1-Y0); }
    function grid(h){
      clear(gG);
      gG.appendChild(el('rect',{x:cx(0),y:Y0,width:cx(3)-cx(0),height:Y1-Y0,fill:BLU,opacity:.13}));
      [0.5,0.6,0.7,0.8,0.9,1].forEach(function(f){
        gG.appendChild(el('line',{x1:X0,y1:cy(f),x2:X1,y2:cy(f),stroke:GRID,'stroke-width':1}));
        gG.appendChild(el('text',{x:X0-6,y:cy(f)+3.5,fill:AXT,'font-size':9,'text-anchor':'end'},f.toFixed(1)));
      });
      [0,15,30,45,60].forEach(function(t){
        gG.appendChild(el('text',{x:cx(t),y:Y1+13,fill:AXT,'font-size':9,'text-anchor':'middle'},t+'s'));
      });
      gG.appendChild(el('line',{x1:X0,y1:cy(h),x2:X1,y2:cy(h),stroke:ORG,'stroke-width':1.4,'stroke-dasharray':'4 3'}));
      gG.appendChild(el('text',{x:cx(3)+4,y:Y1-4,fill:BLU,'font-size':9},'3 s window'));
    }
    function curve(t){
      clear(gC); clear(gM);
      var d='';
      for(var i=0;i<=180;i++){ var tt=TM*i/180; d+=(i?'L':'M')+cx(tt).toFixed(1)+' '+cy(fid(tt)).toFixed(1); }
      gC.appendChild(el('path',{d:d,fill:'none',stroke:GRN,'stroke-width':2.2,'stroke-linecap':'round'}));
      gM.appendChild(el('circle',{cx:cx(t),cy:cy(fid(t)),r:4.5,fill:'#fff',stroke:GRN,'stroke-width':2}));
    }
    function sphere(t){
      clear(gS);
      var CXx=170, CYy=76, R=58, a=ang(t);
      gS.appendChild(el('circle',{cx:CXx,cy:CYy,r:R,fill:'none',stroke:GRID,'stroke-width':1.2}));
      gS.appendChild(el('ellipse',{cx:CXx,cy:CYy,rx:R,ry:R*0.34,fill:'none',stroke:GRID,'stroke-width':1}));
      gS.appendChild(el('line',{x1:CXx-R,y1:CYy,x2:CXx+R,y2:CYy,stroke:GRID,'stroke-width':1}));
      [30,60,90].forEach(function(v){
        var r=R*v/90;
        gS.appendChild(el('circle',{cx:CXx,cy:CYy,r:r,fill:'none',stroke:GRID,'stroke-width':.8,'stroke-dasharray':'2 3'}));
        gS.appendChild(el('text',{x:CXx+r+2,y:CYy-3,fill:AXT,'font-size':8},v+'°'));
      });
      var d='M'+CXx+' '+CYy, N=90;
      for(var i=1;i<=N;i++){
        var f=i/N, aa=ang(t*f), ph=2.4*Math.pow(f,0.55)+0.7;
        d+='L'+(CXx+R*(aa/90)*Math.cos(ph)).toFixed(1)+' '+(CYy+R*(aa/90)*Math.sin(ph)*0.8).toFixed(1);
      }
      gS.appendChild(el('path',{d:d,fill:'none',stroke:GRN,'stroke-width':1.8,opacity:.85}));
      gS.appendChild(el('circle',{cx:CXx,cy:CYy,r:3.4,fill:'#fff'}));
      var ph2=2.4*Math.pow(1,0.55)+0.7;
      gS.appendChild(el('circle',{cx:CXx+R*(a/90)*Math.cos(ph2),cy:CYy+R*(a/90)*Math.sin(ph2)*0.8,r:4.5,fill:GRN,stroke:'#fff','stroke-width':1.5}));
      gS.appendChild(el('text',{x:CXx-R,y:CYy+R+16,fill:AXT,'font-size':9},'start'));
    }
    function upd(){
      var t=+st.value, h=+sh.value/100;
      H.querySelector('#qwr-tv').textContent=t.toFixed(1)+' s';
      H.querySelector('#qwr-hv').textContent=(h*100).toFixed(1)+'%';
      grid(h); curve(t); sphere(t);
      var f=fid(t), a=ang(t), tx=tOf(h);
      H.querySelector('#qwr-f').textContent=(f*100).toFixed(1)+'%';
      H.querySelector('#qwr-a').textContent=a.toFixed(1)+'°';
      H.querySelector('#qwr-x').textContent=fmtT(tx);
      var vd,vc;
      if(f>=h){ vd='still inside the threshold'; vc=GRN; }
      else if(f>=0.95){ vd='the APC would already be firing'; vc=ORG; }
      else if(f>=0.85){ vd='entanglement is visibly degrading'; vc=ORG; }
      else { vd='no usable polarization state left'; vc=RED; }
      var ve=H.querySelector('#qwr-v'); ve.textContent=vd; ve.style.color=vc;
      H.querySelector('#qwr-note').innerHTML=
        'At '+LBL[cond]+', the state crosses the <b>'+(h*100).toFixed(1)+'%</b> trigger threshold about <b>'+fmtT(tx)+
        '</b> after the compensator lets go, and 95% at <b>'+fmtT(tOf(0.95))+'</b>. '+
        (tx<3.5 ? 'That is inside the 3 s window, so the fiber is already out of spec before the window closes and the entangled state is decohering while you measure it.'
                : 'That is outside the 3 s window, so a single compensation buys you the whole window and then some.');
    }
    btnGroup(H,'cond',function(v){ cond=v; upd(); });
    st.addEventListener('input',upd); sh.addEventListener('input',upd);
    upd();
  })();

  /* ---------- 2. how far the pilot can sit ---------- */
  (function(){
    var H=document.getElementById('qw-pilot'); if(!H) return;
    var A=0.0051, V0=0.7992, RT2=2*Math.SQRT2;
    var sd=H.querySelector('#qwp-d');
    var gG=H.querySelector('#qwp-grid'), gC=H.querySelector('#qwp-curves'), gM=H.querySelector('#qwp-mark');
    var X0=52,X1=592,Y0=16,Y1=182,DM=5;
    function fidOf(d){ return Math.max(0.5, 1-A*d*d); }
    function angOf(d){ return 2*Math.acos(Math.sqrt(Math.min(1,fidOf(d))))*180/Math.PI; }
    function sOf(d){
      var th=angOf(d)*Math.PI/180, g=(1+2*Math.cos(th))/3;
      return RT2*V0*g;
    }
    function cx(d){ return X0+(d/DM)*(X1-X0); }
    function cyF(f){ return Y1-((f-0.85)/0.15)*(Y1-Y0); }
    function cyS(s){ return Y1-((s-1.6)/0.8)*(Y1-Y0); }
    function grid(){
      clear(gG);
      [0.85,0.90,0.95,1.00].forEach(function(f){
        gG.appendChild(el('line',{x1:X0,y1:cyF(f),x2:X1,y2:cyF(f),stroke:GRID,'stroke-width':1}));
        gG.appendChild(el('text',{x:X0-7,y:cyF(f)+3.5,fill:GRN,'font-size':9,'text-anchor':'end'},f.toFixed(2)));
      });
      [1.6,1.8,2.0,2.2,2.4].forEach(function(s){
        gG.appendChild(el('text',{x:X1+7,y:cyS(s)+3.5,fill:'#c8c8c8','font-size':9},s.toFixed(1)));
      });
      [0,1,2,3,4,5].forEach(function(d){
        gG.appendChild(el('text',{x:cx(d),y:Y1+14,fill:AXT,'font-size':9,'text-anchor':'middle'},d+' nm'));
      });
      gG.appendChild(el('line',{x1:X0,y1:cyS(2),x2:X1,y2:cyS(2),stroke:RED,'stroke-width':1.4,'stroke-dasharray':'5 4'}));
      gG.appendChild(el('text',{x:X0+6,y:cyS(2)-5,fill:RED,'font-size':9.5},'S = 2, the classical bound'));
      gG.appendChild(el('rect',{x:cx(0),y:Y0,width:cx(1)-cx(0),height:Y1-Y0,fill:GRN,opacity:.09}));
      gG.appendChild(el('text',{x:cx(1)+5,y:Y1-6,fill:AXT,'font-size':9},'the paper’s 1 nm rule'));
      gG.appendChild(el('text',{x:X0-7,y:Y0-4,fill:GRN,'font-size':9,'text-anchor':'end'},'fidelity'));
      gG.appendChild(el('text',{x:X1+7,y:Y0-4,fill:'#c8c8c8','font-size':9},'S'));
    }
    function curves(){
      clear(gC);
      var df='',ds='';
      for(var i=0;i<=200;i++){
        var d=DM*i/200;
        df+=(i?'L':'M')+cx(d).toFixed(1)+' '+cyF(fidOf(d)).toFixed(1);
        ds+=(i?'L':'M')+cx(d).toFixed(1)+' '+cyS(sOf(d)).toFixed(1);
      }
      gC.appendChild(el('path',{d:df,fill:'none',stroke:GRN,'stroke-width':2.2}));
      gC.appendChild(el('path',{d:ds,fill:'none',stroke:'#c8c8c8','stroke-width':2.2,'stroke-dasharray':'6 4'}));
    }
    function upd(){
      var d=+sd.value;
      H.querySelector('#qwp-dv').textContent=d.toFixed(2)+' nm';
      grid(); curves(); clear(gM);
      gM.appendChild(el('line',{x1:cx(d),y1:Y0,x2:cx(d),y2:Y1,stroke:'#fff','stroke-width':1,opacity:.35}));
      gM.appendChild(el('circle',{cx:cx(d),cy:cyF(fidOf(d)),r:4.5,fill:'#fff',stroke:GRN,'stroke-width':2}));
      gM.appendChild(el('circle',{cx:cx(d),cy:cyS(sOf(d)),r:4.5,fill:'#fff',stroke:'#c8c8c8','stroke-width':2}));
      var f=fidOf(d), a=angOf(d), s=sOf(d), v=s/RT2;
      H.querySelector('#qwp-f').textContent=(f*100).toFixed(2)+'%';
      H.querySelector('#qwp-e').textContent=a.toFixed(1)+'°';
      H.querySelector('#qwp-vis').textContent=v.toFixed(3);
      H.querySelector('#qwp-s').textContent=s.toFixed(3);
      var vd,vc;
      if(d<0.05){ vd='what the paper did, zero wavelength error'; vc=GRN; }
      else if(s>=2.2){ vd='the detuning is costing you nothing'; vc=GRN; }
      else if(s>=2.05){ vd='measurable, still a violation'; vc=ORG; }
      else if(s>=2){ vd='barely still a violation'; vc=ORG; }
      else { vd='the pilot is no longer describing the same fiber'; vc=RED; }
      var ve=H.querySelector('#qwp-v'); ve.textContent=vd; ve.style.color=vc;
      var dS=(sOf(0)-s);
      H.querySelector('#qwp-note').innerHTML= d<0.05
        ? 'At zero detuning the pilot and the photons see exactly the same fiber, so this whole error term vanishes. Drag the slider and watch how slowly it comes back: <b>one ITU channel away costs about 0.01 of an S-value</b>, which is well inside the error bar on the paper’s 24 h average.'
        : 'A detuning of <b>'+d.toFixed(2)+' nm</b> leaves a residual rotation of about <b>'+a.toFixed(1)+'°</b> that the compensator cannot see, and costs <b>'+dS.toFixed(3)+'</b> in S. '
          +(d<=1.2 ? 'That is inside the paper’s own rule of thumb and is not what is keeping the pilot in the same channel. The 0.5 mW launch power is.'
                   : 'Out here the wavelength really is the problem, which is what the paper’s 1 nm rule is protecting against.');
    }
    btnGroup(H,'pl',function(v){ sd.value=v; upd(); });
    sd.addEventListener('input',function(){ offGroup(H,'pl'); upd(); });
    upd();
  })();

  /* ---------- 3. the duty cycle ---------- */
  (function(){
    var H=document.getElementById('qw-duty'); if(!H) return;
    var TO=55, RATE=1500;
    var PRE={paper:[3,100,24], night:[3,60,2], bad:[3,140,90], storm:[3,2000,300]};
    var sw=H.querySelector('#qwu-w'), sc=H.querySelector('#qwu-c'), sp=H.querySelector('#qwu-p');
    var gT=H.querySelector('#qwu-tl'), gB=H.querySelector('#qwu-bar');
    function timeline(W,mean,tag){
      clear(gT);
      var X0=4,X1=636,Y=10,Hh=30, SPAN=60, sc2=(X1-X0)/SPAN, t=0, guard=Math.max(0,W-tag);
      gT.appendChild(el('rect',{x:X0,y:Y,width:X1-X0,height:Hh,fill:'#212121',rx:3}));
      var n=0;
      while(t<SPAN && n<400){
        n++;
        var w=Math.min(W,SPAN-t);
        gT.appendChild(el('rect',{x:X0+t*sc2,y:Y,width:Math.max(0.6,w*sc2),height:Hh,fill:GRN,opacity:.55}));
        var tw=Math.min(tag,Math.max(0,SPAN-t));
        if(tw>0) gT.appendChild(el('rect',{x:X0+t*sc2,y:Y+Hh-10,width:Math.max(0.6,tw*sc2),height:10,fill:BLU}));
        t+=W; if(t>=SPAN) break;
        var cw=Math.min(mean,SPAN-t);
        gT.appendChild(el('rect',{x:X0+t*sc2,y:Y,width:Math.max(0.7,cw*sc2),height:Hh,fill:ORG}));
        t+=mean;
      }
      [0,15,30,45,60].forEach(function(s){
        gT.appendChild(el('text',{x:X0+s*sc2,y:Y+Hh+16,fill:AXT,'font-size':9,'text-anchor':s===0?'start':(s===60?'end':'middle')},s+' s'));
      });
      if(guard>0.02) gT.appendChild(el('text',{x:X1,y:Y-2,fill:AXT,'font-size':9,'text-anchor':'end'},'blue = tagger running, '+tag.toFixed(1)+' s of every '+W.toFixed(1)));
    }
    function bar(fUp,fNorm,fTo){
      clear(gB);
      var X0=4,X1=636,Y=6,Hh=26,Wd=X1-X0;
      var segs=[[fUp,GRN,'distributing'],[fNorm,ORG,'normal sessions'],[fTo,RED,'timed-out sessions']];
      var x=X0;
      segs.forEach(function(s){
        var w=s[0]*Wd;
        if(w>0.4) gB.appendChild(el('rect',{x:x,y:Y,width:w,height:Hh,fill:s[1],opacity:s[1]===GRN?.55:.9}));
        if(w>76) gB.appendChild(el('text',{x:x+w/2,y:Y+Hh/2+3.5,fill:s[1]===GRN?'#dff5ee':'#1a1a1a','font-size':10,'text-anchor':'middle','font-weight':600},(s[0]*100).toFixed(1)+'% '+s[2]));
        x+=w;
      });
      gB.appendChild(el('text',{x:X0,y:Y+Hh+15,fill:AXT,'font-size':9},'24 h of link time'));
    }
    function upd(){
      var W=+sw.value, c=+sc.value/1000, p=+sp.value/10000;
      var mean=(1-p)*c+p*TO;
      var cyc=W+mean, up=W/cyc, tag=Math.max(0,W-1), md=tag/cyc;
      var cycles=86400/cyc, tos=p*cycles, toTime=tos*TO, normTime=86400*(1-up)-toTime;
      var share=(1-up)>0 ? toTime/(86400*(1-up)) : 0;
      H.querySelector('#qwu-wv').textContent=W.toFixed(1)+' s';
      H.querySelector('#qwu-cv').textContent=(c*1000).toFixed(0)+' ms';
      H.querySelector('#qwu-pv').textContent=(p*100).toFixed(2)+'%';
      timeline(W,Math.min(mean,20),tag);
      bar(up, Math.max(0,normTime)/86400, Math.max(0,toTime)/86400);
      H.querySelector('#qwu-up').textContent=(up*100).toFixed(1)+'%';
      H.querySelector('#qwu-md').textContent=(md*100).toFixed(1)+'%';
      H.querySelector('#qwu-pr').textContent=Math.round(RATE*up).toLocaleString()+'/s';
      H.querySelector('#qwu-to').textContent=tos<1?tos.toFixed(1):Math.round(tos).toLocaleString();
      var se=H.querySelector('#qwu-sh');
      se.textContent=(share*100).toFixed(0)+'% of it, from '+(tos<1?tos.toFixed(1):Math.round(tos))+' events';
      se.style.color = share>0.5?RED:(share>0.2?ORG:GRN);
      H.querySelector('#qwu-note').innerHTML=
        'Mean session <b>'+fmtT(mean)+'</b> against a <b>'+W.toFixed(1)+' s</b> window gives <b>'+(up*100).toFixed(1)+'%</b> uptime. '
        +'The tagger only runs for '+tag.toFixed(1)+' s of each window, so the measurement duty cycle is <b>'+(md*100).toFixed(1)+'%</b>. '
        +(share>0.4
          ? 'Most of the downtime is coming from <b>'+Math.round(tos)+' timeout events</b> out of '+Math.round(cycles).toLocaleString()+' cycles. Making the ordinary session faster barely moves this; not timing out is the whole game.'
          : 'Timeouts are not driving this configuration, so the ordinary session length is what to attack.');
    }
    btnGroup(H,'dp',function(v){ sw.value=PRE[v][0]; sc.value=PRE[v][1]; sp.value=PRE[v][2]; upd(); });
    [sw,sc,sp].forEach(function(s){ s.addEventListener('input',function(){ offGroup(H,'dp'); upd(); }); });
    upd();
  })();

  /* ---------- 4. the coincidence window ---------- */
  (function(){
    var H=document.getElementById('qw-chsh'); if(!H) return;
    var CTOT=1972, ADEN=78.125, VLOC=0.965, RT2=2*Math.SQRT2;
    var PRE={paper:[1.6,1.6,18], fix:[0.32,0.32,18], narrow:[0.6,1.6,18], out:[1.6,1.6,29]};
    var sw=H.querySelector('#qwc-w'), sj=H.querySelector('#qwc-j'), se=H.querySelector('#qwc-e');
    var gHi=H.querySelector('#qwc-hist'), gCu=H.querySelector('#qwc-curve');
    function frac(w,j){ var sg=j/2.3548; return erf(w/(2*Math.SQRT2*sg)); }
    function model(w,j,e){
      var C=CTOT*frac(w,j), A=ADEN*w, car=A>0?C/A:0;
      var th=e*Math.PI/180, g=(1+2*Math.cos(th))/3;
      var V=VLOC*(car/(car+2))*g;
      return {C:C,A:A,car:car,V:V,S:RT2*V};
    }
    function hist(w,j){
      clear(gHi);
      var X0=34,X1=308,Y0=14,Y1=132, TM=3;
      var sg=j/2.3548, pk=CTOT/(sg*Math.sqrt(2*Math.PI)), top=(pk+ADEN)*1.06;
      function hx(t){ return X0+((t+TM)/(2*TM))*(X1-X0); }
      function hy(v){ return Y1-(v/top)*(Y1-Y0); }
      gHi.appendChild(el('rect',{x:hx(-w/2),y:Y0,width:hx(w/2)-hx(-w/2),height:Y1-Y0,fill:GRN,opacity:.15}));
      gHi.appendChild(el('line',{x1:X0,y1:hy(ADEN),x2:X1,y2:hy(ADEN),stroke:ORG,'stroke-width':1.6,'stroke-dasharray':'4 3'}));
      gHi.appendChild(el('text',{x:X1,y:hy(ADEN)-4,fill:ORG,'font-size':9,'text-anchor':'end'},'accidental floor'));
      var d='';
      for(var i=0;i<=160;i++){
        var t=-TM+2*TM*i/160, v=pk*Math.exp(-t*t/(2*sg*sg))+ADEN;
        d+=(i?'L':'M')+hx(t).toFixed(1)+' '+hy(Math.min(v,top)).toFixed(1);
      }
      gHi.appendChild(el('path',{d:d,fill:'none',stroke:GRN,'stroke-width':2.2}));
      [-2,-1,0,1,2].forEach(function(t){
        gHi.appendChild(el('text',{x:hx(t),y:Y1+13,fill:AXT,'font-size':9,'text-anchor':'middle'},t+' ns'));
      });
      gHi.appendChild(el('line',{x1:X0,y1:Y1,x2:X1,y2:Y1,stroke:GRID,'stroke-width':1}));
      gHi.appendChild(el('text',{x:hx(0),y:Y0+10,fill:'#cfcfcf','font-size':9,'text-anchor':'middle'},'window '+w.toFixed(2)+' ns on a '+j.toFixed(2)+' ns peak'));
    }
    function curve(w,j,e){
      clear(gCu);
      var X0=34,X1=308,Y0=14,Y1=132, WM=3;
      function qx(v){ return X0+(v/WM)*(X1-X0); }
      function qy(s){ return Y1-((s-1.8)/1.0)*(Y1-Y0); }
      [1.8,2.0,2.2,2.4,2.6,2.8].forEach(function(s){
        gCu.appendChild(el('line',{x1:X0,y1:qy(s),x2:X1,y2:qy(s),stroke:GRID,'stroke-width':1}));
        gCu.appendChild(el('text',{x:X0-6,y:qy(s)+3.5,fill:AXT,'font-size':9,'text-anchor':'end'},s.toFixed(1)));
      });
      [0,1,2,3].forEach(function(v){
        gCu.appendChild(el('text',{x:qx(v),y:Y1+13,fill:AXT,'font-size':9,'text-anchor':'middle'},v+' ns'));
      });
      gCu.appendChild(el('line',{x1:X0,y1:qy(2),x2:X1,y2:qy(2),stroke:RED,'stroke-width':1.4,'stroke-dasharray':'5 4'}));
      var d='';
      for(var i=1;i<=180;i++){
        var v=WM*i/180, s=Math.max(1.8,Math.min(2.8,model(v,j,e).S));
        d+=(i>1?'L':'M')+qx(v).toFixed(1)+' '+qy(s).toFixed(1);
      }
      gCu.appendChild(el('path',{d:d,fill:'none',stroke:GRN,'stroke-width':2.2}));
      var cur=model(w,j,e);
      if(w<=WM) gCu.appendChild(el('circle',{cx:qx(w),cy:qy(Math.max(1.8,Math.min(2.8,cur.S))),r:4.5,fill:'#fff',stroke:GRN,'stroke-width':2}));
    }
    function upd(){
      var w=+sw.value, j=+sj.value, e=+se.value, m=model(w,j,e);
      H.querySelector('#qwc-wv').textContent=w.toFixed(2)+' ns';
      H.querySelector('#qwc-jv').textContent=j.toFixed(2)+' ns';
      H.querySelector('#qwc-ev').textContent=e.toFixed(0)+'°';
      hist(w,j); curve(w,j,e);
      H.querySelector('#qwc-c').textContent=Math.round(m.C).toLocaleString()+'/s';
      H.querySelector('#qwc-a').textContent=Math.round(m.A).toLocaleString()+'/s';
      H.querySelector('#qwc-car').textContent=m.car.toFixed(1);
      H.querySelector('#qwc-vis').textContent=m.V.toFixed(3);
      H.querySelector('#qwc-s').textContent=m.S.toFixed(2);
      var vd,vc;
      if(m.S>=2.5){ vd='a violation you could build a protocol on'; vc=GRN; }
      else if(m.S>=2.25){ vd='about where the paper landed'; vc=GRN; }
      else if(m.S>=2.05){ vd='a violation, with nothing to spare'; vc=ORG; }
      else if(m.S>=2){ vd='inside the error bar of no result at all'; vc=ORG; }
      else { vd='no violation'; vc=RED; }
      var ve=H.querySelector('#qwc-v'); ve.textContent=vd; ve.style.color=vc;
      H.querySelector('#qwc-note').innerHTML=
        'A <b>'+w.toFixed(2)+' ns</b> window on a <b>'+j.toFixed(2)+' ns</b> peak keeps '+(100*frac(w,j)).toFixed(0)+'% of the real coincidences and '
        +'lets in <b>'+Math.round(m.A)+'</b> accidentals per second, for a CAR of <b>'+m.car.toFixed(1)+'</b> and S = <b>'+m.S.toFixed(2)+'</b>. '
        +'Closing the window on a fixed peak always improves S, and always costs count rate: at '+w.toFixed(2)+' ns you are keeping <b>'+Math.round(m.C).toLocaleString()+'</b> pairs per second. '
        +(j>1 ? 'Narrowing the peak instead is the free version of the same move. <b>The peak width is set by the classical timing link, not by the detectors</b>, so the jitter slider is the one that buys S without paying in rate.'
              : 'With the jitter this low the window closes down for free: same coincidences, a fifth of the accidentals.');
    }
    btnGroup(H,'cp',function(v){ sw.value=PRE[v][0]; sj.value=PRE[v][1]; se.value=PRE[v][2]; upd(); });
    [sw,sj,se].forEach(function(s){ s.addEventListener('input',function(){ offGroup(H,'cp'); upd(); }); });
    upd();
  })();
})();
</script>{% endraw %}