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

**BS saturates.** noise generated near the far end has to travel all the way back, and gets attenuated doing it, so it contributes almost nothing. past roughly 10 km, adding fiber stops adding backscatter noise. the curve flattens at `P_in · β_BS / (α_s + α_n)`.

**FS peaks and then falls.** forward-scattered noise has to survive the rest of the fiber too, and so does the signal generating it. past about 20 km the FS noise at the far receiver actually goes *down* with length.

which is a better result than you'd expect: **a longer link is not automatically a noisier link.** the noise floor stops climbing while the signal keeps falling, so the problem at long distance is that you're losing photons, not that you're gaining noise.

with one caveat that turns out to matter. that's all at fixed *launch* power. in a real link you set the launch power to hit a required *received* power, so as the fiber gets longer you turn the transmitter up, and then the BS noise doesn't saturate at all. it tracks whatever you had to do to keep the classical receiver alive. this is exactly why receiver sensitivity ends up setting the reach limit at the end of this post.

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