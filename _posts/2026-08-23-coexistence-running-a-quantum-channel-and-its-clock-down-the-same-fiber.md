---
layout: post
section-type: post
title: "Coexistence : running a quantum channel and its clock down the same fiber"
category: 'networking'
tags: [ 'networking', 'fiber', 'quantum' ]
---

every network has a clock problem. two boxes far apart need to agree on what time it is, and the further apart they are, the harder that gets. we solved this decades ago for classical networks. NTP for most things, PTP when you need sub-microsecond, White Rabbit when you need picoseconds.

quantum networks have the same problem, except worse in a way that isn't obvious until you look at the numbers. the two nodes have to agree on *when* to within picoseconds, and the signal they're comparing is one photon at a time. the clock signal that gets them there is a normal laser, launching something like 10<sup>14</sup> photons per second into the same glass.

so you have two choices. run the clock on its own fiber, or put it in the same fiber as the quantum channel and hope the quantum detector can't see it.

a group at NIST and the University of Maryland actually built this and measured it. the paper is [Burenkov et al., "Synchronization and coexistence in quantum networks," *Optics Express* 31(7), 11431 (2023)](https://doi.org/10.1364/OE.480486). the short answer is that coexistence works, the wavelength you pick for the clock matters more than anything else, and the whole thing runs out of road at about 100 km with parts you can buy today.

this post is the long answer. the first section is background for people who don't do quantum. if you already know what a coincidence window is, skip to *the brightness problem*.

## the quantum part, for people who don't do quantum

### what's actually in the fiber

a quantum network moves quantum states between nodes. the two things people actually want to do with it right now:

**quantum key distribution (QKD).** two parties end up holding the same random bit string, and the physics guarantees nobody copied it in transit. the signal is usually single photons, or weak laser pulses attenuated until they contain much less than one photon on average.

**entanglement distribution.** you generate a pair of entangled photons, send one to each end, and now the two nodes share a correlated resource they can spend on other protocols. this is the building block for entanglement swapping, quantum repeaters, and eventually a network that isn't just point to point.

either way, the payload is single photons. that constraint drives everything else.

### you cannot amplify it

in a classical link, loss is annoying but solvable. put an EDFA in every 80 km and keep going. an amplifier works by copying your signal into a stronger version of itself.

you can't do that to a quantum state. the no-cloning theorem says an unknown quantum state cannot be copied, and an amplifier is a copier. so there are no repeaters in a deployed quantum network today. every dB of loss is a dB you never get back, and the link budget is the whole game.

at 1550 nm, SMF-28 gives you about 0.17 to 0.2 dB/km. over 100 km that's 17 to 20 dB, so you keep roughly 1 to 2% of the photons you sent. that is why 100 km keeps showing up as the practical ceiling for repeaterless quantum links. hold onto that number, it comes back later.

### why the clock matters so much

this is the part that catches networking people out. in a classical link, timing errors cost you margin. in a quantum link, timing *is* the measurement.

most quantum protocols work by looking for **coincidences**: photon detected at Alice at time t, photon detected at Bob at time t, therefore these two came from the same pair. you accept a detection as a coincidence if it lands inside a **coincidence window** around the expected arrival time.

that window is where the two clocks show up. if your nodes agree on time to within 1 ns, your window has to be at least 1 ns wide, because otherwise you throw away real events. and a 1 ns window is a 1 ns bucket that any background photon can also fall into. tighten the clock, tighten the window, catch less garbage. the coincidence window is a noise filter whose width is set by your synchronization quality.

then there's the harder case. **Hong-Ou-Mandel (HOM) interference** is a two-photon effect where two identical photons hitting opposite sides of a beamsplitter always leave together, never one out of each port. it is the mechanism behind entanglement swapping, and therefore behind any future repeater.

"identical" is doing a lot of work in that sentence. the photons have to be indistinguishable in every degree of freedom, including arrival time. if one shows up meaningfully later than the other, you can in principle tell them apart, and the interference degrades. the paper works this out analytically: for two Gaussian photon pulses of width `σ` and a clock jitter `δt`, the indistinguishability is

```
    I  =  1 / sqrt( 1 + δt² / 2σ² )
```

which says something useful. **your clock jitter only needs to be small compared to the photon duration.** for the ~10 ps photons they consider, 5 ps of jitter leaves you at I = 0.94 and the paper treats anything under 10 ps as not significantly affecting indistinguishability. 40 ps drops you to 0.33, and the same photons are now telling on each other.

so: picoseconds, not because picoseconds are impressive, but because the photons are picoseconds long.

<div class="qw" id="qw-hom" data-qw><div class="qw-hd"><span class="qw-t">how much clock jitter a HOM measurement tolerates</span><span class="qw-s">I = 1 / &radic;(1 + &delta;t&sup2; / 2&sigma;&sup2;). only the ratio of jitter to photon duration matters.</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">measured in the paper</span><span class="qw-b" data-grp="ps"><button type="button" data-v="2.18">White Rabbit, 2.18 ps</button><button type="button" data-v="3.5">pulsed laser, 3.5 ps</button><button type="button" data-v="0.1">PL averaged, 100 fs</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">clock jitter <span class="qw-gk">&delta;t</span> <b id="qwh-dtv">10.0 ps</b></span><input type="range" id="qwh-dt" min="0" max="60" step="0.1" value="10"></label><label class="qw-sr"><span class="qw-l">photon duration <span class="qw-gk">&sigma;</span> <b id="qwh-sgv">10.0 ps</b></span><input type="range" id="qwh-sg" min="2" max="25" step="0.5" value="10"></label></div><div class="qw-two"><div class="qw-pane"><span class="qw-pl">the two photons arriving</span><svg class="qw-svg" viewBox="0 0 320 180" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Two photon wavepackets offset in time"><g id="qwh-pulse"></g></svg></div><div class="qw-pane"><span class="qw-pl">indistinguishability vs jitter</span><svg class="qw-svg" viewBox="0 0 320 180" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Indistinguishability versus clock jitter"><g id="qwh-curve"></g></svg></div></div><div class="qw-out"><div class="qw-oi"><span class="qw-ok">indistinguishability</span><span class="qw-ov" id="qwh-I">0.816</span></div><div class="qw-oi qw-grow"><span class="qw-ok">verdict</span><span class="qw-ov qw-txt" id="qwh-v">&mdash;</span></div></div><p class="qw-n" id="qwh-note"></p><noscript><p class="qw-n">For the ~10 ps photons the paper considers, 2.18 ps of jitter (White Rabbit) gives I = 0.988, 5 ps gives 0.943, 10 ps gives 0.816 and 40 ps gives 0.333. Because only the ratio &delta;t/&sigma; enters, longer photons tolerate proportionally more jitter.</p></noscript></div>

### how you tell if it worked

one number recurs in this paper: **g<sup>(2)</sup>(0)**, the second order coherence at zero delay. treat it as a purity score for the light.

- g<sup>(2)</sup>(0) = 1 is ordinary laser light.
- g<sup>(2)</sup>(0) = 0 is a perfect single photon source, meaning you never get two photons at once.
- **below 0.5 is the threshold for "this is non-classical"**, the boundary you have to stay under for a HOM measurement to prove anything.

background photons drive that number up, because they add accidental double detections. this is how the paper turns "how much noise is too much" into a hard line you can plot.

## the brightness problem

now the actual engineering.

the classical channel and the quantum channel are not close in power. they aren't in the same universe of power. the paper puts the gap at **7 to 12 orders of magnitude** at the receiver.

a rough version of the arithmetic: a normal transceiver receiver runs happily at -30 dBm, which is 1 µW, which at 1310 nm is around 6.6 × 10<sup>12</sup> photons per second. a single photon detector is doing well to see 10<sup>3</sup> to 10<sup>5</sup> counts per second, and a decent chunk of that is dark counts and stray light. you are asking one piece of glass to carry both, and asking one detector to notice the quiet one.

the obvious move is a separate fiber for the clock. sometimes you can do that. often you can't, because dark fiber between two specific buildings is either unavailable or priced like it's made of something other than sand. and every extra fiber in a quantum network is another thing to route, splice, and pay for. so people want coexistence, and the question is what it costs.

### the noise mechanism

if you put a bright signal at 1310 nm into a fiber, and you look at 1550 nm with a good enough detector, you will see photons. the fiber makes them.

the mechanism is **inelastic scattering**, mostly spontaneous Raman. an incoming photon interacts with a vibrational mode of the silica, gives up (or takes) a bit of energy, and continues at a different wavelength. the wavelength shift is set by the glass, not by you, and in silica it spreads energy across a very wide band. tens of THz wide. that's why a 1310 nm laser pollutes 1550 nm even though those wavelengths are 240 nm apart and sitting in completely different CWDM channels.

this is not something a filter at the receiver can fix. the noise photons are *generated inside the fiber*, along the whole length of it, downstream of every filter you put at the transmitter. they arrive at your quantum detector looking exactly like quantum channel photons, because they are photons in the quantum channel.

two flavors, and the distinction matters:

- **back-scattered (BS)**, counter-propagating. generated along the fiber, travelling back toward the end the classical signal came from.
- **forward-scattered (FS)**, co-propagating. travelling the same direction as the classical signal, into the far receiver.

if your sync protocol is bidirectional, which White Rabbit is, you get both at both ends.

### how the noise scales with length

the paper builds a small model for this and it produces two behaviors worth memorizing, at fixed launch power.

**BS saturates.** noise generated near the far end has to travel all the way back, and gets attenuated doing it, so it contributes almost nothing. the curve flattens out at `P_in · β_BS / (α_s + α_n)`, and it approaches that ceiling gradually: for a 1310 nm sync signal it is at 68% of the ceiling by 10 km and 90% by 20 km. past that, more fiber barely adds any backscatter.

**FS peaks and then falls.** forward-scattered noise has to survive the rest of the fiber too, and so does the signal generating it. past about 20 km the FS noise at the far receiver actually goes *down* with length.

which is a better result than you'd expect: **a longer link is not automatically a noisier link.** the noise floor stops climbing while the signal keeps falling, so the problem at long distance is that you're losing photons, not that you're gaining noise.

with one caveat that turns out to matter. that's all at fixed *launch* power. in a real link you set the launch power to hit a required *received* power, so as the fiber gets longer you turn the transmitter up, and then the BS noise doesn't saturate at all. it tracks whatever you had to do to keep the classical receiver alive. this is exactly why receiver sensitivity ends up setting the reach limit at the end of this post.

<div class="qw" id="qw-noise" data-qw><div class="qw-hd"><span class="qw-t">background noise in the 1550 nm quantum channel</span><span class="qw-s">the paper's own model, Eqs. 2, 3 and 6. hover or tap the chart to read values.</span><span class="qw-lg"><i style="background:#32c29e"></i>back-scattered (BS)<i style="background:#f2a03d"></i>forward-scattered (FS)</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">sync wavelength</span><span class="qw-b" data-grp="wl"><button type="button" data-v="1270">1270</button><button type="button" data-v="1310" class="on" aria-pressed="true">1310</button><button type="button" data-v="1330">1330</button><button type="button" data-v="1490">1490 nm</button></span></div><div class="qw-g"><span class="qw-l">hold constant</span><span class="qw-b" data-grp="md"><button type="button" data-v="in" class="on" aria-pressed="true">launch power</button><button type="button" data-v="out">received power</button></span></div></div><svg class="qw-svg" viewBox="0 0 640 330" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Background noise versus fiber length"><g id="qwn-grid"></g><g id="qwn-curves"></g><g id="qwn-cross"></g></svg><div class="qw-out"><div class="qw-oi"><span class="qw-ok">at</span><span class="qw-ov" id="qwn-L">25.0 km</span></div><div class="qw-oi"><span class="qw-ok qw-bs">back-scattered</span><span class="qw-ov" id="qwn-bs">&mdash;</span></div><div class="qw-oi"><span class="qw-ok qw-fs">forward-scattered</span><span class="qw-ov" id="qwn-fs">&mdash;</span></div></div><p class="qw-n" id="qwn-note"></p><noscript><p class="qw-n">At a fixed launch power of 10<sup>14</sup> photons/s into a 100 GHz channel at 1547.72 nm, back-scattered noise reaches 68% of its ceiling by 10 km and 90% by 20 km, while forward-scattered noise peaks between 18 and 24 km (depending on the sync wavelength) and falls after that. Holding the <em>received</em> power fixed instead removes the ceiling entirely. Switching the sync wavelength from 1490 nm to 1270 nm drops both curves by about 60x.</p></noscript></div>

## the testbed

two nodes, Alice and Bob, one shared spool of SMF-28. they measured at 1, 6, 12 and 25 km.

the quantum channel under test is a single **100 GHz DWDM channel at 1547.72 nm** (C37), read out with a superconducting nanowire single photon detector (SNSPD). the wider quantum band they care about is 1500 to 1620 nm, split into six 20 nm CWDM channels for the spectrometer measurements.

two synchronization schemes were compared:

**White Rabbit.** a pair of WR switches, one grandmaster and one boundary clock, running the high accuracy PTP profile standardized in IEEE 1588-2019. commercial transceivers, bidirectional, using wavelength pairs of 1310/1490 nm or 1270/1330 nm.

**a pulsed laser.** a picosecond diode laser at 1310 nm, under 100 ps pulses at 10 MHz, detected by a fast InGaAs detector. no protocol, just a metronome.

both referenced to a 10 MHz rubidium clock, with a time tagger contributing 1.5 ps RMS per channel.

### the filtering trick

one detail in the setup that a fiber person will appreciate more than a physicist. there are **two** pairs of CWDM modules in the design, not one.

the outer pair does the obvious job: multiplex the classical and quantum channels into the common fiber at each end, demultiplex at the other. standard WDM.

the inner pair sits *before the classical transmitters*, and its only job is to clean up the transmitters themselves. a commercial SFP laser is not spectrally clean. it has a broad, low-level pedestal of emission spreading well outside its nominal channel, and part of that pedestal lands directly in the quantum band. at classical power levels nobody has ever cared. against a single photon detector it's a spotlight.

so they filter the transmitter's own out-of-band garbage before it ever enters the shared fiber. the CWDM modules give more than 30 dB of suppression on adjacent channels and at least 40 dB on non-adjacent ones.

worth noting what this costs, since it's the kind of thing that gets left out of block diagrams. the total detection efficiency of the quantum path came out at **0.172**, and the two biggest losses in that chain are the DWDM at 0.40 and the CWDM at 0.73. the filtering that makes coexistence possible eats most of your photons on the way in.

## what they found

### wavelength choice does most of the work

they extracted a noise generation constant for each classical wavelength, which lets you compare them directly.

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Sync wavelength</th><th>Detuning from 1547.72 nm</th><th>Fiber attenuation</th><th>Relative noise into the quantum channel</th></tr>
</thead>
<tbody>
<tr><td>1490 nm</td><td>58 nm</td><td>0.19 dB/km</td><td>1x (worst)</td></tr>
<tr><td>1330 nm</td><td>218 nm</td><td>0.28 dB/km</td><td>~0.20x</td></tr>
<tr><td>1310 nm</td><td>238 nm</td><td>0.32 dB/km</td><td>~0.12x</td></tr>
<tr><td>1270 nm</td><td>278 nm</td><td>0.34 dB/km</td><td>~0.016x</td></tr>
</tbody>
</table>
</div>

moving your clock from 1490 nm to 1270 nm buys you about **60x less noise** in the quantum channel. the paper calls it almost two orders of magnitude, and it is by far the cheapest thing on the list, because both of those are stock CWDM wavelengths on stock SFPs.

the reason is the shape of the Raman gain spectrum in silica. it falls off as you detune further from the pump, so the further your classical carrier sits from the quantum band, the less of its energy scatters into it. **detune the clock below 1300 nm and stay there.**

the cost is real but small: 1270 nm attenuates at 0.34 dB/km against 0.19 for 1490 nm. you are spending link budget on the classical channel, which has budget to spare, to buy quiet in the quantum channel, which has none.

### the 12 km spool

a detail I liked, because it's the sort of thing that happens on real installs and usually gets quietly dropped from papers.

the 12 km spool consistently returned higher loss and higher noise constants than the model predicted, and than the other three spools. rather than smooth it over, they went and looked. a separate investigation found a **defect in that fiber coupling light into the cladding**, which then generated extra background. they excluded the 12 km numbers from their averages and said why.

it's a good reminder that in this regime the fiber is a component with a datasheet and a failure mode, not an ideal pipe. and that a fiber defect too small to bother a classical link is a measurable noise source when your receiver counts individual photons.

### you can fit more than 100 quantum channels next to one clock

they measured the background across the full 1500 to 1620 nm window with a spectrometer, not just in the single DWDM channel. the noise is broad and fairly flat, so one sync channel raises the floor across the whole band by roughly the same amount.

that band divides into more than **100 channels of 100 GHz**. so a single classical synchronization channel can coexist with a hundred-odd quantum channels in one fiber. the clock is a fixed tax on the fiber rather than a per-channel tax, which is the difference between a lab demo and something you would actually deploy.

### both sync schemes are fast enough

**White Rabbit** measured 2.18 ps RMS jitter with single-pulse averaging. the honest caveat is in the paper: the time tagger's own intrinsic jitter over two channels is 2.12 ps RMS, so the measurement is essentially reading the instrument. WR is at least this good, and they can't say how much better from this setup. what they can see is that WR drifts on millisecond timescales, with the time interval error climbing back up to several ps when averaged over 10<sup>4</sup> to 10<sup>6</sup> pulses.

**the pulsed laser** is worse on a single shot, about 3.5 ps, limited by detector signal-to-noise. but that error is Gaussian and averages down cleanly, reaching **100 fs with 10<sup>4</sup> pulses averaged**, which at 10 MHz is 1 ms of averaging. no drift bump.

different tools. WR gives you a protocol, a network, and a number that's good enough immediately. a pulsed laser gives you a better number if you're willing to average and to build the rest of the system yourself.

practical reach note: they synchronized nodes over 18 km using commercial 40 km SFPs at 1310/1490 nm, and over 12 km using 10 km SFPs at 1330/1270 nm. the quiet wavelength pair reached less far because they had shorter-reach modules on it, not because of anything to do with the wavelengths.

### you will need to gate the detector

the ungated background rate in the quantum channel can exceed **10<sup>6</sup> counts per second**. no single photon detector wants to see that, and every one of those counts is a chance to fake a coincidence.

the fix is gating: only listen during the window when a real photon could arrive. the required window is `2σ + δt`, the photon duration plus the clock jitter, which for 10 ps photons is tens of picoseconds. that's far below what you can gate a detector at directly, so the paper suggests putting an **electro-optic modulator in front of the detector** and gating the light instead of the electronics.

this is where good synchronization pays for itself twice. it keeps your photons indistinguishable, and it lets you shrink the gate, and the gate is what keeps a million background counts per second from ever reaching the detector.

## the actual limit

the last piece pulls it together. they solve for where g<sup>(2)</sup>(0) crosses 0.5, the line where your HOM measurement stops proving anything non-classical, as a function of fiber length and classical receiver sensitivity.

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Classical receiver</th><th>Required optical power at Rx</th><th>Max coexisting fiber length</th></tr>
</thead>
<tbody>
<tr><td>Off-the-shelf transceiver + White Rabbit</td><td>1 µW (-30 dBm)</td><td>~100 km</td></tr>
<tr><td>Quantum receiver near the fundamental limit</td><td>~10<sup>5</sup> photons/s</td><td>~300 km</td></tr>
<tr><td>Any receiver, absolute wall</td><td>1 photon/s</td><td>~400 km</td></tr>
</tbody>
</table>
</div>

the logic connects back to the launch power point from earlier. a longer fiber means you have to launch more classical power to deliver the same 1 µW to the far receiver. more launch power means more Raman noise in the quantum channel along the whole span. eventually the noise wins.

so the lever is **receiver sensitivity**. a classical receiver that can work with less light lets you launch less light, which generates less noise. the paper estimates that a receiver operating near the Helstrom bound, roughly one photon per bit, would need about 8 orders of magnitude less optical power than an off-the-shelf module, and that buys you from 100 km to about 300 km. past 400 km nothing helps, because by then the classical signal arriving at the far end is under one photon per second and there's no protocol left to run.

now put that next to the number from the top of the post. **~100 km is already roughly the practical ceiling for a repeaterless quantum link** because of plain fiber attenuation. the coexistence limit and the loss limit land in the same place. not a total coincidence, since the same 0.2 dB/km drives both of them, but a convenient one.

so with today's equipment, sharing the fiber costs you nothing in reach. it saves you a fiber you would otherwise have had to lease.

## the short version

- quantum networks need picosecond clock sync, because the coincidence window is a noise filter and the photons are picoseconds long.
- the clock is a classical signal, 7 to 12 orders of magnitude brighter than the quantum one. sharing a fiber with it should be hopeless.
- it isn't, because the noise mechanism is Raman scattering in the glass, and Raman falls off with detuning.
- **the sync wavelength is the decision that matters most.** 1270 nm instead of 1490 nm is about 60x less noise into the C-band, and it costs you nothing but a slightly worse classical link budget.
- filter the classical transmitters *before* the shared fiber. cheap SFPs have a broad emission pedestal that lands in the quantum band.
- at fixed launch power the noise stops growing past ~20 km. at fixed received power it doesn't, which is why receiver sensitivity sets the reach.
- one sync channel coexists with more than 100 quantum DWDM channels in the same fiber.
- White Rabbit off the shelf gives ~2 ps, good enough. a pulsed laser plus 1 ms of averaging gives 100 fs.
- gate the detector optically, or a million background counts per second will find their way into your coincidence window.
- with parts you can buy, this works to about 100 km. which is about as far as a repeaterless quantum link goes anyway.

the thing I keep coming back to is how much of this is ordinary fiber engineering. detuning, filtering, launch power, receiver sensitivity, link budget. the quantum part sets the requirements, and then the requirements get met with WDM modules and a careful look at where every dB went.

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
.qw-l b{text-transform:none}
.qw-l b{color:#32c29e;font-weight:600;font-variant-numeric:tabular-nums}
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
.qw-pane{flex:1 1 260px;min-width:0}
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
  var HC = 6.62607015e-34 * 299792458;
  function ak(a){ return a * Math.LN10 / 10 / 1000; }         // dB/km -> 1/m
  var AN = ak(0.17), DNU = 1e11, PIN = 1e14, POUT_W = 1e-6;
  var W = {
    1270:{as:ak(0.34), bs:0.061e-23, fs:0.058e-23},
    1310:{as:ak(0.32), bs:0.449e-23, fs:0.421e-23},
    1330:{as:ak(0.28), bs:0.745e-23, fs:0.699e-23},
    1490:{as:ak(0.19), bs:3.75e-23,  fs:3.69e-23}
  };
  function noise(lam, L, mode){
    var w = W[lam], as = w.as, bs, fs, P;
    if (mode === 'in') {
      P = PIN;
      bs = (1 - Math.exp(-(as+AN)*L)) * w.bs * DNU * P / (as+AN);
      fs = (Math.exp(-AN*L) - Math.exp(-as*L)) * w.fs * DNU * P / (as-AN);
    } else {
      P = POUT_W / (HC / (lam*1e-9));
      bs = (Math.exp(as*L) - Math.exp(-AN*L)) * w.bs * DNU * P / (as+AN);
      fs = (Math.exp((as-AN)*L) - 1) * w.fs * DNU * P / (as-AN);
    }
    return [bs, fs];
  }
  // ---- shared helpers ----
  var NS = 'http://www.w3.org/2000/svg';
  function el(tag, attrs, txt){
    var e = document.createElementNS(NS, tag);
    for (var k in attrs) e.setAttribute(k, attrs[k]);
    if (txt != null) e.textContent = txt;
    return e;
  }
  function clear(g){ while (g.firstChild) g.removeChild(g.firstChild); }
  function sup(v){
    if (!(v > 0)) return '0';
    var e = Math.floor(Math.log10(v)), m = v / Math.pow(10, e);
    return m.toFixed(1) + ' × 10<sup>' + e + '</sup>';
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
    var wrap = root.querySelector('[data-grp="'+grp+'"]');
    if (!wrap) return;
    wrap.addEventListener('click', function(e){
      var b = e.target.closest('button'); if (!b) return;
      Array.prototype.forEach.call(wrap.children, function(x){
        x.classList.remove('on'); x.removeAttribute('aria-pressed');
      });
      b.classList.add('on'); b.setAttribute('aria-pressed','true');
      cb(b.getAttribute('data-v'));
    });
  }
  // ============ widget 1 : noise vs fiber length ============
  var N = document.getElementById('qw-noise');
  if (N) (function(){
    var GX=62, GY=14, GW=562, GH=252, LMAX=80, Y0=3, Y1=7;  // y decades, reset per mode
    var gGrid=N.querySelector('#qwn-grid'), gCur=N.querySelector('#qwn-curves'),
        gCross=N.querySelector('#qwn-cross'), svg=N.querySelector('.qw-svg');
    var st={lam:1310, md:'in', L:25000};
    function px(km){ return GX + (km/LMAX)*GW; }
    function py(v){
      var t = (Math.log(Math.max(v,1e-12))/Math.LN10 - Y0)/(Y1-Y0);
      return GY + (1 - Math.min(Math.max(t,0),1))*GH;
    }
    function grid(){
      clear(gGrid);
      for (var d=Y0; d<=Y1; d++){
        var y=py(Math.pow(10,d));
        gGrid.appendChild(el('line',{x1:GX,y1:y,x2:GX+GW,y2:y,stroke:'#2e2e2e','stroke-width':1}));
        var t=el('text',{x:GX-9,y:y+4,'text-anchor':'end',fill:'#7d7d7d','font-size':10});
        t.appendChild(el('tspan',{},'10'));
        t.appendChild(el('tspan',{dy:-4,'font-size':7.5}, String(d)));
        gGrid.appendChild(t);
      }
      for (var km=0; km<=LMAX; km+=20){
        var x=px(km);
        gGrid.appendChild(el('line',{x1:x,y1:GY,x2:x,y2:GY+GH,stroke:'#282828','stroke-width':1}));
        gGrid.appendChild(el('text',{x:x,y:GY+GH+18,'text-anchor':'middle',fill:'#7d7d7d','font-size':10}, km));
      }
      gGrid.appendChild(el('text',{x:GX+GW/2,y:GY+GH+34,'text-anchor':'middle',fill:'#8f8f8f','font-size':11},'fiber length, km'));
      var yl=el('text',{x:0,y:0,'text-anchor':'middle',fill:'#8f8f8f','font-size':11,
        transform:'translate(13,'+(GY+GH/2)+') rotate(-90)'},'noise, photons/s');
      gGrid.appendChild(yl);
    }
    function path(which){
      var d='', first=true;
      for (var km=0; km<=LMAX; km+=0.4){
        var v=noise(st.lam, km*1000, st.md)[which];
        if (!(v > Math.pow(10,Y0))) continue;   // start the curve at the axis floor
        d += (first?'M':'L') + px(km).toFixed(1) + ' ' + py(v).toFixed(1);
        first=false;
      }
      return d;
    }
    function draw(){
      clear(gCur);
      gCur.appendChild(el('path',{d:path(0),fill:'none',stroke:'#32c29e','stroke-width':2.2,
        'stroke-linejoin':'round','stroke-linecap':'round'}));
      gCur.appendChild(el('path',{d:path(1),fill:'none',stroke:'#f2a03d','stroke-width':2.2,
        'stroke-linejoin':'round','stroke-linecap':'round'}));
      cross();
    }
    function cross(){
      clear(gCross);
      var km=st.L/1000, x=px(km), v=noise(st.lam, st.L, st.md);
      gCross.appendChild(el('line',{x1:x,y1:GY,x2:x,y2:GY+GH,stroke:'#6a6a6a','stroke-width':1,'stroke-dasharray':'3 3'}));
      [[v[0],'#32c29e'],[v[1],'#f2a03d']].forEach(function(p){
        if (p[0]>0) {
          gCross.appendChild(el('circle',{cx:x,cy:py(p[0]),r:4.5,fill:p[1],stroke:'#191919','stroke-width':1.5}));
        }
      });
      N.querySelector('#qwn-L').textContent = km.toFixed(1) + ' km';
      N.querySelector('#qwn-bs').innerHTML = sup(v[0]);
      N.querySelector('#qwn-fs').innerHTML = sup(v[1]);
    }
    function note(){
      var w=W[st.lam], pk=Math.log(w.as/AN)/(w.as-AN)/1000;
      var sat20=100*(1-Math.exp(-(w.as+AN)*20000)), sat10=100*(1-Math.exp(-(w.as+AN)*10000));
      var n;
      if (st.md==='in'){
        n = 'At a fixed launch power of 10<sup>14</sup> photons/s, back-scattered noise runs into a ceiling: it is at <b>'
          + sat10.toFixed(0) + '%</b> of that ceiling by 10 km and <b>' + sat20.toFixed(0)
          + '%</b> by 20 km. Forward-scattered noise peaks at <b>' + pk.toFixed(1)
          + ' km</b> and falls after that, because past the peak the fiber attenuates the noise and the signal making it faster than new noise accumulates.';
      } else {
        n = 'Holding the <em>received</em> power at 1 &micro;W is what a real link does, and it removes the ceiling. '
          + 'To deliver the same 1 &micro;W over a longer fiber you have to launch more power, so both curves keep climbing. '
          + 'This is why receiver sensitivity, not the noise physics, sets the reach limit.';
      }
      N.querySelector('#qwn-note').innerHTML = n;
    }
    function all(){
      if (st.md==='in'){ Y0=3; Y1=7; } else { Y0=2; Y1=8; }
      grid(); draw(); note();
    }
    btnGroup(N,'wl',function(v){ st.lam=+v; all(); });
    btnGroup(N,'md',function(v){ st.md=v;  all(); });
    function pick(e){
      var r=svg.getBoundingClientRect(), cx=(e.touches?e.touches[0].clientX:e.clientX);
      var km=((cx-r.left)/r.width*640 - GX)/GW*LMAX;
      st.L = Math.min(Math.max(km,0),LMAX)*1000;
      cross();
    }
    var down=false;
    svg.addEventListener('pointerdown',function(e){down=true;pick(e);});
    svg.addEventListener('pointermove',function(e){ if(down||e.pointerType==='mouse') pick(e); });
    window.addEventListener('pointerup',function(){down=false;});
    svg.style.cursor='crosshair';
    all();
  })();
  // ============ widget 2 : HOM indistinguishability ============
  var H = document.getElementById('qw-hom');
  if (H) (function(){
    var dt=H.querySelector('#qwh-dt'), sg=H.querySelector('#qwh-sg');
    var gP=H.querySelector('#qwh-pulse'), gC=H.querySelector('#qwh-curve');
    function I(d,s){ return 1/Math.sqrt(1 + d*d/(2*s*s)); }
    function pulses(d,s){
      clear(gP);
      var X0=10, X1=310, Y0=14, Y1=140, T=100;   // +-100 ps
      var tx=function(t){ return X0+(t+T)/(2*T)*(X1-X0); };
      var ty=function(v){ return Y1-v*(Y1-Y0); };
      gP.appendChild(el('line',{x1:X0,y1:Y1,x2:X1,y2:Y1,stroke:'#3a3a3a','stroke-width':1}));
      [-100,-50,0,50,100].forEach(function(t){
        gP.appendChild(el('line',{x1:tx(t),y1:Y1,x2:tx(t),y2:Y1+4,stroke:'#3a3a3a','stroke-width':1}));
        gP.appendChild(el('text',{x:tx(t),y:Y1+16,'text-anchor':'middle',fill:'#7d7d7d','font-size':9},t));
      });
      gP.appendChild(el('text',{x:(X0+X1)/2,y:Y1+32,'text-anchor':'middle',fill:'#8f8f8f','font-size':10},'arrival time, ps'));
      [[-d/2,'#32c29e'],[d/2,'#f2a03d']].forEach(function(p){
        var dd='M'+tx(-T)+' '+Y1;
        for (var t=-T;t<=T;t+=1.5){
          dd+='L'+tx(t).toFixed(1)+' '+ty(Math.exp(-Math.pow(t-p[0],2)/(2*s*s))).toFixed(1);
        }
        dd+='L'+tx(T)+' '+Y1+'Z';
        gP.appendChild(el('path',{d:dd,fill:p[1],'fill-opacity':.22,stroke:p[1],'stroke-width':1.8}));
      });
      if (d>0.5){
        var yA=Y0-2;
        gP.appendChild(el('line',{x1:tx(-d/2),y1:yA,x2:tx(d/2),y2:yA,stroke:'#8f8f8f','stroke-width':1}));
        gP.appendChild(el('text',{x:(tx(-d/2)+tx(d/2))/2,y:yA-4,'text-anchor':'middle',fill:'#9a9a9a','font-size':9.5},'δt = '+d.toFixed(1)+' ps'));
      }
    }
    function curve(d,s){
      clear(gC);
      var X0=34, X1=310, Y0=14, Y1=140, DM=60;
      var cx=function(v){ return X0+v/DM*(X1-X0); };
      var cy=function(v){ return Y1-v*(Y1-Y0); };
      [0,0.5,1].forEach(function(v){
        gC.appendChild(el('line',{x1:X0,y1:cy(v),x2:X1,y2:cy(v),stroke:v===0.5?'#4a3a2a':'#2e2e2e','stroke-width':1,'stroke-dasharray':v===0.5?'4 3':''}));
        gC.appendChild(el('text',{x:X0-7,y:cy(v)+4,'text-anchor':'end',fill:'#7d7d7d','font-size':9},v.toFixed(1)));
      });
      [0,20,40,60].forEach(function(t){
        gC.appendChild(el('text',{x:cx(t),y:Y1+16,'text-anchor':'middle',fill:'#7d7d7d','font-size':9},t));
      });
      gC.appendChild(el('text',{x:(X0+X1)/2,y:Y1+32,'text-anchor':'middle',fill:'#8f8f8f','font-size':10},'clock jitter δt, ps'));
      gC.appendChild(el('text',{x:X0+4,y:cy(0.5)-5,fill:'#a8823f','font-size':9},'I = 0.5'));
      var dd='';
      for (var t=0;t<=DM;t+=0.5) dd+=(t?'L':'M')+cx(t).toFixed(1)+' '+cy(I(t,s)).toFixed(1);
      gC.appendChild(el('path',{d:dd,fill:'none',stroke:'#32c29e','stroke-width':2.2,'stroke-linecap':'round'}));
      if (d<=DM) gC.appendChild(el('circle',{cx:cx(d),cy:cy(I(d,s)),r:4.5,fill:'#fff',stroke:'#32c29e','stroke-width':2}));
    }
    function verdict(v){
      if (v>=0.98) return ['essentially perfect','#32c29e'];
      if (v>=0.90) return ['fine, barely costs you anything','#32c29e'];
      if (v>=0.75) return ['usable, starting to show','#d8c257'];
      if (v>=0.50) return ['measurably degraded','#f2a03d'];
      return ['the photons are telling on each other','#e2603f'];
    }
    function upd(){
      var d=+dt.value, s=+sg.value, v=I(d,s), vd=verdict(v);
      H.querySelector('#qwh-dtv').textContent = d.toFixed(1)+' ps';
      H.querySelector('#qwh-sgv').textContent = s.toFixed(1)+' ps';
      H.querySelector('#qwh-I').textContent = v.toFixed(3);
      var ve=H.querySelector('#qwh-v'); ve.textContent=vd[0]; ve.style.color=vd[1];
      pulses(d,s); curve(d,s);
      H.querySelector('#qwh-note').innerHTML =
        'Jitter of <b>'+d.toFixed(1)+' ps</b> against a <b>'+s.toFixed(1)+
        ' ps</b> photon gives a ratio &delta;t/&sigma; of <b>'+(d/s).toFixed(2)+
        '</b>. Slide the photon duration and watch the curve stretch: a longer photon tolerates proportionally more jitter, which is why the paper can call anything under 10 ps good enough for its ~10 ps pulses.';
    }
    function clearPresets(){
      Array.prototype.forEach.call(H.querySelectorAll('[data-grp="ps"] button'), function(b){
        b.classList.remove('on'); b.removeAttribute('aria-pressed');
      });
    }
    dt.addEventListener('input',function(){ clearPresets(); upd(); });
    sg.addEventListener('input',upd);
    btnGroup(H,'ps',function(v){ dt.value=v; upd(); });
    upd();
  })();
})();
</script>{% endraw %}