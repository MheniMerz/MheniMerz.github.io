---
layout: post
section-type: post
title: "Trans-Atlantic fiber : when the photon that arrives is not the one you sent"
category: 'networking'
tags: [ 'networking', 'fiber', 'quantum' ]
---

almost everything on this blog fits inside a commute. [the aerial link]({% post_url 2026-07-20-holding-entanglement-together-on-a-wire-in-the-wind %}) was 62 km, most of it hanging off telephone poles between Gaithersburg and College Park, and the longest experiments i have been part of are around 100 km. the cables that carry almost everything any of us send to Europe run 5,000 to 7,000 km, and they lie on the bottom of the Atlantic.

i realized i could not explain how that works. not the brochure version, the link budget version. my fiber intuition is metro-shaped: a launch power, a receiver sensitivity, some splices, some patch panels, a few dB of margin. the aerial link was 18 dB end to end, and that one number decided most of what the experiment could do.

so this one is a different kind of post. no paper and no disclosure. it started as a question about how a trans-Atlantic cable deals with loss, and it turned into a second, more interesting question. the repeaters on the seabed keep the light as light the whole way across. so could a qubit ride along?

it cannot, and the reason is a theorem rather than an engineering problem, and the same theorem is hiding inside the noise figure of every amplifier on the cable. the first half of this post is how a hundred amplifiers in a row are made to work. the second half is why that exact trick is the one thing a qubit cannot survive.

below: why a 1,000 dB link has no link budget, why the real ceiling is noise against the glass and not loss, what the receiver quietly took over, why watts on the seabed are scarcer than fiber, and then what an amplifier does to a photon, what would work instead, and who has tried the Atlantic so far.

## a link with no budget

MAREA, Virginia Beach to Bilbao, is about 6,600 km. modern ultra-low-loss submarine fiber loses about 0.15 to 0.16 dB/km, so

```
6,600 km  ×  0.16 dB/km  ≈  1,056 dB
```

there is no transmitter and no receiver that close that gap, and there never will be. so you do not budget the loss. **you chop it up.**

the cable is cut into spans of 50 to 90 km, with an optical amplifier at the end of each one. that is roughly a hundred repeaters on the seabed, most of them several kilometres down, each built to run untouched for 25 years. some cables also carry branching units in the middle of the ocean that peel off fiber pairs, or in newer designs individual wavelengths, toward other landings.

it was not always amplifiers. TAT-8 in 1988, the first trans-Atlantic fiber cable, used electronic regenerators: detect the 280 Mb/s signal, retime it, send a fresh one. regeneration deletes the noise completely, which sounds perfect. it also locks the cable to one bit rate and one wavelength for its entire life, and it puts complicated electronics on the ocean floor.

the erbium-doped fiber amplifier changed that, and TAT-12/13 in 1996 were the first trans-Atlantic systems built on it. an EDFA does not care about bit rate, modulation format, or how many wavelengths go through it, and that transparency is the whole architecture of a modern cable. the wet plant is deliberately dumb: glass, amplifiers, passive filters. all the intelligence lives in shore terminals that get replaced every few years. cables laid around 2000 for 10 Gb/s wavelengths were later relit with coherent 100G-class wavelengths, and nobody touched the seabed.

## every amplifier is heard

why 50 to 90 km and not 150? every amplifier adds amplified spontaneous emission, ASE, in proportion to its gain, and its gain has to equal the loss of the span in front of it. span loss is exponential in length. the number of amplifiers is only linear in how short you make the spans.

so halve the span from 100 km to 50 km. each amplifier's gain drops from 16 dB to 8 dB, about 6× less noise per amplifier. you need twice as many, and the net is roughly **5 dB of OSNR for free**. you keep shortening until the repeaters, and above all the electrical power to run them, stop paying for themselves.

and here is the part that does not exist at metro scale. every one of those amplifiers is heard at the far end. each repeater's gain exactly cancels the loss in front of it, so the ASE added by the first amplifier arrives at the receiver at the same level as the ASE added by the last. N amplifiers, N times the noise. the rule of thumb everybody uses:

```
OSNR (dB, 0.1 nm)  ≈  58  +  P_ch  −  L_span  −  NF  −  10·log10(N)
```

`P_ch` is the per-channel launch power in dBm, `L_span` the span loss, `NF` the amplifier noise figure. 0 dBm per channel, 10 dB spans, a 5 dB noise figure and 110 amplifiers gives about 23 dB.

two terms in that line are worth holding onto.

the last one is the price of distance. **every doubling of the cable costs 3 dB, forever**, and no component fixes it. it comes back in the quantum half with a much worse exponent.

and `NF`. it looks like a component spec, a number on a datasheet you would like to be smaller. hold that thought too.

## the glass fights back

in a metro network the binding constraint is usually the power budget or the transceiver. under the ocean, power is never the problem, because every amplifier restores it. the problem is SNR, and noise only accumulates.

so the obvious move is more launch power, since OSNR goes up dB for dB. this is where the glass pushes back. silica's refractive index depends very slightly on intensity, the Kerr effect, and at high power that produces three related distortions:

- **self-phase modulation**: each channel's own intensity modulates its phase.
- **cross-phase modulation**: every other channel's intensity modulates it too.
- **four-wave mixing**: channels beat together and create new frequency components.

in high-dispersion fiber carrying coherent signals, all three behave, to a good approximation, like extra Gaussian noise whose power grows with the cube of the launch power. that observation is [the GN model](https://arxiv.org/abs/1209.0394), and it is how people budget these links in practice. the received SNR looks like

```
SNR  ≈  P  /  ( P_ASE  +  η·P³ )
```

and the consequences fall straight out of it.

- **there is an optimum launch power**, exactly where the nonlinear noise equals half the ASE. at that point you sit 1.76 dB below what a perfectly linear fiber would give you at the same power.
- **noise figure has diminishing returns.** a 3 dB better amplifier buys about 2 dB of peak SNR, because the optimum power drops at the same time.
- **distance still costs 3 dB per doubling.**
- **less nonlinearity raises the ceiling.** that is the whole point of large-area submarine fiber. spreading the light over 125 µm² instead of G.652's 80 µm² drops the nonlinear coefficient from about 1.3 to 0.8 /W/km.

undoing the nonlinearity digitally works in principle, but a receiver can only undo what a channel did to itself, not what neighbours it never sees did to it. in practice the fiber and the power you choose set the ceiling.

play with it:

<div class="qw" id="qw-gn" data-qw><div class="qw-hd"><span class="qw-t">noise against the glass</span><span class="qw-s">received SNR against launch power, for a fully loaded C-band of 64 GBd channels. amplifier noise rewards more power, the Kerr effect punishes it, and the peak sits where the nonlinear noise is half the amplifier noise.</span><span class="qw-lg"><i style="background:#32c29e"></i>total<i style="background:#f2a03d"></i>amplifier noise only, a linear fiber<i style="background:#7f8fd6"></i>nonlinear noise only</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">jump to</span><span class="qw-b" data-grp="pre"><button type="button" data-v="marea" class="on" aria-pressed="true">MAREA-like</button><button type="button" data-v="pac">trans-Pacific</button><button type="button" data-v="metro">built like a metro link</button></span></div><div class="qw-g"><span class="qw-l">fiber</span><span class="qw-b" data-grp="fb"><button type="button" data-v="m">metro G.652, 80 &micro;m&sup2;</button><button type="button" data-v="u" class="on" aria-pressed="true">submarine, 125 &micro;m&sup2;</button><button type="button" data-v="w">submarine, 150 &micro;m&sup2;</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">span length <b id="qwg-spv"></b></span><input type="range" id="qwg-sp" min="40" max="120" step="5" value="60"></label><label class="qw-sr"><span class="qw-l">total distance <b id="qwg-dsv"></b></span><input type="range" id="qwg-ds" min="500" max="13000" step="100" value="6600"></label><label class="qw-sr"><span class="qw-l">amplifier noise figure <b id="qwg-nfv"></b></span><input type="range" id="qwg-nf" min="3.5" max="7" step="0.1" value="5"></label></div><svg class="qw-svg" viewBox="0 0 640 320" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Received SNR against launch power per channel: amplifier-noise-limited on the left, nonlinearity-limited on the right, with the optimum marked"><g id="qwg-plot"></g></svg><div class="qw-out"><div class="qw-oi"><span class="qw-ok">amplifiers</span><span class="qw-ov" id="qwg-N">-</span><span class="qw-os" id="qwg-Ns"></span></div><div class="qw-oi"><span class="qw-ok">span loss</span><span class="qw-ov" id="qwg-L">-</span><span class="qw-os">fiber loss only</span></div><div class="qw-oi"><span class="qw-ok qw-bs">optimum launch</span><span class="qw-ov" id="qwg-P">-</span><span class="qw-os" id="qwg-Ps"></span></div><div class="qw-oi"><span class="qw-ok qw-bs">peak SNR</span><span class="qw-ov" id="qwg-S">-</span><span class="qw-os" id="qwg-Ss"></span></div><div class="qw-oi"><span class="qw-ok">Shannon ceiling</span><span class="qw-ov" id="qwg-C">-</span><span class="qw-os">per fiber, theoretical</span></div></div><p class="qw-n" id="qwg-note"></p><noscript><p class="qw-n">With 60 km spans over 6,600 km, 5 dB amplifiers and 125 &micro;m&sup2; submarine fiber, the link has 110 amplifiers and its SNR peaks at 13.7 dB (about 20.8 dB OSNR in 0.1 nm) at a launch power of &minus;0.4 dBm per channel, a theoretical ceiling of about 41 Tb/s per fiber. Building the same link like a metro network, with 80 km spans of standard G.652 fiber, drops the peak to 9.2 dB and the ceiling to about 29 Tb/s. Halving the distance buys exactly 3 dB, and a 1.5 dB better amplifier buys only 1 dB.</p></noscript></div>

two things worth noticing while the sliders are moving.

**shorter spans lift the whole curve**, and the optimum moves to lower power. you are trading repeaters, and watts, for dB.

**"built like a metro link" is a cable designed on terrestrial assumptions**: 80 km spans of G.652. it gives up about 4.5 dB of peak SNR against the submarine design and close to 30% of the theoretical capacity, which is the entire case for submarine fiber in one click.

the model behind the widget is the incoherent GN model with fiber loss only, identical spans and a fully loaded C-band of 64 GBd channels, nothing fitted. the Shannon number it prints is a ceiling, not a spec. deployed cables quote roughly half of it once margins and implementation penalties are paid, and newer designs deliberately run each fiber below its optimum, for a reason that has to do with electricity and comes up further down.

## the receiver ate the dispersion map

chromatic dispersion used to shape the cable itself. in the 10G direct-detection era, submarine links were dispersion-mapped: long stretches of fiber with a small negative dispersion, and every few hundred kilometres a section of positive-dispersion fiber to pull the running total back toward zero. the terminals trimmed each channel separately to handle the slope.

zero dispersion was never an option once WDM arrived. channels that travel at the same speed stay in step long enough for four-wave mixing to build up. the map was a compromise: some local dispersion to break the mixing, near-zero accumulated dispersion so a direct-detection receiver could cope.

coherent detection ended all of that. once the receiver captures the full optical field, amplitude, phase and both polarizations, dispersion is just a known all-pass filter for the DSP to invert. new cables use uniform fiber at about 20 to 21 ps/nm·km with no inline compensation at all, which accumulates roughly 140,000 ps/nm across the Atlantic. a 64 GBd symbol arrives smeared across about 70 ns, some 4,500 symbol periods, and an FFT-based equalizer in the receiver chip undoes it continuously.

leaving dispersion uncompensated turned out to be better for nonlinearity as well. with high local dispersion, channels walk off each other quickly and their interactions average out. that is why old dispersion-managed cables get less out of coherent modems than new uncompensated ones.

polarization mode dispersion is benign. modern fiber runs around 0.05 ps/√km, so the mean DGD over 6,600 km is about 4 ps, a quarter of a symbol, and the receiver's adaptive 2×2 MIMO equalizer tracks it along with the constantly wandering state of polarization. the polarization effect that does cost margin is polarization-dependent loss. every isolator, coupler and filter in every repeater contributes a little, a hundred repeaters add it up, and designers reserve a fraction of a dB for SNR that fluctuates as the polarization drifts.

it is worth holding that next to the aerial link. there, daytime wind walked the polarization far enough to take fidelity below 95% in under 20 seconds. the deep seafloor is the opposite environment: dark, 2 to 4 °C, no sun, no wind. the polarization still drifts, slowly, and nobody notices, because a coherent receiver does not care which way the light is polarized. it only has to keep up.

the rest is coding. soft-decision FEC at 20 to 25% overhead turns a pre-FEC error rate of a few percent into effectively error-free output, and probabilistic constellation shaping lets the modem tune its bits per symbol almost continuously to whatever SNR the cable delivers. that is why trans-Atlantic capacity is quoted per fiber pair, on the order of 20 Tb/s for recent designs, and why that number keeps climbing on cables that are already in the water.

## a hundred amplifiers have to agree

errors that do not matter in one amplifier matter a great deal in a hundred. a gain shape that sits 0.1 dB higher at one end of the band is a 10 dB tilt at the far end. so every repeater carries a gain-flattening filter matched to its own EDFA, passive equalizers are inserted periodically to remove whatever shape still accumulates, and the terminal does the final trim with per-channel pre-emphasis. stimulated Raman scattering, which steadily drags power from short wavelengths to long ones, gets folded into the same tilt budget.

submarine repeaters also run differently from most terrestrial ones: deep in saturation, at essentially constant total output power. that makes the chain self-healing. if a span gets lossier, from aging or from a repair that adds cable, the next amplifier's input drops but its output barely moves, and within a few spans the power profile is back on track.

the catch is that a constant-power amplifier divides its output among whatever light shows up. on day one, with a handful of real wavelengths lit, each would get an enormous share and drown in nonlinearity. so the terminals fill the unused spectrum with shaped ASE or dummy carriers, and swap them out for revenue channels as capacity is sold. **a submarine line is always fully loaded**, whether or not anybody is paying for it.

## watts are scarcer than glass

the repeaters need electricity, and the only way to get it there is down the cable. around the fibers sits a copper conductor, and power feeding equipment at the landing stations drives a constant DC current of about an amp through the whole chain, at up to roughly 15 kV, with the ocean as the return path through sea-earth electrodes. usually one end feeds positive and the other negative, and either end can typically carry the whole cable alone. each repeater drops a few tens of volts to run its pumps, the conductor itself burns a large share of the total as heat, and the constant-current feed also absorbs the slow voltage difference between the two shores, which a geomagnetic storm can swing by hundreds of volts.

that electrical budget caps the pump power in every repeater, which caps the total optical power all the fiber pairs share. for years the answer was a few fiber pairs, each run near its nonlinear optimum. but capacity grows only logarithmically with SNR and linearly with the number of parallel paths. in a line limited by amplifier noise, take the power that gives one fiber 20 dB of SNR and split it across four: each falls to 14 dB, and together they carry about 2.8× the capacity. split it sixteen ways and it is about 7×.

that is space-division multiplexing, and it is why Dunant carries 12 fiber pairs and Grace Hopper 16, with pump lasers shared between pairs and each fiber run at lower power, often well below its optimum. so the design philosophy flipped, and **the best cables now run each fiber in a mostly linear regime at modest SNR, because watts on the seabed are scarcer than glass.**

keep that envelope in mind for later: tens of watts per repeater housing, off a constant-current line, for 25 years, with no visits.

## watching a cable you cannot reach

you cannot OTDR a repeatered cable the way you shoot a metro span, because every EDFA contains isolators that block the backscatter. instead each repeater has a high-loss loopback path that couples a tiny fraction of the light in one fiber into its partner heading back to shore. a probe launched from the terminal comes back from every repeater at a different delay, which gives a per-repeater gain measurement, and the same loopbacks let a coherent OTDR see Rayleigh backscatter span by span. tracked over months, that catches a weakening pump before the traffic notices.

when something breaks, it is usually in shallow water, from fishing gear and anchors. so the continental-shelf sections are armoured and buried, while in the deep ocean the cable is about the diameter of a garden hose and simply lies on the bottom. a repair means grappling the cable, cutting it, hauling both ends to the surface and splicing in new cable, which inserts two to three times the water depth in extra fiber. in 5 km of water that is over 10 km of glass and a couple of dB in one span, which is why the budgets carry explicit margin for repairs and for 25 years of aging.

two impairments only show up at this length. equalization-enhanced phase noise: the receiver's local oscillator is mixed in before that enormous digital dispersion compensation, which turns the laser's phase noise into extra noise in proportion to the accumulated dispersion, so long-haul transponders need narrow-linewidth lasers. and guided acoustic-wave Brillouin scattering: thermally driven vibrations of the fiber's cross-section add a phase and polarization noise floor that scales with signal power, so more launch power does not help. both are small, and both show up in trans-Pacific budgets.

that is the classical half. now the question that started the second one.

## does the repeater turn light into electricity?

no. TAT-8's regenerators did. no trans-Atlantic cable laid since 1996 has.

an EDFA is a coil of fiber doped with erbium, pumped by a laser, typically at 980 nm, that lifts the erbium ions into an excited state. when a 1550 nm signal photon passes an excited ion, it triggers stimulated emission: the ion gives up a second photon with the same wavelength, the same phase, the same polarization and the same direction. light is amplified as light. the electricity in the cable runs the pumps and their control, and nothing electrical ever touches the signal.

so the photons stay photons the whole way across, which sounds like exactly what a qubit wants. it is the opposite.

## amplification is copying

an amplifier that boosts an unknown field is a machine for copying it, and [an unknown quantum state cannot be copied](https://doi.org/10.1038/299802a0). nature enforces that with noise: [an amplifier that boosts the field regardless of its phase has to add noise](https://doi.org/10.1103/PhysRevD.26.1817), and at high gain the minimum is a noise figure of 3 dB. that added noise is spontaneous emission, the ASE from the top of the post.

so here is the thought i asked you to hold. **the `NF` in the OSNR formula is not really a component spec. it is the price quantum mechanics charges for amplification**, and real EDFAs sit only 1.5 to 2 dB above the floor, at 4.5 to 5 dB.

for a classical signal that price is affordable, because the signal is enormous by quantum standards:

```
photons per symbol  =  P / (h·ν · R_s)
                    =  1 mW / (1.28e-19 J  ×  64e9 /s)
                    ≈  1.2e5
```

even after a 10 dB span, about twelve thousand of those photons per symbol reach the next amplifier, which adds noise equivalent to a few photons per symbol at its input. a hundred amplifiers' worth of that still leaves an SNR of around 15 dB, plenty for coherent modulation, because the information is spread redundantly across a bright field.

a qubit is one photon, and the state of that one photon is the information. the amplifier's noise is now the same size as the signal. at high gain the best an amplifier can do is equivalent to measuring the photon and preparing new ones from the result. stimulated emission is in fact [the optimal universal cloner](https://arxiv.org/abs/quant-ph/9910048), and for a polarization qubit its copies approach a fidelity of 2/3, which is exactly what you get by measuring and guessing. any entanglement the photon carried is gone after the first amplifier, and a hundred more do nothing to bring it back.

phase-sensitive amplifiers can beat 3 dB, but only by amplifying one quadrature of the field and squeezing the other. that works for a classical signal with a known phase reference. it cannot preserve a state you do not know. **the limit lives in the theorem, so no amount of better hardware moves it.**

### the photon that arrives is not the one you sent

even the classical photons are not conserved. the chance that any particular photon survives 1,056 dB is 10<sup>−105.6</sup>.

run the bookkeeping on MAREA: 110 spans of 9.6 dB, each amplifier's gain matched to the span in front of it. the last amplifier's output is about nine times its input, so roughly 89% of what reaches the receiver was emitted in the last amplifier, 10% in the one before it, 1% in the one before that, and so on back to the transmitter, whose share is the same 10<sup>−105.6</sup>.

<div class="qw" id="qw-cp" data-qw><div class="qw-hd"><span class="qw-t">whose photons are these</span><span class="qw-s">where the photons reaching the receiver were actually emitted, for a chain of amplifiers whose gain matches the span loss in front of each one. same line, two different things sent down it.</span><span class="qw-lg"><i style="background:#32c29e"></i>emitted by an amplifier<i style="background:#f2a03d"></i>sent by the transmitter</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">what goes in</span><span class="qw-b" data-grp="md"><button type="button" data-v="cl" class="on" aria-pressed="true">a classical symbol, 1 mW at 64 GBd</button><button type="button" data-v="q">one qubit</button></span></div><div class="qw-g"><span class="qw-l">line</span><span class="qw-b" data-grp="ln"><button type="button" data-v="marea" class="on" aria-pressed="true">MAREA, 110 &times; 9.6 dB</button><button type="button" data-v="short">220 &times; 4.8 dB</button><button type="button" data-v="long">55 &times; 19.2 dB</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">span loss <b id="qwp-lv"></b></span><input type="range" id="qwp-l" min="3" max="20" step="0.1" value="9.6"></label><label class="qw-sr"><span class="qw-l">amplifiers <b id="qwp-nv"></b></span><input type="range" id="qwp-n" min="1" max="220" step="1" value="110"></label></div><div class="qw-pane"><span class="qw-pl">share of the photons at the receiver, by where they were emitted</span><svg class="qw-svg" viewBox="0 0 640 204" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Share of received photons emitted by each of the last amplifiers, by all earlier ones, and by the transmitter"><g id="qwp-bars"></g></svg></div><div class="qw-out"><div class="qw-oi"><span class="qw-ok">photons per symbol sent</span><span class="qw-ov" id="qwp-in">-</span></div><div class="qw-oi"><span class="qw-ok">reaching each amplifier</span><span class="qw-ov" id="qwp-arr">-</span><span class="qw-os">per symbol</span></div><div class="qw-oi"><span class="qw-ok qw-bs">SNR after the chain</span><span class="qw-ov" id="qwp-snr">-</span><span class="qw-os">amplifier noise only, 5 dB NF</span></div><div class="qw-oi"><span class="qw-ok qw-fs">transmitter's share</span><span class="qw-ov" id="qwp-tx">-</span></div><div class="qw-oi qw-grow"><span class="qw-ok" id="qwp-xk">&nbsp;</span><span class="qw-ov" id="qwp-x">-</span></div></div><p class="qw-n" id="qwp-note"></p><noscript><p class="qw-n">On MAREA, 110 spans of 9.6 dB, about 89% of the photons reaching the receiver were emitted by the last amplifier, 10% by the one before it and 1% by the one before that. The transmitter's share is 10<sup>&minus;105.6</sup>. A 1 mW classical symbol still arrives with an SNR of about 16 dB, because each amplifier receives roughly 13,000 photons per symbol and adds the equivalent of about 3. Send one qubit instead and the photon is lost before the first amplifier 89% of the time; when it arrives, the best fidelity any copy can have to the original is 0.70, against 0.67 for measuring and guessing.</p></noscript></div>

flip it to one qubit and nothing about the line changes. what changes is that there are no spare photons left to hand the information on with.

two more things worth trying. the three line presets all add up to the same 1,056 dB, so the transmitter's share never moves, but the SNR does: shorter spans buy back almost 2 dB, longer spans lose more than 6. that is the "5 dB for free" argument from earlier, seen from the photon's side. and in qubit mode, shorter spans make each copy *better*, because a lower-gain amplifier makes fewer copies. at 3 dB, the bottom of the slider, the copy fidelity is 0.83, and that is the first amplifier of a few hundred.

photons do not carry name tags, so this is bookkeeping rather than biography. but the bookkeeping is the point. **a classical cable works because the information is handed from photon to photon at every span, and a qubit is the one kind of information that cannot be handed over.**

## what if you take the amplifiers out?

then the fiber finishes the job instead. [the data center post]({% post_url 2026-08-03-quantum-data-centers-when-a-hop-costs-an-exponent %}) was built on loss sitting in an exponent, where every 3 dB doubles the time to get a pair. inside a building, what sat in that exponent was a few switches. here it is 1,056 dB.

```
η  =  10^(−1056/10)  ≈  10^(−105.6)

source at 1e10 photons/s   →   one arrival every ~10^95.6 s
age of the universe        ≈   4e17 s
```

cleverness does not help either. [the PLOB bound](https://arxiv.org/abs/1510.08863) caps any quantum protocol over a lossy channel, with no quantum node in the middle, at

```
C  =  −log2(1 − η)  ≈  1.44·η     bits per channel use
```

twin-field QKD puts an untrusted measurement station in the middle and improves the scaling to √η, which is how it has [run past 1,000 km of spooled fiber](https://arxiv.org/abs/2303.15795). the square root of 10<sup>−105.6</sup> is 10<sup>−52.8</sup>. fifty-three orders of magnitude better, and still nothing.

the general shape is the useful part. put stations in the line and, at best, each one only has to cross its own segment, so **the exponent gets divided by the number of segments.** that is the entire argument for quantum repeaters, and it comes with a price.

here it is with every experiment so far plotted on it:

<div class="qw" id="qw-ch" data-qw><div class="qw-hd"><span class="qw-t">chop the exponent</span><span class="qw-s">the chance that a single photon makes it falls exponentially with distance. put stations in the line and, at best, each segment only has to cross itself, so the exponent is divided by the number of segments. an envelope, not a protocol: perfect memories, deterministic swaps, no multiplexing, no purification.</span><span class="qw-lg"><i style="background:#32c29e"></i>straight through<i style="background:#f2a03d"></i>one station in the middle<i style="background:#7f8fd6"></i>your chain, per segment</span><span class="qw-lg"><i style="background:transparent;border:1.5px solid #cfcfcf;width:9px;height:9px;border-radius:50%"></i>fiber experiments, at their measured loss</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">jump to</span><span class="qw-b" data-grp="pre"><button type="button" data-v="ms">Malta-Sicily, 96 km</button><button type="button" data-v="loop">the Sicily loop, 192 km</button><button type="button" data-v="m1">MAREA, straight through</button><button type="button" data-v="m66" class="on" aria-pressed="true">MAREA, 66 &times; 100 km</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">distance <b id="qwc-dv"></b></span><input type="range" id="qwc-d" min="0" max="1000" step="1" value="906"></label><label class="qw-sr"><span class="qw-l">segments <b id="qwc-nv"></b></span><input type="range" id="qwc-n" min="0" max="18" step="1" value="15"></label><label class="qw-sr"><span class="qw-l">fiber loss <b id="qwc-av"></b></span><input type="range" id="qwc-a" min="0.15" max="0.26" step="0.001" value="0.16"></label></div><svg class="qw-svg" viewBox="0 0 640 320" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Chance per attempt on a logarithmic scale against distance: straight through, one station in the middle, and a chain of segments, with real experiments marked"><g id="qwc-plot"></g></svg><div class="qw-out"><div class="qw-oi"><span class="qw-ok">total loss</span><span class="qw-ov" id="qwc-L">-</span></div><div class="qw-oi"><span class="qw-ok qw-bs">straight through</span><span class="qw-ov" id="qwc-E">-</span><span class="qw-os">chance per photon</span></div><div class="qw-oi"><span class="qw-ok">one photon every</span><span class="qw-ov" id="qwc-T">-</span><span class="qw-os" id="qwc-Ts"></span></div><div class="qw-oi"><span class="qw-ok">repeaterless bound</span><span class="qw-ov" id="qwc-B">-</span><span class="qw-os">bits per channel use, any protocol</span></div><div class="qw-oi"><span class="qw-ok qw-fs">one station in the middle</span><span class="qw-ov" id="qwc-M">-</span><span class="qw-os">chance per attempt, &radic;&eta;</span></div><div class="qw-oi"><span class="qw-ok qw-ps">your chain, per segment</span><span class="qw-ov" id="qwc-C">-</span><span class="qw-os" id="qwc-Cs"></span></div><div class="qw-oi qw-grow"><span class="qw-ok">memories must hold</span><span class="qw-ov" id="qwc-H">-</span><span class="qw-os" id="qwc-Hs"></span></div></div><p class="qw-n" id="qwc-note"></p><noscript><p class="qw-n">At 6,600 km of 0.16 dB/km fiber, the chance that one photon crosses is 10<sup>&minus;105.6</sup>: a source emitting ten billion photons a second would wait about 10<sup>95.6</sup> seconds per arrival, roughly 10<sup>78</sup> times the age of the universe. One station in the middle, the twin-field scaling, improves that to 10<sup>&minus;52.8</sup>. Cutting the same distance into 66 segments of 100 km leaves each segment a 2.5% chance per attempt, but every node's memory then has to hold for about 120 ms, roughly 90 for the slowest segment to succeed and 32 for the result to cross the ocean. The longest fiber experiment marked here covers about 6% of the distance to Bilbao.</p></noscript></div>

two things worth noticing.

**every fiber experiment so far sits at the left edge.** the Malta-Sicily cable at 22 dB, its 192 km loop at 48 dB, the USTC memories at 78.7 dB. jump to MAREA and they collapse into the first few percent of the axis, with about a thousand dB still to go.

**chopping only helps if every piece can wait for the others.** with 66 segments of 100 km, each segment succeeds about 2.5% of the time per attempt, which sounds very doable. but the memories at every node have to hold their half until the slowest segment comes through and the result has crossed the ocean, and light alone needs about 32 ms for the crossing. **however finely you chop, the memories wait at least 32 ms**, and on this napkin, with perfect everything, closer to 120.

the napkin is deliberately generous: perfect memories, deterministic swaps, every segment attempting in parallel once per round trip, no multiplexing and no purification. a real chain is slower than this, never faster.

[the ion post]({% post_url 2026-08-30-Long-lived-ion-entanglement %}) was about the first time a memory pair outlived its own wait: 550 ms of coherence against a 450 ms average generation time, over 10 km, for one link, with the stored pair expected to have decayed to a fidelity of 0.578 by the time the next success arrived. that is the right order of magnitude for the Atlantic. it is also one link, and the Atlantic is dozens of them in a row.

## what would work instead

a quantum repeater works on a completely different principle from an amplifier. it never copies anything.

1. generate entanglement over short links, tens of kilometres long.
2. store each half in a quantum memory until the neighbouring link also succeeds.
3. do a joint Bell measurement at the junction, an entanglement swap, which fuses two short pairs into one longer one.
4. repeat hierarchically, purifying as you go, until a pair spans the whole distance, then use it to teleport the qubit.

loss still happens on every link, but failures are local and get retried, so the cost grows polynomially with distance instead of exponentially. [the store-and-forward post]({% post_url 2026-06-08-Memory-enhanced-quantum-communication %}) covered the first demonstration that a single memory in the middle can beat direct transmission at all.

the building blocks are only now becoming real. in August a USTC team [entangled two atomic memories through 420 km of fiber](https://arxiv.org/abs/2504.05660), and beyond 320 km their rate beat the PLOB bound, the most any direct transmission could deliver. the caveats matter: both memories sat in the same lab, and at the longest setting each arm was about 10 km of deployed fiber plus roughly 200 km of ultra-low-loss fiber on a spool. one link, not a chain.

satellites are the realistic near-term route. in free space the beam spreads, so loss grows with the square of the distance instead of exponentially, and only the last stretch of atmosphere absorbs anything. Micius [sent entangled photons to two ground stations 1,120 km apart in 2020](https://doi.org/10.1038/s41586-020-2401-y), for QKD that did not require trusting the satellite. the intercontinental satellite links so far, [China to Austria over 7,600 km](https://arxiv.org/abs/1801.04418) and [China to South Africa over 12,900 km with the Jinan-1 microsatellite](https://www.nature.com/articles/s41586-025-08739-z), used the satellite as a trusted relay. whoever compromises the satellite learns the key.

## has anybody tried the Atlantic?

not with qubits, as far as i can find.

in March, Toshiba Europe and Quantum Bridge Technologies [demonstrated quantum-safe data transmission between the UK and Canada](https://www.telecomtv.com/content/security/toshiba-goes-transatlantic-with-quantum-safe-connectivity-55173/). QKD ran inside metro networks at each end, in Telehouse facilities in Cambridge and Toronto, and the ocean crossing used QBT's distributed symmetric key establishment, which builds keys from symmetric cryptography, secret sharing and pre-shared entropy ([technical paper](https://www.toshiba.eu/quantum/wp-content/uploads/resources/QKD_DSKE-Submit.pdf)). it is real engineering, and Toshiba is upfront that it exists because ultra-long-distance fiber QKD does not and satellite QKD services are scarce. but no photon crossed the ocean in a quantum state. **"quantum-safe" and trans-Atlantic in the same headline means cryptography and key management, not qubits.**

the quantum experiments that have actually used a submarine telecom cable are short and unamplified:

<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Experiment</th><th>Fiber</th><th>Loss</th><th>Result</th></tr>
</thead>
<tbody>
<tr><td><a href="https://arxiv.org/abs/1803.00583">Malta-Sicily, 2018</a></td><td>96 km, one dark fiber</td><td>about 22 dB</td><td>about 257 pairs/s, visibility above 90%</td></tr>
<tr><td><a href="https://arxiv.org/abs/1907.04864">Sicily and back, 2019</a></td><td>192 km, two fibers of the same cable</td><td>48 dB</td><td>Bell-state fidelity 85% for hours, no active stabilization</td></tr>
<tr><td>MAREA, for scale</td><td>6,600 km</td><td>about 1,056 dB</td><td>nothing yet</td></tr>
</tbody>
</table>

both carried polarization-entangled photons over a deployed cable with nothing in the line. the longer one is under 3% of the way across.

the Atlantic is on the agenda, by satellite:

- the Canadian Space Agency put out [a call worth up to $5M](https://www.asc-csa.gc.ca/eng/funding-programs/funding-opportunities/ao/2026-secure-transatlantic-quantum-communications-demonstration.asp) for a Canada-UK demonstration that shares quantum keys via satellite between at least two ground stations, following a June 2025 commitment by the two prime ministers.
- Canada's [QEYSSat](https://nrc.canada.ca/en/research-development/research-collaboration/programs/qeyssat-future-secure-communications-canada), in low orbit about 500 km up, is scheduled for late 2026, and it flips the usual design: the photons go up from the ground to the satellite.
- [a Waterloo study](https://arxiv.org/abs/2606.09805) of eight architectures for trans-Atlantic entanglement found the best was a source on a low-orbit satellite with two passive relays on high-altitude platforms, worth about 5×10<sup>6</sup> secure key bits a year with 30 cm ground telescopes, nearly two orders of magnitude more than one medium-orbit satellite with 1 m telescopes. averaged over a year that is about one bit every six seconds, and it is still a plan on paper.

## what i'd push on

no paper this time, so the push is aimed at the field rather than at authors.

**1. say which photons crossed the ocean.** the UK-Canada demo is a perfectly good result described in a way that invites the wrong reading. the fix is a one-line convention for any long-haul "quantum" link: for every hop, say whether it carried qubits, went through a trusted node, or ran classical cryptography, and if classical, whether the security is computational or comes from pre-shared secrets. [the quantum LAN post]({% post_url 2026-09-07-a-quantum-lan-on-deployed-fiber %}) asked for an integration time and a basis set behind every ebits-per-second number. this is the same ask, aimed at press releases.

**2. print the repeaterless bound next to every distance.** a distance on its own says very little. 420 km on a spool, 420 km of buried fiber and 420 km with a memory in the middle are three different experiments. the transmission, the PLOB bound at that transmission, and the achieved rate fit on one line. USTC printed that comparison, and it is what makes their result more than a distance record. print the counting convention too: the store-and-forward post found the same experiment at 1.43× or 0.71× the repeaterless capacity depending on how channel uses were counted.

**3. write the seabed node spec in watts before coherence times.** a quantum repeater node on a trans-Atlantic cable would get what a classical repeater gets: tens of watts off a constant-current line, at 2 to 4 °C, under kilometres of water, for 25 years, with no service visit. the memories this blog has covered are a silicon vacancy at about 20 mK in a dilution refrigerator, trapped calcium ions, and clouds of laser-cooled rubidium. none of them fits that envelope today, and coherence time is not the number that rules them out. the first fiber repeater chains will be built on land, in huts with mains power and a technician, and anybody drawing a trans-oceanic quantum cable should draw the power feed first.

**4. let the cable be the classical half.** a satellite QKD link still needs a classical channel for sifting and error correction, and an entanglement link needs heralds and clock comparisons. those will cross the ocean on one of these cables. the cable is also already an instrument: polarization tracking on a transoceanic cable [has picked up earthquakes and ocean swell](https://par.nsf.gov/biblio/10220270-optical-polarizationbased-seismic-water-wave-sensing-transoceanic-cables), and the loopbacks on a UK-Canada cable [were turned into span-by-span seafloor sensors](https://www.research.ed.ac.uk/en/publications/optical-interferometry-based-array-of-seafloor-environmental-sens/). the trans-Atlantic quantum link i would bet on is a satellite on top and a submarine cable underneath, and the interface between them (timing, framing, getting photons from a telescope into a metro network) is exactly where people who build metro testbeds come in.

## the short version

- a trans-Atlantic cable is about 1,056 dB of glass. there is no link budget for that, so it is **chopped into roughly a hundred spans** of 50 to 90 km, with an EDFA at the end of each.
- every amplifier's noise arrives at the far end. OSNR pays `10·log10(N)`, which is **3 dB for every doubling of distance**.
- the real ceiling is noise against the glass. Kerr nonlinearity grows as P³, the optimum launch is where it equals half the ASE, and large-area fiber raises the ceiling.
- dispersion and polarization became the receiver's problem: about 140,000 ps/nm undone in DSP, and dispersion maps are gone.
- capacity is logarithmic in SNR and linear in fibers, and the seabed is power-limited, so new cables carry 12 to 16 pairs run below their optimum. **watts are scarcer than glass.**
- the repeaters never convert to electricity. an EDFA copies photons by stimulated emission, and **copying an unknown quantum state is exactly what no-cloning forbids.**
- the 3 dB quantum limit on amplifier noise is what sits inside the `NF` in the OSNR formula. a classical symbol has about 10<sup>5</sup> photons to spare. a qubit has one.
- about 90% of the photons reaching the receiver were emitted by the last amplifier. the transmitter's share is 10<sup>−105.6</sup>.
- take the amplifiers out and a 10 GHz single-photon source waits 10<sup>95.6</sup> seconds per arrival. no repeaterless protocol beats about 1.44η, and one station in the middle only gets √η.
- repeaters divide the exponent by the number of segments, but every memory must hold for at least the 32 ms the result needs to cross, and the state of the art is one link: USTC, 420 km, one lab.
- nothing quantum has crossed the Atlantic by fiber. the UK-Canada "quantum-safe" demo used QKD at both ends and key management in the middle, and the submarine quantum experiments i can find are 96 and 192 km of unamplified dark fiber.

the thing i did not appreciate before writing this is how completely a submarine cable depends on never delivering the photon it was given. a hundred amplifiers each make a fresh, slightly noisier copy, and the information survives because it was spread across a hundred thousand photons per symbol. **the internet crosses the Atlantic by copying, and copying is the one thing a qubit forbids.** so the first qubit to make the crossing will most likely go up a few hundred kilometres and come back down, and it will land in a metro network a lot like the ones this blog usually writes about. the cable on the seafloor will carry the classical half of that conversation which, for a machine built entirely out of copies, is exactly the job it was designed for.

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
.qw-ok.qw-ps{color:#7f8fd6}
.qw-ov{display:block;font-family:ui-monospace,SFMono-Regular,Menlo,Consolas,monospace;font-size:14px;color:#fff;margin-top:2px;font-variant-numeric:tabular-nums}
.qw-ov sup{font-size:9px;line-height:0}
.qw-os{display:block;font-size:10.5px;color:#8f8f8f;margin-top:2px;line-height:1.35}
.qw-os sup{font-size:8px;line-height:0}
.qw-n sup{font-size:9px;line-height:0}
.qw-ov.qw-txt{font-family:inherit;font-size:13px;font-weight:600}
.qw-n{font-size:12px;color:#9a9a9a;margin:11px 0 0;line-height:1.55}
.qw-n b{color:#cfcfcf;font-weight:600}
@media(max-width:600px){.qw{padding:14px 12px 12px}.qw-ctl{gap:12px}}
</style>
<script>
(function(){
  var GREEN='#32c29e', AMBER='#f2a03d', PURPLE='#7f8fd6', BLUE='#6ba3f0', AXIS='#3a3a3a', TICK='#7d7d7d', LIGHT='#cfcfcf';
  var MINUS='−';
  function $(id){ return document.getElementById(id); }
  function fm(v){ return (v<0?MINUS:'')+Math.abs(v).toFixed(1); }
  function km(d){ return Math.round(d).toLocaleString('en-US'); }
  function sig(x,k){ var v=Number(x.toPrecision(k)); return v.toLocaleString('en-US',{maximumSignificantDigits:k}); }
  function ex(l){ return '10<sup>'+(l<0?MINUS:'')+Math.abs(l).toFixed(1)+'</sup>'; }
  function mark(root,grp,val){
    var bs=root.querySelectorAll('[data-grp="'+grp+'"] button'), i;
    for(i=0;i<bs.length;i++){
      var on=bs[i].getAttribute('data-v')===val;
      bs[i].classList.toggle('on',on);
      if(on) bs[i].setAttribute('aria-pressed','true'); else bs[i].removeAttribute('aria-pressed');
    }
  }
  function onBtn(root,grp,cb){
    var bs=root.querySelectorAll('[data-grp="'+grp+'"] button');
    Array.prototype.forEach.call(bs,function(b){ b.addEventListener('click',function(){ cb(b.getAttribute('data-v')); }); });
  }
  function vw(R){ var w=R.clientWidth||640; return w<520 ? 400 : 640; }
  function onResize(fn){ var t=null; window.addEventListener('resize',function(){ clearTimeout(t); t=setTimeout(fn,120); }); }
  function hline(x1,y,x2,op){ return '<line x1="'+x1+'" y1="'+y+'" x2="'+x2+'" y2="'+y+'" stroke="'+AXIS+'" stroke-width="1" opacity="'+op+'"/>'; }
  function txt(x,y,s,o){
    o=o||{};
    return '<text x="'+x+'" y="'+y+'" fill="'+(o.fill||TICK)+'" font-size="'+(o.fs||9.5)+'" text-anchor="'+(o.a||'start')+'"'+(o.extra||'')+'>'+s+'</text>';
  }

  /* =======================================================================
     WIDGET 1 - noise against the glass

     Incoherent GN model, fibre loss only, identical spans, full C-band.

       N      = ceil(distance / span)
       G      = span loss, linear
       P_ASE  = N . NF . h nu . G . Rs                 amplifier noise, per channel
       eta    = N . (8/27) gamma^2 Leff^2 asinh(pi^2/2 |b2| La B^2)
                  / (pi |b2| La Rs^2)                   nonlinear coefficient
       SNR(P) = P / (P_ASE + eta P^3)

     The optimum is P_opt = (P_ASE / 2 eta)^(1/3), where the nonlinear term is
     half the ASE, and the peak SNR is P_opt / (1.5 P_ASE).  Shannon ceiling is
     2 B log2(1 + SNR) for two polarisations over B = 4.5 THz.
     ==================================================================== */
  (function(){
    var R=$('qw-gn'); if(!R) return;
    var h=6.62607e-34, c=2.99792458e8, lam=1.55e-6, nu=c/lam, n2=2.6e-20, Rs=64e9, B=4.5e12, NCH=Math.floor(B/Rs);
    var F={m:{a:0.20,ae:80,D:17},u:{a:0.16,ae:125,D:21},w:{a:0.155,ae:150,D:21}};
    var PRE={marea:{sp:60,ds:6600,nf:5,f:'u'},pac:{sp:60,ds:10000,nf:5,f:'u'},metro:{sp:80,ds:6600,nf:5,f:'m'}};
    var fk='u';
    var sp=$('qwg-sp'), ds=$('qwg-ds'), nf=$('qwg-nf'), G=$('qwg-plot');

    function model(s,d,nfdb,f){
      var N=Math.max(1,Math.ceil(d/s));
      var ap=f.a/(10*Math.LOG10E)/1000, L=s*1000, loss=f.a*s, Gl=Math.pow(10,loss/10);
      var Le=(1-Math.exp(-ap*L))/ap, La=1/ap;
      var g=2*Math.PI*n2/(lam*f.ae*1e-12);
      var b2=f.D*1e-6*lam*lam/(2*Math.PI*c);
      var k=(8/27)*g*g*Le*Le*Math.asinh(Math.PI*Math.PI/2*b2*La*B*B)/(Math.PI*b2*La);
      var eta=k*N/(Rs*Rs);
      var Pase=N*Math.pow(10,nfdb/10)*h*nu*Gl*Rs;
      var Po=Math.cbrt(Pase/(2*eta));
      return {N:N,loss:loss,eta:eta,Pase:Pase,Po:Po,snr:Po/(1.5*Pase)};
    }

    var svgG=R.querySelector('svg'), ml=50, mr=14, mt=18, mb=40, x0=-10, x1=8;

    function draw(){
      var s=+sp.value, d=+ds.value, nfv=+nf.value, m=model(s,d,nfv,F[fk]);
      var W=vw(R), H=W<640?300:320, pw=W-ml-mr, ph=H-mt-mb;
      svgG.setAttribute('viewBox','0 0 '+W+' '+H);
      $('qwg-spv').textContent=s+' km';
      $('qwg-dsv').textContent=km(d)+' km';
      $('qwg-nfv').textContent=nfv.toFixed(1)+' dB';

      var pk=10*Math.log10(m.snr), po=10*Math.log10(m.Po/1e-3);
      var y1=Math.ceil((pk+6)/5)*5, y0=y1-25;
      function X(p){ return ml+(p-x0)/(x1-x0)*pw; }
      function Y(v){ return mt+(y1-v)/(y1-y0)*ph; }
      var a='', b='', t='', i, p, P, cmd, xx;
      for(i=0;i<=180;i++){
        p=x0+i*0.1; P=1e-3*Math.pow(10,p/10);
        cmd=(i===0)?'M':'L'; xx=X(p).toFixed(1);
        t+=cmd+xx+','+Y(10*Math.log10(P/(m.Pase+m.eta*P*P*P))).toFixed(1);
        a+=cmd+xx+','+Y(10*Math.log10(P/m.Pase)).toFixed(1);
        b+=cmd+xx+','+Y(-10*Math.log10(m.eta*P*P)).toFixed(1);
      }
      var g='', v;
      for(v=y0;v<=y1;v+=5){
        var yy=Y(v).toFixed(1);
        g+=hline(ml,yy,W-mr,0.45);
        g+=txt(ml-7,(+yy+3.5).toFixed(1),v,{a:'end'});
      }
      for(p=x0;p<=x1;p+=(W<640?4:2)){
        g+='<line x1="'+X(p).toFixed(1)+'" y1="'+mt+'" x2="'+X(p).toFixed(1)+'" y2="'+(mt+ph)+'" stroke="'+AXIS+'" stroke-width="1" opacity="0.22"/>';
        g+=txt(X(p).toFixed(1),mt+ph+15,(p<0?MINUS+(-p):p),{a:'middle'});
      }
      var mx=X(po), my=Y(pk);
      G.innerHTML=g+
        '<defs><clipPath id="qwg-clip"><rect x="'+ml+'" y="'+mt+'" width="'+pw+'" height="'+ph+'"/></clipPath></defs>'+
        '<line x1="'+ml+'" y1="'+(mt+ph)+'" x2="'+(W-mr)+'" y2="'+(mt+ph)+'" stroke="'+AXIS+'" stroke-width="1.5"/>'+
        '<g clip-path="url(#qwg-clip)" fill="none" stroke-linecap="round" stroke-linejoin="round">'+
        '<path d="'+a+'" stroke="'+AMBER+'" stroke-width="1.8" stroke-dasharray="6 4"/>'+
        '<path d="'+b+'" stroke="'+PURPLE+'" stroke-width="1.8" stroke-dasharray="1.5 4"/>'+
        '<path d="'+t+'" stroke="'+GREEN+'" stroke-width="2.4"/></g>'+
        '<line x1="'+mx.toFixed(1)+'" y1="'+(my+6).toFixed(1)+'" x2="'+mx.toFixed(1)+'" y2="'+(mt+ph)+'" stroke="#dedede" stroke-width="1" stroke-dasharray="2 3" opacity="0.5"/>'+
        '<circle cx="'+mx.toFixed(1)+'" cy="'+my.toFixed(1)+'" r="4.5" fill="'+GREEN+'" stroke="#111" stroke-width="1.2"/>'+
        txt((mx+8).toFixed(1),(my+40).toFixed(1),'optimum',{fill:LIGHT})+
        txt(ml+6,mt+ph-7,'noise-limited',{fs:9})+
        txt(W-mr-6,mt+ph-7,'nonlinearity-limited',{fs:9,a:'end'})+
        txt((ml+pw/2),H-6,'launch power per channel (dBm)',{fs:10.5,a:'middle'})+
        txt(ml-36,(mt+ph/2),'received SNR (dB)',{fs:10.5,a:'middle',extra:' transform="rotate(-90 '+(ml-36)+' '+(mt+ph/2)+')"'});

      $('qwg-N').textContent=m.N;
      $('qwg-Ns').textContent=(m.N-1)+' on the seabed';
      $('qwg-L').textContent=m.loss.toFixed(1)+' dB';
      $('qwg-P').textContent=fm(po)+' dBm';
      $('qwg-Ps').textContent='per channel, about '+fm(po+10*Math.log10(NCH))+' dBm per fiber';
      $('qwg-S').textContent=fm(pk)+' dB';
      $('qwg-Ss').textContent='about '+fm(pk+10*Math.log10(Rs/12.5e9))+' dB OSNR in 0.1 nm';
      $('qwg-C').textContent=Math.round(2*B*Math.log2(1+m.snr)/1e12)+' Tb/s';
      $('qwg-note').innerHTML=m.N+' amplifiers over '+km(d)+' km. the best launch is <b>'+fm(po)+' dBm per channel</b>, where the received SNR peaks at '+fm(pk)+' dB. each dB of launch power above that adds 3 dB of nonlinear noise, and each dB below it hands a dB of SNR back to the amplifiers.';

      var key=null, kk, q;
      for(kk in PRE){ if(PRE.hasOwnProperty(kk)){ q=PRE[kk]; if(q.sp===s && q.ds===d && Math.abs(q.nf-nfv)<0.01 && q.f===fk) key=kk; } }
      mark(R,'pre',key); mark(R,'fb',fk);
    }
    [sp,ds,nf].forEach(function(el){ el.addEventListener('input',draw); });
    onBtn(R,'fb',function(v){ fk=v; draw(); });
    onBtn(R,'pre',function(v){ var q=PRE[v]; sp.value=q.sp; ds.value=q.ds; nf.value=q.nf; fk=q.f; draw(); });
    onResize(draw);
    draw();
  })();

  /* =======================================================================
     WIDGET 2 - whose photons are these

     N amplifiers, each with gain G matched to the span loss in front of it.
     Of the photons leaving an amplifier, 1/G came in and (1 - 1/G) were
     emitted there.  So at the receiver:

       share(last amplifier)      = 1 - T
       share(k-th from the end)   = (1 - T) T^(k-1)
       share(transmitter)         = T^N              T = 1/G

     Noise: an amplifier with noise figure NF adds, referred to its input,
     about NF photons per symbol (the GN widget uses the same G ~ G-1
     approximation).  N matched amplifiers add N times that, so

       SNR = n_launch T / (N . NF)

     For one photon the relevant number is the optimal universal 1 -> M qubit
     cloner, F = (2M + 1) / 3M, with M ~ G copies.  It tends to 2/3, which is
     what measuring the photon and guessing gets you.
     ==================================================================== */
  (function(){
    var R=$('qw-cp'); if(!R) return;
    var NF=Math.pow(10,0.5), NPH={cl:1.2e5,q:1};
    var LN={marea:{l:9.6,n:110},short:{l:4.8,n:220},long:{l:19.2,n:55}};
    var mode='cl', sl=$('qwp-l'), sn=$('qwp-n'), G=$('qwp-bars'), svg=R.querySelector('svg');

    function share(l){
      var p=Math.pow(10,l);
      if(p>=0.001) return sig(p*100,2)+'%';
      return '10<tspan dy="-4" font-size="8">'+MINUS+Math.abs(l).toFixed(1)+'</tspan>';
    }
    function photons(x){ return x>=100 ? km(x) : sig(x,2); }

    function draw(){
      var L=+sl.value, N=+sn.value, T=Math.pow(10,-L/10), g=1/T;
      $('qwp-lv').textContent=L.toFixed(1)+' dB';
      $('qwp-nv').textContent=N;

      var rows=[], k, names=['the last amplifier','the one before it','two before','three before','four before'];
      for(k=1;k<=Math.min(N,5);k++) rows.push({n:names[k-1],l:Math.log10(1-T)+(k-1)*Math.log10(T)});
      if(N>5) rows.push({n:'the other '+(N-5),l:5*Math.log10(T)+Math.log10(1-Math.pow(T,N-5))});
      rows.push({n:'the transmitter',l:-N*L/10,tx:true});

      var VW=vw(R), lw=VW<640?118:150, x0=lw+8, x1=VW-78, rh=28, H=rows.length*rh+8, out='', i, r, w;
      svg.setAttribute('viewBox','0 0 '+VW+' '+H);
      for(i=0;i<rows.length;i++){
        r=rows[i]; var y=4+i*rh, p=Math.pow(10,r.l);
        w=Math.max(1.5,(x1-x0)*p);
        out+=txt(lw,y+rh/2+3.5,r.n,{fs:10.5,a:'end',fill:r.tx?LIGHT:'#b4b4b4'});
        out+='<rect x="'+x0+'" y="'+(y+6)+'" width="'+(x1-x0)+'" height="'+(rh-12)+'" fill="#212121" rx="2"/>';
        out+='<rect x="'+x0+'" y="'+(y+6)+'" width="'+w.toFixed(1)+'" height="'+(rh-12)+'" fill="'+(r.tx?AMBER:GREEN)+'" rx="2"/>';
        out+='<text x="'+(Math.min(x0+w,x1)+8).toFixed(1)+'" y="'+(y+rh/2+3.5)+'" fill="'+(r.tx?AMBER:'#e6e6e6')+'" font-size="10.5">'+share(r.l)+'</text>';
      }
      G.innerHTML=out;

      var n=NPH[mode], nin=n*T, snr=10*Math.log10(nin/(N*NF));
      $('qwp-in').textContent=(mode==='cl'?'120,000':'1');
      $('qwp-arr').textContent=photons(nin);
      $('qwp-snr').textContent=fm(snr)+' dB';
      $('qwp-tx').innerHTML=ex(-N*L/10);
      var note;
      if(mode==='cl'){
        $('qwp-xk').textContent="signal against one amplifier's noise";
        $('qwp-x').textContent=Math.round(nin/NF).toLocaleString('en-US')+' to 1';
        note='each amplifier receives about '+photons(nin)+' photons per symbol and adds the equivalent of about 3 at its input. after '+N+' of them the SNR is still <b>'+fm(snr)+' dB</b>, which a coherent receiver is happy with. almost none of the photons are the ones that were sent, and that does not matter, because the information was handed on at every span.';
      } else {
        var F=(2*g+1)/(3*g);
        $('qwp-xk').textContent='best fidelity of any copy';
        $('qwp-x').textContent=F.toFixed(2);
        note='the photon is lost before the first amplifier '+Math.round((1-T)*100)+'% of the time. when it does arrive, a '+L.toFixed(1)+' dB amplifier turns it into about '+(g<10?g.toFixed(1):Math.round(g))+' photons plus noise, and the best fidelity any one of them can have to the original state is <b>'+F.toFixed(2)+'</b>. measuring the photon and guessing gets 0.67. the rest of the chain cannot win any of that back.';
      }
      $('qwp-note').innerHTML=note;
      var key=null, kk;
      for(kk in LN){ if(LN.hasOwnProperty(kk) && Math.abs(LN[kk].l-L)<0.01 && LN[kk].n===N) key=kk; }
      mark(R,'ln',key); mark(R,'md',mode);
    }
    sl.addEventListener('input',draw);
    sn.addEventListener('input',draw);
    onBtn(R,'md',function(v){ mode=v; draw(); });
    onBtn(R,'ln',function(v){ sl.value=LN[v].l; sn.value=LN[v].n; draw(); });
    onResize(draw);
    draw();
  })();

  /* =======================================================================
     WIDGET 3 - chop the exponent

     eta = 10^(-alpha d / 10).  Straight through: eta.  One untrusted station
     in the middle (twin-field scaling): sqrt(eta).  A chain of n segments,
     best case: eta^(1/n) per segment.  Repeaterless bound (PLOB):
     -log2(1 - eta).

     Memory hold time for the chain, perfect everything: segments all attempt
     in parallel, one attempt per segment round trip T_a = (d/n)/v.  The
     expected maximum of n geometric waits with success p is about
     H_n / (-ln(1 - p)) attempts, then the result needs d/v to cross.
     ==================================================================== */
  (function(){
    var R=$('qw-ch'); if(!R) return;
    var NSEG=[1,2,3,4,5,6,8,10,12,16,20,24,32,40,48,66,80,96,128];
    var V=2.04e5, AGE=4.35e17, LAGE=Math.log10(AGE), YEAR=3.156e7;
    var EXP=[{t:'Malta-Sicily',d:96,db:22},{t:'Sicily loop',d:192,db:48},{t:'USTC, 420 km',d:420,db:78.7}];
    var PRE={ms:{d:96,a:0.229,n:1},loop:{d:192,a:0.25,n:1},m1:{d:6600,a:0.16,n:1},m66:{d:6600,a:0.16,n:66}};
    var ds=$('qwc-d'), ns=$('qwc-n'), as=$('qwc-a'), G=$('qwc-plot');
    var dist=6600, alpha=0.16;

    function s2d(v){ return 10*Math.pow(1300,v/1000); }
    function d2s(d){ return Math.round(1000*Math.log(d/10)/Math.log(1300)); }
    function niceD(d){ if(d<100) return Math.round(d); if(d<1000) return Math.round(d/5)*5; return Math.round(d/50)*50; }
    function segLen(d,n){ var L=d/n; return L<10 ? L.toFixed(1) : km(L); }
    function chance(l){ if(l>-6) return '1 in '+sig(Math.pow(10,-l),3); return ex(l); }
    function pct(l){ var p=Math.pow(10,l); if(p>=0.001) return sig(p*100,2)+'%'; return chance(l); }
    function tm(ls){
      if(ls<-6) return sig(Math.pow(10,ls)*1e9,2)+' ns';
      if(ls<-3) return sig(Math.pow(10,ls)*1e6,2)+' µs';
      if(ls<0) return sig(Math.pow(10,ls)*1e3,2)+' ms';
      var s=Math.pow(10,ls);
      if(s<60) return sig(s,2)+' s';
      if(s<3600) return sig(s/60,2)+' min';
      if(s<86400) return sig(s/3600,2)+' h';
      if(s<YEAR) return sig(s/86400,2)+' days';
      if(s<YEAR*1e6) return sig(s/YEAR,2)+' years';
      return '10<sup>'+ls.toFixed(1)+'</sup> s';
    }
    function harm(n){ var s=0,k; for(k=1;k<=n;k++) s+=1/k; return s; }
    function niceCeil(x){ var e=Math.pow(10,Math.floor(Math.log10(x))),m=x/e,c=[1,1.5,2,2.5,3,4,5,6,8,10],i; for(i=0;i<c.length;i++){ if(m<=c[i]+1e-9) return c[i]*e; } return 10*e; }
    function niceStep(r,maxT){ var c=[0.5,1,2,5,10,20,25,50,100,200],i; for(i=0;i<c.length;i++){ if(r/c[i]<=maxT) return c[i]; } return 500; }

    var svgC=R.querySelector('svg'), ml=56, mr=14, mt=18, mb=40;

    function draw(){
      var n=NSEG[+ns.value], d=dist, a=alpha;
      var W=vw(R), H=W<640?300:320, pw=W-ml-mr, ph=H-mt-mb, nt=W<640?4:6;
      svgC.setAttribute('viewBox','0 0 '+W+' '+H);
      $('qwc-dv').textContent=km(d)+' km';
      $('qwc-nv').textContent=(n===1?'1, straight through':n+' × '+segLen(d,n)+' km');
      $('qwc-av').textContent=a.toFixed(3)+' dB/km';
      var Lt=a*d, le=-Lt/10, lm=le/2, lp=le/n;

      var xmax=niceCeil(d*1.25), ybot=-a*xmax/10;
      var st=niceStep(-ybot,nt); ybot=-Math.ceil(-ybot/st-1e-9)*st;
      function X(x){ return ml+x/xmax*pw; }
      function Y(y){ return mt+(y/ybot)*ph; }
      var g='', v, yy, xs=(xmax<1000)?niceStep(xmax,nt):niceStep(xmax/1000,nt)*1000;
      for(v=0;v>=ybot-1e-9;v-=st){
        yy=Y(v).toFixed(1);
        g+=hline(ml,yy,W-mr,0.45);
        g+='<text x="'+(ml-7)+'" y="'+(+yy+3.5).toFixed(1)+'" fill="'+TICK+'" font-size="9.5" text-anchor="end">10<tspan dy="-4" font-size="7.5">'+(v===0?'0':MINUS+Math.abs(v))+'</tspan></text>';
      }
      for(v=0;v<=xmax+1e-9;v+=xs) g+=txt(X(v).toFixed(1),mt+ph+15,km(v),{a:'middle'});

      function line(sl,col,dash,w){ return '<line x1="'+X(0)+'" y1="'+Y(0)+'" x2="'+X(xmax).toFixed(1)+'" y2="'+Y(-a*xmax/10/sl).toFixed(1)+'" stroke="'+col+'" stroke-width="'+w+'"'+(dash?' stroke-dasharray="'+dash+'"':'')+' stroke-linecap="round"/>'; }
      var body=line(n,PURPLE,'9 3 2 3',1.8)+line(2,AMBER,'6 4',1.8)+line(1,GREEN,'',2.4);
      var mx=X(d).toFixed(1);
      body+='<line x1="'+mx+'" y1="'+mt+'" x2="'+mx+'" y2="'+(mt+ph)+'" stroke="#dedede" stroke-width="1" stroke-dasharray="2 3" opacity="0.5"/>';
      body+='<circle cx="'+mx+'" cy="'+Y(lp).toFixed(1)+'" r="4.5" fill="'+PURPLE+'" stroke="#111" stroke-width="1.2"/>';
      body+='<circle cx="'+mx+'" cy="'+Y(lm).toFixed(1)+'" r="4.5" fill="'+AMBER+'" stroke="#111" stroke-width="1.2"/>';
      body+='<circle cx="'+mx+'" cy="'+Y(le).toFixed(1)+'" r="4.5" fill="'+GREEN+'" stroke="#111" stroke-width="1.2"/>';
      var lx=X(d)+6, anc='start'; if(X(d)>W-mr-70){ lx=X(d)-6; anc='end'; }
      body+=txt(lx.toFixed(1),mt+ph-6,km(d)+' km',{fill:LIGHT,a:anc});
      var inr=[], i, e;
      for(i=0;i<EXP.length;i++){ e=EXP[i]; if(e.d<=xmax){ inr.push(e); body+='<circle cx="'+X(e.d).toFixed(1)+'" cy="'+Y(-e.db/10).toFixed(1)+'" r="4" fill="none" stroke="'+LIGHT+'" stroke-width="1.5"/>'; } }
      if(xmax<=1500){
        for(i=0;i<inr.length;i++){ e=inr[i]; var ex_=X(e.d)+7, ey=Y(-e.db/10)-8, an='start'; if(ex_>W-mr-80){ ex_=X(e.d)-7; an='end'; }
          body+=txt(ex_.toFixed(1),ey.toFixed(1),e.t,{fill:LIGHT,a:an}); }
      } else if(inr.length){
        var cx=X(inr[inr.length-1].d), cy=Y(-inr[inr.length-1].db/10)+7, tx=ml+14, ty=mt+ph-34;
        body+='<line x1="'+(tx+4)+'" y1="'+(ty-11)+'" x2="'+cx.toFixed(1)+'" y2="'+cy.toFixed(1)+'" stroke="'+LIGHT+'" stroke-opacity=".5" stroke-width="1"/>';
        body+=txt(tx,ty,'every fiber experiment so far',{fill:LIGHT});
      }
      G.innerHTML=g+
        '<defs><clipPath id="qwc-clip"><rect x="'+ml+'" y="'+(mt-6)+'" width="'+pw+'" height="'+(ph+12)+'"/></clipPath></defs>'+
        '<line x1="'+ml+'" y1="'+(mt+ph)+'" x2="'+(W-mr)+'" y2="'+(mt+ph)+'" stroke="'+AXIS+'" stroke-width="1.5"/>'+
        '<g clip-path="url(#qwc-clip)">'+body+'</g>'+
        txt((ml+pw/2),H-6,'distance (km)',{fs:10.5,a:'middle'})+
        txt(ml-42,(mt+ph/2),'chance per attempt',{fs:10.5,a:'middle',extra:' transform="rotate(-90 '+(ml-42)+' '+(mt+ph/2)+')"'});

      $('qwc-L').textContent=(Lt<100?Lt.toFixed(1):km(Lt))+' dB';
      $('qwc-E').innerHTML=chance(le);
      var lt=-10-le;
      $('qwc-T').innerHTML=tm(lt);
      $('qwc-Ts').innerHTML=(lt>LAGE ? 'about 10<sup>'+(lt-LAGE).toFixed(0)+'</sup> × the age of the universe' : (lt>9.5 ? 'longer than recorded history' : 'single-photon source at 10 GHz'));
      var lb=(le>-6) ? Math.log10(-Math.log2(1-Math.pow(10,le))) : Math.log10(1/Math.LN2)+le;
      $('qwc-B').innerHTML=(lb>-3 ? sig(Math.pow(10,lb),2) : ex(lb));
      $('qwc-M').innerHTML=chance(lm);
      $('qwc-C').innerHTML=(n===1 ? chance(lp) : pct(lp));
      $('qwc-Cs').textContent=(n===1 ? 'one segment: the whole distance' : n+' segments, '+(Lt/n).toFixed(1)+' dB each');
      var note;
      if(n===1){
        $('qwc-H').textContent='none';
        $('qwc-Hs').textContent='nothing to store in a single hop';
        note='straight through '+km(d)+' km of '+a.toFixed(3)+' dB/km fiber, the chance per photon is '+chance(le)+'. a source at ten billion photons a second would see one arrive every <b>'+tm(lt)+'</b>'+(lt>LAGE ? ', about 10<sup>'+(lt-LAGE).toFixed(0)+'</sup> times the age of the universe.' : '.');
      } else {
        var p=Math.pow(10,lp), ll=(lp>-12) ? Math.log10(-Math.log(1-p)) : lp;
        var Ta=(d/n)/V, Hn=harm(n);
        var la=Math.max(0,Math.log10(Hn)-ll);
        var lw=la+Math.log10(Ta), Tc=d/V, lc=Math.log10(Tc);
        var tot=(lw>6) ? lw : Math.log10(Math.pow(10,lw)+Tc);
        $('qwc-H').innerHTML=tm(tot);
        $('qwc-Hs').innerHTML='slowest segment about '+tm(lw)+', then '+tm(lc)+' to cross';
        note='cut into '+n+' segments of '+segLen(d,n)+' km, each segment only has to beat '+(Lt/n).toFixed(1)+' dB, a '+pct(lp)+' chance per attempt. the memories along the way then have to hold for about <b>'+tm(tot)+'</b>: roughly '+tm(lw)+' for the slowest segment to come through, plus '+tm(lc)+' for the result to cross '+km(d)+' km. however finely you chop, that last term stays.';
      }
      $('qwc-note').innerHTML=note;
      var key=null, kk, q;
      for(kk in PRE){ if(PRE.hasOwnProperty(kk)){ q=PRE[kk]; if(q.d===d && Math.abs(q.a-a)<0.0006 && q.n===n) key=kk; } }
      mark(R,'pre',key);
    }
    ds.addEventListener('input',function(){ dist=niceD(s2d(+ds.value)); draw(); });
    as.addEventListener('input',function(){ alpha=+as.value; draw(); });
    ns.addEventListener('input',draw);
    onBtn(R,'pre',function(v){ var q=PRE[v]; dist=q.d; alpha=q.a; ds.value=d2s(q.d); as.value=q.a; ns.value=NSEG.indexOf(q.n); draw(); });
    onResize(draw);
    draw();
  })();
})();
</script>{% endraw %}