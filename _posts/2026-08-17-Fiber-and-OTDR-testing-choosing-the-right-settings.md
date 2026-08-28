---
layout: post
section-type: post
title: "Five kinds of fiber : MMF, SMF, PMF, multicore and hollow core"
category: 'networking'
tags: [ 'networking', 'fiber', 'quantum' ]
---

people say "fiber" the way they say "cable", as if it's one thing. it isn't. the glass in a data center patch panel and the glass in a quantum lab are as different from each other as a garden hose is from a hydraulic line. they both move stuff down a tube, and that's about where the similarity ends.

this post walks through the five families you'll actually meet: **multimode**, **singlemode**, **polarization maintaining**, **multicore**, and **hollow core**. for each one: what the cross section looks like, how light physically gets from one end to the other, what that architecture costs you, and what it's good for.

then, at the end, the part that made me want to write this: what changes when the thing you're sending down the fiber is a single photon.

## the one job

every fiber is doing the same job. keep light travelling in a straight line for a distance where it would otherwise spread out and leave.

there are only two ways anyone has found to do this well.

**index guiding (total internal reflection).** put a high-index core inside a lower-index cladding. light hitting the boundary at a shallow enough angle reflects instead of escaping. four of the five fibers below work this way.

<svg viewBox="0 0 640 218" width="100%" style="max-width:640px;margin:26px auto;display:block" role="img" aria-label="Index guiding: fiber cross section and a ray totally internally reflecting inside the core">
<g font-family="'Open Sans',Helvetica,sans-serif">
<circle cx="84" cy="104" r="68" fill="#2b2b2b" stroke="#5a5a5a" stroke-width="1.5"/>
<circle cx="84" cy="104" r="15" fill="#3f5560" stroke="#7fd1ff" stroke-width="1.5"/>
<text x="84" y="196" text-anchor="middle" font-size="11" fill="#8d8d8d">cross section</text>
<rect x="206" y="36" width="330" height="136" fill="#2b2b2b" stroke="#5a5a5a" stroke-width="1.5"/>
<rect x="206" y="84" width="330" height="40" fill="#3f5560"/>
<line x1="206" y1="84" x2="536" y2="84" stroke="#7fd1ff" stroke-width="1.2"/>
<line x1="206" y1="124" x2="536" y2="124" stroke="#7fd1ff" stroke-width="1.2"/>
<path d="M206 120 L266 88 L326 120 L386 88 L446 120 L506 88 L536 104" fill="none" stroke="#ffd479" stroke-width="2.2"/>
<circle cx="266" cy="88" r="3" fill="#ffd479"/><circle cx="326" cy="120" r="3" fill="#ffd479"/>
<circle cx="386" cy="88" r="3" fill="#ffd479"/><circle cx="446" cy="120" r="3" fill="#ffd479"/>
<text x="216" y="58" font-size="12" fill="#cfcfcf">cladding</text>
<text x="216" y="74" font-size="10.5" fill="#8d8d8d">n = 1.4440</text>
<text x="216" y="152" font-size="12" fill="#cfcfcf">cladding</text>
<text x="548" y="100" font-size="12" fill="#7fd1ff">core</text>
<text x="548" y="116" font-size="10.5" fill="#8d8d8d">n = 1.4492</text>
<text x="371" y="198" text-anchor="middle" font-size="11.5" fill="#8d8d8d">shallow enough angle → it reflects instead of escaping</text>
<text x="371" y="24" text-anchor="middle" font-size="11.5" fill="#8d8d8d">Δn = 0.36 %, and that is all it takes</text>
</g>
</svg>

the index difference is tiny. for standard singlemode it's about 0.36%. a fraction of a percent of germanium doping in the core is what holds the internet together.

**antiresonance.** don't use a core at all. put the light in air and surround it with thin glass membranes tuned so that light hitting them is reflected rather than transmitted. this is hollow core, and it's the odd one out. more on it later.

## modes, and why the whole taxonomy hangs off them

a **mode** is a field pattern that can travel down the fiber without changing shape. the fiber supports a discrete set of them, and how many depends on one number:

```
    V  =  (2 pi a / lambda)  x  NA          NA = sqrt(n_core^2 - n_clad^2)

    V < 2.405   ->   one mode      (singlemode)
    V > 2.405   ->   many modes    (multimode), roughly V^2 / 2 of them
```

`a` is the core radius. and that single number is the fork in the road.

plug in a 50 µm core with NA 0.2 at 850 nm and you get **V ≈ 37**, so several hundred modes. plug in an 8.2 µm core with NA 0.12 at 1550 nm and you get **V ≈ 2.0**, so one.

the same formula tells you the **cutoff wavelength**: shrink lambda far enough and even a "singlemode" fiber stops being singlemode. for G.652 that happens around 1260 nm, which is exactly why the O band starts where it does and why nobody runs singlemode at 1100 nm.

everything below is a consequence of this one number, plus what you do about polarization.

---

# 1. multimode fiber (MMF)

## the architecture

fat core, so light has room to take many paths.

<svg viewBox="0 0 640 214" width="100%" style="max-width:640px;margin:26px auto;display:block" role="img" aria-label="Multimode fiber cross section with a 50 micron core and many ray paths">
<g font-family="'Open Sans',Helvetica,sans-serif">
<circle cx="84" cy="102" r="68" fill="#2b2b2b" stroke="#5a5a5a" stroke-width="1.5"/>
<circle cx="84" cy="102" r="28" fill="#3f5560" stroke="#7fd1ff" stroke-width="1.5"/>
<line x1="56" y1="102" x2="112" y2="102" stroke="#9fe8f5" stroke-width="1" stroke-dasharray="3 3"/>
<text x="84" y="188" text-anchor="middle" font-size="11" fill="#8d8d8d">50 µm core in 125 µm cladding</text>
<rect x="206" y="34" width="414" height="136" fill="#2b2b2b" stroke="#5a5a5a" stroke-width="1.5"/>
<rect x="206" y="62" width="414" height="80" fill="#3f5560"/>
<line x1="206" y1="62" x2="620" y2="62" stroke="#7fd1ff" stroke-width="1.2"/>
<line x1="206" y1="142" x2="620" y2="142" stroke="#7fd1ff" stroke-width="1.2"/>
<line x1="206" y1="102" x2="620" y2="102" stroke="#7fd1ff" stroke-width="2"/>
<path d="M206 102 L288 66 L370 138 L452 66 L534 138 L616 78" fill="none" stroke="#ffd479" stroke-width="2"/>
<path d="M206 102 L330 68 L454 136 L578 72 L620 94" fill="none" stroke="#ff9f6e" stroke-width="2"/>
<text x="216" y="52" font-size="11.5" fill="#cfcfcf">cladding</text>
<text x="216" y="160" font-size="11.5" fill="#cfcfcf">cladding</text>
<text x="413" y="192" text-anchor="middle" font-size="11.5" fill="#8d8d8d">hundreds of modes = hundreds of paths of different length</text>
<text x="413" y="24" text-anchor="middle" font-size="11.5" fill="#8d8d8d">a fat core gives light room to take many routes</text>
</g>
</svg>

the important detail is not the diameter, it's the **index profile**. there are two:

- **step index**, a uniform core index with a sharp jump at the cladding. rays zig-zag. cheap and awful, essentially obsolete for data.
- **graded index**, where the index is highest at the centre and falls off as a parabola toward the cladding. this is what every modern MMF is.

## how light travels

in a step index fiber, a ray bouncing at a steep angle travels a physically longer path than one going straight down the middle. same speed, longer path, later arrival. fire a sharp pulse in and a smeared blob comes out. this is **modal dispersion** and it is brutal. it's what limits step index MMF to a few tens of megabits over any useful distance.

graded index is the fix, and it's elegant. because the index falls off toward the edges, the outer rays spend their time in *faster* glass. they take a longer route at a higher speed. instead of zig-zagging they follow smooth sinusoidal curves, and the two effects very nearly cancel:

<svg viewBox="0 0 640 284" width="100%" style="max-width:640px;margin:26px auto;display:block" role="img" aria-label="Step index versus graded index multimode fiber, comparing ray paths and arrival times">
<g font-family="'Open Sans',Helvetica,sans-serif">
<text x="16" y="18" font-size="12.5" fill="#cfcfcf" font-weight="700">step index</text>
<rect x="16" y="28" width="486" height="76" fill="#2b2b2b" stroke="#5a5a5a" stroke-width="1.2"/>
<rect x="16" y="46" width="486" height="40" fill="#3f5560"/>
<path d="M16 66 L84 48 L152 84 L220 48 L288 84 L356 48 L424 84 L492 54" fill="none" stroke="#ff9f6e" stroke-width="2"/>
<line x1="16" y1="66" x2="502" y2="66" stroke="#7fd1ff" stroke-width="2"/>
<text x="512" y="58" font-size="11" fill="#7fd1ff">on time</text>
<text x="512" y="76" font-size="11" fill="#ff9f6e">late</text>
<text x="320" y="122" font-size="11.5" fill="#8d8d8d" text-anchor="middle">uniform core index: longer path, same speed, so it arrives late</text>
<text x="16" y="166" font-size="12.5" fill="#cfcfcf" font-weight="700">graded index</text>
<rect x="16" y="176" width="486" height="76" fill="#2b2b2b" stroke="#5a5a5a" stroke-width="1.2"/>
<rect x="16" y="186" width="486" height="56" fill="#2f3a41"/>
<rect x="16" y="194" width="486" height="40" fill="#354349"/>
<rect x="16" y="202" width="486" height="24" fill="#3f5560"/>
<path d="M16 214 Q65 186 114 214 Q163 242 212 214 Q261 186 310 214 Q359 242 408 214 Q457 186 492 210" fill="none" stroke="#7fdb9b" stroke-width="2"/>
<line x1="16" y1="214" x2="502" y2="214" stroke="#7fd1ff" stroke-width="2"/>
<text x="512" y="212" font-size="11" fill="#7fdb9b">≈ on time</text>
<text x="320" y="270" font-size="11.5" fill="#8d8d8d" text-anchor="middle">brightest = highest index = slowest glass, so the outer rays run faster and catch up</text>
</g>
</svg>

"very nearly" is doing work in that sentence. the cancellation is never perfect, and what's left over is the **modal bandwidth**, quoted in MHz·km. that number is most of what a multimode datasheet is telling you.

## the grades

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Grade</th><th>Core</th><th>Jacket</th><th>Modal BW @ 850 nm</th><th>Source</th><th>10G SR</th><th>40G SR4</th><th>100G SR4</th></tr>
</thead>
<tbody>
<tr><td>OM1</td><td>62.5 µm</td><td>orange</td><td>200 MHz·km</td><td>LED</td><td>33 m</td><td>-</td><td>-</td></tr>
<tr><td>OM2</td><td>50 µm</td><td>orange</td><td>500 MHz·km</td><td>LED</td><td>82 m</td><td>-</td><td>-</td></tr>
<tr><td>OM3</td><td>50 µm</td><td>aqua</td><td>2000 MHz·km</td><td>VCSEL</td><td>300 m</td><td>100 m</td><td>70 m</td></tr>
<tr><td>OM4</td><td>50 µm</td><td>violet</td><td>4700 MHz·km</td><td>VCSEL</td><td>400 m</td><td>150 m</td><td>100 m</td></tr>
<tr><td>OM5</td><td>50 µm</td><td>lime</td><td>4700 MHz·km (+ 2470 @ 953 nm)</td><td>VCSEL</td><td>400 m</td><td>150 m</td><td>100 m</td></tr>
</tbody>
</table>
</div>

two things about that table. the bandwidth column is **effective modal bandwidth**, measured with a laser launch. you'll also see overfilled launch numbers quoted (1500 and 3500 for OM3 and OM4), which are measured with an LED and are the wrong number for a VCSEL link. and the 100G column is SR4; the older SR10 goes further because it spreads the rate over ten lanes.

OM3 onward are **laser optimized**: the index profile is manufactured to match how a VCSEL actually fills the fiber, which is not how an LED fills it. this is why you can't just swap an OM1 patch cord into a 10G link and hope.

OM5 adds characterized bandwidth at 953 nm so you can run **SWDM**, four wavelengths between 850 and 940 nm on one pair. in practice it never took off the way the marketing suggested, and most people building new go OM4 or jump to singlemode.

## the deal

you're buying cheap transceivers and paying in distance. a big core is easy to couple into, so you can use a VCSEL, a laser that costs a few dollars, runs at low power and is easy to align. a singlemode transceiver needs a precisely aligned narrow-linewidth source and costs multiples of that.

that trade is why multimode still exists at all.

## use it for

- **inside a building or a data center hall**, under ~150 m at high rates
- **server to top-of-rack, and rack to spine**, where you're buying hundreds of transceivers and the price difference is the budget
- anything where the link count is large and the distance is small

## don't use it for

- anything you'll want to upgrade later. multimode distance shrinks every time the rate goes up. a run that did 10G at 400 m does 100G at 100 m.
- long campus runs. the crossover point where singlemode gets cheaper overall keeps moving toward shorter distances.
- **anything to do with quantum.** more on that below, but the short version is that a fiber with hundreds of modes is a machine for scrambling exactly the properties you were trying to preserve.

---

# 2. singlemode fiber (SMF)

## the architecture

shrink the core until only one spatial mode fits.

<svg viewBox="0 0 640 226" width="100%" style="max-width:640px;margin:26px auto;display:block" role="img" aria-label="Singlemode fiber cross section with an 8.2 micron core, and the mode field profile spilling into the cladding">
<g font-family="'Open Sans',Helvetica,sans-serif">
<defs>
<radialGradient id="mfd" cx="50%" cy="50%" r="50%">
<stop offset="0%" stop-color="#7fd1ff" stop-opacity="0.85"/>
<stop offset="45%" stop-color="#4a7f9c" stop-opacity="0.5"/>
<stop offset="100%" stop-color="#3f5560" stop-opacity="0"/>
</radialGradient>
</defs>
<circle cx="84" cy="102" r="68" fill="#2b2b2b" stroke="#5a5a5a" stroke-width="1.5"/>
<circle cx="84" cy="102" r="17" fill="url(#mfd)"/>
<circle cx="84" cy="102" r="6" fill="none" stroke="#7fd1ff" stroke-width="1.4"/>
<text x="84" y="188" text-anchor="middle" font-size="11" fill="#8d8d8d">8.2 µm core in 125 µm cladding</text>
<rect x="212" y="34" width="200" height="136" fill="#2b2b2b" stroke="#5a5a5a" stroke-width="1.2"/>
<line x1="212" y1="86" x2="412" y2="86" stroke="#7fd1ff" stroke-width="1" stroke-dasharray="4 4"/>
<line x1="212" y1="118" x2="412" y2="118" stroke="#7fd1ff" stroke-width="1" stroke-dasharray="4 4"/>
<path d="M224 166 C242 164 256 150 268 128 C282 100 288 50 312 50 C336 50 342 100 356 128 C368 150 382 164 400 166 Z" fill="url(#mfd)" stroke="#7fd1ff" stroke-width="1.6"/>
<text x="312" y="28" text-anchor="middle" font-size="11" fill="#7fd1ff">LP01 mode</text>
<line x1="268" y1="150" x2="356" y2="150" stroke="#9fe8f5" stroke-width="1"/>
<line x1="268" y1="145" x2="268" y2="155" stroke="#9fe8f5" stroke-width="1"/>
<line x1="356" y1="145" x2="356" y2="155" stroke="#9fe8f5" stroke-width="1"/>
<text x="312" y="144" text-anchor="middle" font-size="10.5" fill="#9fe8f5">MFD</text>
<text x="222" y="80" font-size="9.5" fill="#8d8d8d">core</text>
<text x="222" y="112" font-size="9.5" fill="#8d8d8d">edge</text>
<text x="440" y="72" font-size="12" fill="#cfcfcf">the mode is wider than the core</text>
<text x="440" y="94" font-size="11" fill="#8d8d8d">a real fraction of the power rides</text>
<text x="440" y="110" font-size="11" fill="#8d8d8d">in the cladding, which is why the</text>
<text x="440" y="126" font-size="11" fill="#8d8d8d">spec is mode field diameter and</text>
<text x="440" y="142" font-size="11" fill="#8d8d8d">not core diameter</text>
<text x="320" y="206" text-anchor="middle" font-size="11.5" fill="#8d8d8d">mismatch two of these at a splice → loss, or a phantom gainer on the OTDR</text>
</g>
</svg>

the core is about 8.2 µm across, and the mode field diameter (the number that's actually on the spec sheet) is 9.2 µm at 1310 nm and about 10.4 µm at 1550. cladding is 125 µm, the same outside diameter as multimode, so connectors and splicers are common hardware.

## how light travels

there's no useful ray picture here, because the core is only about six wavelengths across. what propagates is a single field distribution, the **LP01 mode**, and it isn't confined to the core, and a meaningful fraction of the power travels in the cladding. that's why the spec is *mode field diameter* and not core diameter, and why splicing two fibers with mismatched MFD produces loss and phantom gainers on an OTDR trace.

with modal dispersion gone, three things are left to limit you:

- **attenuation**. 0.32-0.35 dB/km at 1310, 0.17-0.20 dB/km at 1550. ultra low loss pure-silica-core fiber reaches **0.14 dB/km**, which is roughly the floor for glass; below that you're fighting Rayleigh scattering off the atomic structure of silica itself.
- **chromatic dispersion**, which is different wavelengths travelling at slightly different speeds. about **17 ps/nm/km** at 1550 in G.652. this is what limits high rate links, and it's correctable in the DSP of a coherent transceiver.
- **polarization mode dispersion (PMD)**, which gets the next section to itself, because this is where PM fiber comes from.

## the grades

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>ITU-T</th><th>What it is</th><th>Where it goes</th></tr>
</thead>
<tbody>
<tr><td>G.652.D</td><td>the standard. low water peak, usable 1260-1625 nm</td><td>everything, by default</td></tr>
<tr><td>G.657.A1 / A2</td><td>bend insensitive, 10 mm / 7.5 mm bend radius, still G.652 compatible</td><td>FTTH drops, risers, patch panels, anywhere someone will kink it</td></tr>
<tr><td>G.657.B3</td><td>extremely bend insensitive, 5 mm radius, not fully G.652 compatible</td><td>inside apartments, around door frames</td></tr>
<tr><td>G.654.E</td><td>large effective area, pure silica core, ~0.15-0.17 dB/km</td><td>terrestrial long haul, high power coherent</td></tr>
<tr><td>G.655</td><td>non-zero dispersion shifted, low but non-zero D at 1550</td><td>legacy DWDM builds, don't specify it new</td></tr>
</tbody>
</table>
</div>

**if you're not sure, it's G.652.D.** if it's a drop cable or it lives in a wall, it's G.657.A2.

## the deal

you're paying for transceivers and getting distance and headroom. the fiber itself is cheap, often cheaper than multimode by the metre. the cost is at the ends.

but the fiber is also *future proof in a way multimode never is*: the same G.652 in the ground carried 2.5G, then 10G, then 100G coherent, then 400G, and will carry 800G. the glass didn't change. only the boxes did.

## use it for

- **anything outside a building.** access, metro, long haul, subsea, FTTH.
- **any run over ~150 m**, and increasingly any run at all in new data centers.
- **anything you want to wavelength-multiplex.** the C band gives you ~80 channels on one fiber pair.
- the default for quantum links over deployed infrastructure, because it's what's already in the ground.

---

# 3. polarization maintaining fiber (PMF)

## the problem it solves

when people say "singlemode carries one mode" they are leaving something out. it carries one *spatial* mode, but that mode has **two polarization states**. in a perfect, perfectly circular, perfectly unstressed fiber those two states travel at exactly the same speed and nothing happens.

no real fiber is like that. the core is slightly elliptical, the cable is bent, it's squeezed in a duct, the sun heats one side of an aerial span. every one of those imposes a small, random, *time-varying* birefringence. the two polarization components pick up random relative phase and exchange energy as they go.

the consequences:

- **PMD**. the two components arrive at different times, smearing the pulse. small, but it accumulates as sqrt(distance) and it's a real limit on old fiber at high rates.
- the output polarization is unpredictable and drifts. point a linearly polarized laser into a spool of SMF and what comes out is some arbitrary elliptical state that changes when someone walks past the rack.

for classical intensity-modulated links, nobody cares, because the receiver measures power. for coherent links, the DSP tracks and undoes it. for anything that encodes information in polarization itself, this is fatal.

## the architecture

you can't get rid of birefringence. so you overwhelm it.

<svg viewBox="0 0 640 250" width="100%" style="max-width:640px;margin:26px auto;display:block" role="img" aria-label="Three polarization maintaining fiber designs: PANDA, bow-tie and elliptical clad, with fast and slow axes marked">
<g font-family="'Open Sans',Helvetica,sans-serif">
<text x="106" y="20" text-anchor="middle" font-size="13" fill="#cfcfcf" font-weight="700">PANDA</text>
<circle cx="106" cy="112" r="72" fill="#2b2b2b" stroke="#5a5a5a" stroke-width="1.5"/>
<circle cx="48" cy="112" r="26" fill="#8a6fd6" opacity="0.75"/>
<circle cx="164" cy="112" r="26" fill="#8a6fd6" opacity="0.75"/>
<circle cx="106" cy="112" r="9" fill="#3f5560" stroke="#7fd1ff" stroke-width="1.5"/>
<line x1="26" y1="112" x2="186" y2="112" stroke="#ffd479" stroke-width="1.2" stroke-dasharray="5 4"/>
<line x1="106" y1="32" x2="106" y2="192" stroke="#7fdb9b" stroke-width="1.2" stroke-dasharray="5 4"/>
<text x="196" y="110" font-size="10.5" fill="#ffd479">slow</text>
<text x="106" y="206" text-anchor="middle" font-size="10.5" fill="#7fdb9b">fast</text>
<text x="106" y="232" text-anchor="middle" font-size="11" fill="#8d8d8d">boron-doped stress rods</text>
<text x="320" y="20" text-anchor="middle" font-size="13" fill="#cfcfcf" font-weight="700">bow-tie</text>
<circle cx="320" cy="112" r="72" fill="#2b2b2b" stroke="#5a5a5a" stroke-width="1.5"/>
<path d="M320 112 L252 74 L252 150 Z" fill="#8a6fd6" opacity="0.75"/>
<path d="M320 112 L388 74 L388 150 Z" fill="#8a6fd6" opacity="0.75"/>
<circle cx="320" cy="112" r="9" fill="#3f5560" stroke="#7fd1ff" stroke-width="1.5"/>
<text x="320" y="232" text-anchor="middle" font-size="11" fill="#8d8d8d">stress reaches closer, so more of it</text>
<text x="534" y="20" text-anchor="middle" font-size="13" fill="#cfcfcf" font-weight="700">elliptical clad</text>
<circle cx="534" cy="112" r="72" fill="#2b2b2b" stroke="#5a5a5a" stroke-width="1.5"/>
<ellipse cx="534" cy="112" rx="54" ry="26" fill="#8a6fd6" opacity="0.7"/>
<circle cx="534" cy="112" r="9" fill="#3f5560" stroke="#7fd1ff" stroke-width="1.5"/>
<text x="534" y="232" text-anchor="middle" font-size="11" fill="#8d8d8d">asymmetric stress layer</text>
</g>
</svg>

two rods of boron-doped glass on either side of the core. boron glass has a different thermal expansion coefficient from silica, so when the preform cools from the draw, the rods shrink more and squeeze the core along one axis. that permanent stress makes the fiber **strongly birefringent by design**: a slow axis and a fast axis, with a large, fixed index difference between them.

**bow-tie** shapes the stress zones to reach closer to the core, giving stronger birefringence. **elliptical clad** uses an asymmetric stress layer. PANDA is the one you'll actually buy.

## how light travels

launch light polarized along one of the two axes (you align the connector key to it) and it stays there.

the reason is a coupling argument, not a confinement one. energy transfers between two modes efficiently only when they're phase matched. the built-in birefringence makes the two polarization modes so mismatched in propagation constant that a random perturbation can't push energy from one to the other before the phase relationship has scrambled. the built-in birefringence beats the random birefringence, so the random birefringence stops mattering.

the number that describes this is the **beat length**: the distance over which the two axes accumulate 2π of relative phase.

```
    L_beat  =  lambda / delta_n

    PANDA at 1550 nm:  L_beat = 3 - 5 mm   ->   delta_n ~ 3-5 x 10^-4
```

compare that to the ~10⁻⁷ of accidental birefringence in normal fiber, where the beat length is metres and moves around. you've made the deliberate effect thousands of times bigger than the accidental one. the rest of the fiber is just packaging around that fact.

launch off-axis, though, and you've excited both modes, and now that 3 mm beat length works against you: the output polarization spins wildly with temperature and wavelength. PM fiber is unforgiving about alignment. that's not a defect, it's the same property viewed from the wrong side.

## the cost

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Parameter</th><th>PANDA PM 1550</th><th>Standard SMF</th></tr>
</thead>
<tbody>
<tr><td>Attenuation</td><td>&le; 0.5 dB/km</td><td>0.17 - 0.20 dB/km</td></tr>
<tr><td>Beat length</td><td>3.0 - 5.0 mm</td><td>metres, and random</td></tr>
<tr><td>Crosstalk</td><td>-30 dB at 100 m</td><td>n/a</td></tr>
<tr><td>Mode field diameter</td><td>10.5 µm</td><td>10.4 µm at 1550</td></tr>
<tr><td>Price</td><td>10 - 50x</td><td>1x</td></tr>
<tr><td>Connectorization</td><td>keyed, rotationally aligned</td><td>any</td></tr>
</tbody>
</table>
</div>

that **-30 dB per 100 m** figure is the one to internalize. polarization extinction ratio degrades with length, and it degrades faster with temperature swings and bends. PM fiber is a **short-run component**, not a transmission medium. a kilometre of it is already a stretch, and there is essentially no deployed PM plant anywhere in the world.

every connector has to be rotationally keyed, every splice has to be angle-aligned, and every component in the chain has to be PM too, or you've thrown the property away at the first junction.

## use it for

- **inside instruments.** modulator and laser pigtails, interferometer arms, fiber optic gyroscopes.
- **coherent detection front ends**, where the local oscillator has to arrive in a known state.
- **fiber sensing**, particularly interferometric and Sagnac-based sensors.
- **anything nonlinear**, like frequency doubling and parametric sources, because the efficiency depends on polarization overlap.
- quantum optics benches and short quantum links. this is the big one, and it gets its own section below.

## don't use it for

- **spans.** the loss and the PER degradation make it a losing proposition past a few hundred metres.
- anything where you can instead fix polarization at the receiver. an active polarization controller with feedback is usually cheaper and better than a PM span.

---

# 4. multicore fiber (MCF)

## the architecture

several separate cores inside one cladding.

<svg viewBox="0 0 640 268" width="100%" style="max-width:640px;margin:26px auto;display:block" role="img" aria-label="Four core and seven core multicore fiber layouts, showing core pitch, trench assistance and adjacent versus diagonal crosstalk">
<g font-family="'Open Sans',Helvetica,sans-serif">
<text x="120" y="20" text-anchor="middle" font-size="13" fill="#cfcfcf" font-weight="700">4-core</text>
<circle cx="120" cy="122" r="86" fill="#2b2b2b" stroke="#5a5a5a" stroke-width="1.5"/>
<g>
<circle cx="86" cy="88" r="19" fill="none" stroke="#4a4a4a" stroke-width="7"/>
<circle cx="154" cy="88" r="19" fill="none" stroke="#4a4a4a" stroke-width="7"/>
<circle cx="86" cy="156" r="19" fill="none" stroke="#4a4a4a" stroke-width="7"/>
<circle cx="154" cy="156" r="19" fill="none" stroke="#4a4a4a" stroke-width="7"/>
</g>
<circle cx="86" cy="88" r="9" fill="#3f5560" stroke="#7fd1ff" stroke-width="1.5"/>
<circle cx="154" cy="88" r="9" fill="#3f5560" stroke="#7fd1ff" stroke-width="1.5"/>
<circle cx="86" cy="156" r="9" fill="#3f5560" stroke="#7fd1ff" stroke-width="1.5"/>
<circle cx="154" cy="156" r="9" fill="#3f5560" stroke="#7fd1ff" stroke-width="1.5"/>
<path d="M96 88 L144 88" stroke="#ff8f7a" stroke-width="2"/>
<path d="M94 96 L146 148" stroke="#7fdb9b" stroke-width="2" stroke-dasharray="4 4"/>
<text x="120" y="62" text-anchor="middle" font-size="10.5" fill="#ff8f7a">adjacent: worst</text>
<text x="120" y="230" text-anchor="middle" font-size="10.5" fill="#7fdb9b">diagonal: the quiet seat</text>
<text x="120" y="250" text-anchor="middle" font-size="11" fill="#8d8d8d">pitch ~50 µm, 125 µm cladding</text>
<text x="386" y="20" text-anchor="middle" font-size="13" fill="#cfcfcf" font-weight="700">7-core</text>
<circle cx="386" cy="122" r="86" fill="#2b2b2b" stroke="#5a5a5a" stroke-width="1.5"/>
<g fill="#3f5560" stroke="#7fd1ff" stroke-width="1.5">
<circle cx="386" cy="122" r="9"/><circle cx="386" cy="76" r="9"/><circle cx="386" cy="168" r="9"/>
<circle cx="346" cy="99" r="9"/><circle cx="346" cy="145" r="9"/>
<circle cx="426" cy="99" r="9"/><circle cx="426" cy="145" r="9"/>
</g>
<text x="386" y="250" text-anchor="middle" font-size="11" fill="#8d8d8d">pitch 35 - 42 µm, higher density, more crosstalk</text>
<g font-size="11.5">
<circle cx="500" cy="80" r="9" fill="none" stroke="#4a4a4a" stroke-width="5"/>
<circle cx="500" cy="80" r="4.5" fill="#3f5560" stroke="#7fd1ff" stroke-width="1.2"/>
<text x="516" y="84" font-size="11.5" fill="#cfcfcf">trench assistance</text>
<text x="490" y="110" font-size="10.5" fill="#8d8d8d">a moat of down-doped</text>
<text x="490" y="126" font-size="10.5" fill="#8d8d8d">glass round each core,</text>
<text x="490" y="142" font-size="10.5" fill="#8d8d8d">cutting the evanescent</text>
<text x="490" y="158" font-size="10.5" fill="#8d8d8d">tail before it reaches</text>
<text x="490" y="174" font-size="10.5" fill="#8d8d8d">the neighbour</text>
</g>
</g>
</svg>

the pitch, meaning the centre to centre spacing, is the hard part of the design. cores too close and they talk to each other. cores too far and you can't fit enough in a standard 125 µm cladding, and if you go to a 200 µm cladding you've given up on existing cable, connector and splicing infrastructure.

two families:

- **uncoupled (weakly coupled) MCF**. cores are deliberately isolated. each one is an independent singlemode channel. this is what's being standardized and deployed.
- **coupled core MCF**. cores are close enough to couple on purpose, and the receiver untangles them with MIMO DSP, the same way wireless does. higher density, much harder receiver. research and specialist use.

the trick that makes uncoupled MCF work is **trench assistance**: a ring of down-doped, low index glass around each core that acts as a moat, cutting the evanescent tail before it reaches the neighbour.

## how light travels

in each core, exactly what happens in singlemode fiber. the interesting physics is what happens *between* cores.

confinement is never total. the mode's field decays exponentially into the cladding but never reaches zero. if a neighbouring core is close enough to sit in that tail, and the two cores are phase matched, power transfers. this is **inter-core crosstalk**, and unlike most impairments it grows *linearly with length* and is very sensitive to bending, because bending shifts the effective indices and can accidentally phase-match two cores that were designed not to match.

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Design</th><th>Typical crosstalk</th></tr>
</thead>
<tbody>
<tr><td>Standard uncoupled 4-core, 125 µm</td><td>-30 to -40 dB accumulated over 100 km</td></tr>
<tr><td>Submarine grade, larger pitch</td><td>coupling below -70 dB/km, so ~-50 dB over 100 km</td></tr>
<tr><td>Adjacent cores</td><td>the worst pair, always</td></tr>
<tr><td>Diagonal cores (4-core square)</td><td>often 10s of dB better than adjacent</td></tr>
</tbody>
</table>
</div>

that last row is not trivia. which core you put a sensitive signal in is a design decision, and the diagonal is the quiet seat.

## the cost

the fiber is the easy part. everything around it is hard:

- **fan-in / fan-out (FIFO)** devices to break the cores out into ordinary singlemode pigtails, because every test set and every transceiver on earth is single-core. you have to match FIFO vendors at both ends so the core mapping agrees.
- **rotational alignment.** splicing an MCF means aligning it to better than about 1°, which needs a splicer that can rotate and a fiber with a physical marker (a notch or a stripe) defining core zero.
- **testing.** certifying a 4-core connector properly means measuring every core pair combination. IEC 61300-3-55 defines how, and the exhaustive method is over two thousand measurements per connector.

standardization is live rather than settled: ITU-T G Supplement 87 covers space division multiplexing fibers, and IEC TC86 is working the MPO mechanical interfaces.

## use it for

- **capacity per cross section.** this is the real argument. a submarine cable's capacity is limited by the electrical power you can push down it and by the physical diameter of the cable, not by how much glass you can afford. four cores in one 125 µm strand is four times the capacity in the same space, sharing one amplifier pump.
- **subsea and repeatered systems**, where multicore amplifiers let you pump all cores together.
- **AI and data center interconnect**, where the bottleneck is increasingly the number of fibers you can physically terminate on a faceplate.
- **quantum-classical coexistence.** see below. spatial isolation turns out to solve a problem that wavelength isolation can't.

---

# 5. hollow core fiber (HCF)

## the architecture

get rid of the glass core entirely.

<svg viewBox="0 0 640 314" width="100%" style="max-width:640px;margin:26px auto;display:block" role="img" aria-label="Hollow core nested antiresonant nodeless fiber cross section, showing the air core surrounded by nested glass capillaries">
<defs>
<radialGradient id="airglow" cx="50%" cy="50%" r="50%">
<stop offset="0%" stop-color="#0e3a44"/><stop offset="100%" stop-color="#0a2028"/>
</radialGradient>
</defs>
<g font-family="'Open Sans',Helvetica,sans-serif">
<circle cx="185" cy="150" r="128" fill="#2b2b2b" stroke="#5a5a5a" stroke-width="1.5"/>
<circle cx="185" cy="150" r="62" fill="url(#airglow)"/>
<g fill="none" stroke="#9fe8f5" stroke-width="2.4">
<circle cx="185" cy="46" r="42"/><circle cx="275" cy="98" r="42"/>
<circle cx="275" cy="202" r="42"/><circle cx="185" cy="254" r="42"/>
<circle cx="95" cy="202" r="42"/><circle cx="95" cy="98" r="42"/>
</g>
<g fill="none" stroke="#4fb3c7" stroke-width="1.8">
<circle cx="185" cy="60" r="28"/><circle cx="263" cy="105" r="28"/>
<circle cx="263" cy="195" r="28"/><circle cx="185" cy="240" r="28"/>
<circle cx="107" cy="195" r="28"/><circle cx="107" cy="105" r="28"/>
</g>
<text x="185" y="145" text-anchor="middle" font-size="17" font-weight="700" fill="#7fd1ff">AIR</text>
<text x="185" y="166" text-anchor="middle" font-size="11" fill="#6f9fb0">n ≈ 1.0003</text>
<g font-size="12.5" fill="#cfcfcf">
<line x1="330" y1="66" x2="300" y2="86" stroke="#666"/>
<text x="338" y="62">outer capillary</text>
<text x="338" y="80" font-size="11" fill="#8d8d8d">under a micron of glass</text>
<line x1="330" y1="150" x2="291" y2="150" stroke="#666"/>
<text x="338" y="146">nested capillary</text>
<text x="338" y="164" font-size="11" fill="#8d8d8d">suppresses leakage to the cladding</text>
<line x1="330" y1="236" x2="243" y2="215" stroke="#666"/>
<text x="338" y="232">nodeless</text>
<text x="338" y="250" font-size="11" fill="#8d8d8d">they never touch, and a contact point</text>
<text x="338" y="266" font-size="11" fill="#8d8d8d">is a resonance, and a resonance leaks</text>
</g>
<text x="185" y="296" text-anchor="middle" font-size="11.5" fill="#8d8d8d">~30 µm air core in a 125 µm cladding</text>
</g>
</svg>

a ring of thin glass capillaries, each with a smaller capillary nested inside it, surrounding an air hole roughly 30 µm across. the membranes are under a micron thick. **nodeless** means the capillaries don't touch each other, because contact points create resonances that leak light. so the geometry is designed to avoid them. **DNANF** adds a second nested layer.

## how light travels

not by total internal reflection. air has a *lower* index than glass, so TIR is running the wrong way and should push the light out.

what confines it is **antiresonance**. each thin glass membrane behaves like a Fabry-Pérot etalon. at wavelengths where the membrane thickness is resonant, light passes straight through and leaks away. at wavelengths *between* those resonances, the membrane reflects, and light stays in the core.

```
    leaky (resonant) at:   lambda_m = (2 t / m) x sqrt(n_glass^2 - 1)

    t = membrane thickness,  m = 1, 2, 3 ...
```

the transmission windows are the gaps between those resonances. this has a consequence that matters enormously: the guiding wavelength is set by one manufacturing dimension. want a fiber that guides at 780 nm instead of 1550 nm? change the membrane thickness. you cannot do that with silica fiber, where the loss versus wavelength curve is a property of the material and you're stuck with it.

over 99.9% of the power ends up in air. everything downstream follows from that.

## what you get

**latency.** light in glass is slowed by the group index.

```
    silica,  n_g = 1.468   ->   4.90 us/km
    air,     n_g = 1.0003  ->   3.34 us/km
    -----------------------------------------
    saving                      1.55 us/km    (~32% lower latency,
                                               or ~47% faster, same fact)
```

**nonlinearity, essentially gone.** the Kerr effect happens in glass. no glass, no Kerr. you can launch far more power without four-wave mixing and self-phase modulation eating the signal.

**scattering, mostly gone.** Rayleigh scattering is a property of glass density fluctuations, and Brillouin and Raman scattering need glass to scatter off. this one is the sleeper. it turns out to matter more for quantum than the latency does.

**dispersion, very low and flat**, because it isn't material dispersion, it's waveguide dispersion in air.

**loss.** this got interesting recently. hollow core was a curiosity at 2.5 dB/km for years, which is fine for a 40 km trading route and useless for anything else. a DNANF from the Southampton group and Microsoft has now measured **0.091 dB/km at 1550 nm**, held across 18 THz of bandwidth. that is below the 0.14 dB/km practical floor of silica, the one set by Rayleigh scattering off the glass itself. it is the first time any fiber has beaten glass at its own game. field builds are in the hundreds to low thousands of kilometres, and the fiber you can actually buy is still some way behind the record, but the direction of travel is clear.

## what it costs

- **splicing.** joining an air core to anything is genuinely hard, because the arc that fuses glass also collapses the capillary structure. splice losses run **0.3-0.6 dB** against under 0.05 dB for SMF. on a long link the splices can dominate the loss budget, which is a strange place to be.
- **bending.** antiresonance is a resonant condition, so bending it doesn't degrade gracefully the way index guiding does. it shifts the guiding window. modern NANFs are much better than early designs, but don't assume G.657 habits transfer.
- **connectorization and test.** the ecosystem is young. your OTDR's IOR setting is about 1.0003, which is going to feel wrong the first time you type it.
- **cost and availability.** manufacturing is ramping, not ramped.

## use it for

- **latency-critical routes.** high frequency trading was the first real market, and 1.55 µs/km is worth actual money on an exchange route.
- **data center and AI interconnect**, where microseconds of tail latency across a collective operation multiply up.
- **high power delivery**, industrial and medical, where you'd burn a glass core.
- **wavelengths silica handles badly.** mid-IR, or the near-IR band where quantum sources live.
- **quantum links**, for reasons that are not the obvious ones.

---

# putting the five side by side

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th></th><th>MMF</th><th>SMF</th><th>PMF</th><th>MCF</th><th>HCF</th></tr>
</thead>
<tbody>
<tr><td>Core</td><td>50 / 62.5 µm</td><td>~9 µm</td><td>~9 µm + stress rods</td><td>N x ~9 µm</td><td>~30 µm of air</td></tr>
<tr><td>Guidance</td><td>graded index TIR</td><td>TIR</td><td>TIR + birefringence</td><td>TIR per core</td><td>antiresonance</td></tr>
<tr><td>Loss @ 1550</td><td>n/a (850/1300 nm fiber)</td><td>0.17 - 0.20 dB/km</td><td>&le; 0.5 dB/km</td><td>0.16 - 0.20 dB/km</td><td>0.09 - 1 dB/km</td></tr>
<tr><td>Latency</td><td>4.9 µs/km</td><td>4.9 µs/km</td><td>4.9 µs/km</td><td>4.9 µs/km</td><td>3.34 µs/km</td></tr>
<tr><td>Reach</td><td>&lt; 400 m</td><td>1000s of km</td><td>&lt; 1 km practical</td><td>1000s of km</td><td>10s to 100s of km</td></tr>
<tr><td>Preserves polarization</td><td>no</td><td>no</td><td>yes</td><td>no</td><td>partly (low birefringence)</td></tr>
<tr><td>Nonlinearity</td><td>high</td><td>moderate</td><td>moderate</td><td>moderate</td><td>~1000x lower</td></tr>
<tr><td>Ecosystem</td><td>mature, cheap</td><td>mature, cheap</td><td>niche, expensive</td><td>emerging</td><td>early</td></tr>
<tr><td>Killer feature</td><td>cheap transceivers</td><td>universal</td><td>fixed polarization</td><td>capacity per mm²</td><td>no glass in the path</td></tr>
</tbody>
</table>
</div>

## which technology wants which fiber

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Application</th><th>Fiber</th><th>Because</th></tr>
</thead>
<tbody>
<tr><td>Server to top-of-rack</td><td>OM4</td><td>hundreds of links, tens of metres, transceiver price dominates</td></tr>
<tr><td>Data center spine, &gt; 100 m</td><td>G.652.D</td><td>multimode has run out of distance at 400G</td></tr>
<tr><td>AI cluster scale-out</td><td>G.652.D, MCF and HCF emerging</td><td>faceplate density and tail latency</td></tr>
<tr><td>FTTH drop</td><td>G.657.A2</td><td>installers will bend it, and they will not apologise</td></tr>
<tr><td>Metro / long haul</td><td>G.652.D, G.654.E over ~80 km</td><td>loss and effective area for coherent</td></tr>
<tr><td>Subsea</td><td>G.654 pure silica, MCF arriving</td><td>power limited, so capacity per unit of pump power wins</td></tr>
<tr><td>High frequency trading</td><td>HCF</td><td>1.55 µs/km is worth money</td></tr>
<tr><td>5G fronthaul</td><td>G.652.D / G.657</td><td>tight latency budget, tight ducts</td></tr>
<tr><td>Distributed sensing (DAS/DTS)</td><td>G.652.D</td><td>you want the Rayleigh and Brillouin scattering, it's the signal</td></tr>
<tr><td>Interferometric sensing, gyroscopes</td><td>PMF</td><td>the measurement is a phase, so polarization must be fixed</td></tr>
<tr><td>Coherent transceiver internals</td><td>PMF</td><td>the local oscillator has to arrive in a known state</td></tr>
<tr><td>Laser cutting, medical delivery</td><td>HCF or large core MMF</td><td>power levels that would destroy a small glass core</td></tr>
</tbody>
</table>
</div>

---

# and now, quantum networks

this is where the choice stops being an optimization and starts being a constraint.

## why quantum is a different problem

four things break at once.

**you cannot amplify.** the no-cloning theorem isn't an engineering limitation you'll route around next year. an EDFA works by stimulated emission, and copying an unknown quantum state perfectly is forbidden. what a real amplifier does instead is add spontaneous emission noise, which is fine when you have 10⁶ photons per bit and catastrophic when you have one. so there is no repeater in the classical sense, and loss is not a budget item you top up. it's an exponential.

```
    eta  =  10^( -alpha L / 10 )

    SMF at 0.2 dB/km,  100 km  ->  20 dB  ->  1 photon in 100 survives
    HCF at 0.09 dB/km, 100 km  ->   9 dB  ->  13 photons in 100 survive
```

a 0.11 dB/km difference is a 13x difference in key rate at 100 km, which is why people who don't care about latency at all still care about hollow core.

noise photons are not attenuated with your signal, they're generated along the way. put a classical DWDM channel at +10 dBm in the same fiber as your quantum channel and spontaneous Raman scattering from the glass sprays photons across a hundred nanometres of spectrum. your detector cannot tell those from signal. a classical link with an OSNR problem loses a bit of margin; a quantum link with a Raman problem stops working entirely.

**the encoding has to survive.** you're not sending intensity. depending on the protocol you're sending polarization states, time bins, or relative phase, and the fiber has to leave those alone or you have to actively undo what it did.

**timing is a resource.** entanglement swapping, memory-based repeaters, and MDI protocols all need photons from different places to arrive within a coherence window. propagation delay isn't just latency here, it's the rate at which you can attempt an operation before a quantum memory decoheres.

## MMF : basically no

multimode is disqualified almost by definition. hundreds of modes means hundreds of paths with different delays and different polarization evolutions, so the spatial, temporal and polarization structure of a single photon comes out scrambled. you also can't couple it efficiently into a superconducting nanowire detector, which has a small active area matched to singlemode.

the exceptions are all short and all inside a box: collecting fluorescence from a trapped ion or an atom cloud, free-space link receivers where you're trying to catch a distorted wavefront and don't care about mode purity, some detector packaging.

for a quantum link between two places, multimode is not a candidate.

## SMF : the one you'll actually use

not because it's optimal. it's because the stuff is already in the ground, and the promise of fiber quantum networking rests on running quantum channels over infrastructure that already exists. that infrastructure is G.652.

what works well: **time-bin and phase encoding.** the fiber doesn't care much about relative delay between two temporal modes, so a photon in a superposition of "early" and "late" survives a long span with its encoding intact. this is why so many deployed QKD systems are time-bin or phase based rather than polarization based. fully controllable time-bin entangled states have been distributed over 100 km of ordinary singlemode.

what doesn't work well: **polarization encoding, without help.** the random birefringence discussed earlier applies with full force. how bad depends entirely on how the fiber is installed:

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Installation</th><th>Polarization stability</th></tr>
</thead>
<tbody>
<tr><td>Buried duct</td><td>stable for hours</td></tr>
<tr><td>Aerial span, night</td><td>minutes</td></tr>
<tr><td>Aerial span, daytime sun and wind</td><td>seconds</td></tr>
<tr><td>Indoor patch panel, someone working in it</td><td>the moment they touch it</td></tr>
</tbody>
</table>
</div>

the fix is active compensation, not special fiber: send a time-multiplexed classical reference through the same fiber, measure the transformation it suffered, and drive an inline polarization controller to invert it. a 62 km link that was 70% aerial, where fidelity fell below 95% in under 20 seconds unassisted, held above **98% fidelity with 92.8% uptime** once compensated.

that's the pattern to remember: for deployed quantum links, you don't buy a better fiber, you buy a feedback loop.

## PMF : essential, but not for the span

PM fiber is everywhere in a quantum lab and almost nowhere between labs.

**where it belongs:** inside the source and the receiver. an entangled photon source is a chain of components (pump laser, waveguide or crystal, filters, interferometer arms, modulators) where the polarization at each interface determines whether the thing works at all. a Sagnac-based entangled pair source has a loop whose two counter-propagating paths must stay polarization-defined. a phase modulator has a defined axis and does nothing useful off it. all of that is PM, patch by patch, keyed connector by keyed connector.

**why it doesn't extend to the span:**

- **0.5 dB/km against 0.2 dB/km.** over 50 km that's an extra 15 dB, which is a factor of 30 in key rate. you would be paying for polarization stability with the very thing polarization stability was supposed to buy you.
- extinction ratio degrades with length and temperature. -30 dB at 100 m is excellent; over tens of kilometres in a duct with a diurnal temperature cycle, it's not the guarantee it looks like.
- **there is no PM plant.** nobody has PM fiber in the ground, and nobody is going to install any.

so it comes out as PMF for the metre scale and active compensation for the kilometre scale. if you find yourself specifying a multi-kilometre PM span, check whether time-bin encoding would let you avoid the problem entirely.

## MCF : the coexistence answer

the problem multicore solves is a real one, and it is not the one you would guess.

nobody is going to give a quantum network its own dark fiber plant. so quantum channels have to share fiber with classical traffic, and the usual way to share a fiber is wavelength: put the quantum channel at 1310 and the classical at 1550, or find a quiet slot in the C band. that helps, but it doesn't save you, because Raman scattering from a strong classical channel is *broadband*. there is no wavelength far enough away to be truly clean when the classical channel is at +10 dBm.

multicore replaces wavelength isolation with **spatial isolation**. put the quantum channel in its own core. the coupling between non-adjacent cores in uncoupled MCF is below -60 dB/km, which is isolation that wavelength filtering cannot approach.

it works. in a field deployed 4-core testbed in L'Aquila, a QKD channel ran in one core over **25.2 km** while the other three cores carried **110.8 Tb/s** across 510 wavelength-and-space channels, and the quantum channel still produced a secret key at 0.41 kbps. that is a genuinely absurd amount of classical light to be sharing a cladding with.

two practical notes from that work:

- **adjacent cores hurt, diagonal cores don't.** the diagonal core measured below -70 dB/km coupling and contributed essentially nothing. so: put the quantum channel diagonally opposite the loudest classical core. core assignment is a design parameter.
- when very high classical launch power is the requirement, MCF beats hollow core, because the spatial isolation scales with power in a way that spectral separation doesn't. one study puts tolerable classical power at **over 30 dBm per channel** with MCF against about 17 dBm with HCF when quantum and classical share a band.

there's a second, subtler use: **phase-stable synchronization**. distributed quantum protocols need shared timing references, and running the reference and the quantum signal in cores of the same fiber means they experience nearly the same environment, so the differential drift is tiny. sub-femtosecond stabilization across a multicore fiber has been demonstrated at 100% duty cycle.

## HCF : the one that's genuinely different

hollow core's quantum advantages are not the ones the marketing leads with. latency is nice. these are better.

**1. Raman noise essentially disappears.** spontaneous Raman scattering needs glass. take the glass out of the light path and the dominant noise mechanism for quantum-classical coexistence goes with it. in a four node entanglement network over 11.5 km of NANF carrying four 200 Gbps classical channels alongside, accidental coincidences ran at **0.0075 per second per bin**. the same setup in standard SMF would have given 0.4, roughly two orders of magnitude worse, and entanglement visibility would have collapsed from 94.3% to 14.4%. that is not a degraded measurement so much as no measurement.

**2. it guides where silica doesn't.** this one is underrated and might matter most of all. a lot of quantum hardware does not emit at 1550 nm. rubidium memories work at 780 and 795 nm. quantum dots emit around 930-980 nm. silica fiber at those wavelengths is bad, 1.7 dB/km at 930 nm and several dB/km in the visible, because Rayleigh scattering scales as 1/λ⁴ and there's nothing you can do about it.

hollow core doesn't have that constraint, because the guiding window is set by a membrane thickness rather than by the material:

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Wavelength</th><th>Hollow core</th><th>Silica (practical SM fiber)</th><th>What lives there</th></tr>
</thead>
<tbody>
<tr><td>660 nm</td><td>2.85 dB/km</td><td>~8 - 12 dB/km</td><td>visible sources, some detectors</td></tr>
<tr><td>780 / 795 nm</td><td>~1.5 - 2 dB/km</td><td>~4 dB/km</td><td>rubidium quantum memories</td></tr>
<tr><td>850 nm</td><td>1.45 dB/km</td><td>~2.5 - 3 dB/km</td><td>caesium, silicon detectors</td></tr>
<tr><td>934 nm</td><td>0.65 dB/km</td><td>1.7 dB/km</td><td>InGaAs quantum dots</td></tr>
<tr><td>1064 nm</td><td>0.52 dB/km</td><td>~1 - 1.5 dB/km</td><td>pump lasers, some sources</td></tr>
<tr><td>1550 nm</td><td>0.09 - 1 dB/km</td><td>0.14 - 0.20 dB/km</td><td>telecom, everything else</td></tr>
</tbody>
</table>
</div>

the silica column is what you'd actually get from an HP-series short-wavelength fiber, not the theoretical Rayleigh floor. the shape is what matters: silica loss climbs as 1/λ⁴ going down in wavelength, and hollow core just doesn't.

the alternative is **quantum frequency conversion**, a nonlinear crystal that shifts your 930 nm photon to 1550 nm so it can use telecom fiber. that works, and it's a real technique, but it costs you a conversion setup at each end and it costs you photons. being able to skip it changes the architecture. all four BB84 polarization states have been sent through 340 m of HCF at 934 nm with **0.1% QBER**, alongside strong classical traffic at 1550 nm in the same fiber, with single photon indistinguishability preserved at 92.7% HOM visibility.

3. low birefringence, so polarization survives better. less glass and a more symmetric structure means less stress birefringence, so polarization encoding drifts less than in SMF. HCF isn't polarization maintaining, since it has no defined axis, but it's a quieter channel to run polarization through.

**4. latency as a rate multiplier.** in a repeater chain, the attempt rate for entanglement swapping is set by the round trip time to the midpoint. cutting 32% off the propagation delay directly raises how many attempts per second you get before your memory decoheres. this is a real effect, not a marketing one, and it compounds along a chain.

**the catch:** those 0.85-0.98 dB/km numbers in the network demonstration are what real deployed hollow core looked like recently, not 0.091. the record fiber and the fiber you can buy are not the same fiber yet, and the splice losses are still 0.3-0.6 dB a joint. for now, hollow core wins on noise and on wavelength access, and it will win on loss later.

## the quantum summary

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Fiber</th><th>Role in quantum networking</th><th>Verdict</th></tr>
</thead>
<tbody>
<tr><td>MMF</td><td>collection optics inside an apparatus, free space receivers</td><td>never for a link</td></tr>
<tr><td>SMF</td><td>every deployed link. time-bin and phase encoding natively, polarization with active compensation</td><td>the default, because it exists</td></tr>
<tr><td>PMF</td><td>inside sources, modulators, interferometers, receivers. metres, not kilometres</td><td>essential at the ends, wrong for the middle</td></tr>
<tr><td>MCF</td><td>quantum and classical in the same cable with spatial isolation, and phase-stable sync</td><td>best answer to coexistence at high classical power</td></tr>
<tr><td>HCF</td><td>near-zero Raman noise, access to 700-1100 nm, low latency, low birefringence</td><td>the most interesting, still maturing</td></tr>
</tbody>
</table>
</div>

## the short version

**multimode** trades distance for cheap transceivers. use it inside a room.

**singlemode** is the default for everything else, and the reason is that one strand of it has carried every generation of transceiver anyone has invented and will carry the next one.

**PM fiber** solves polarization by making the fiber so birefringent that the random birefringence stops mattering. it's a component, not a medium. think metres.

**multicore** buys capacity per square millimetre, which matters when your limit is cable diameter or faceplate space rather than glass cost. it also happens to be the cleanest way to put quantum and classical traffic in the same cable.

**hollow core** takes the glass out of the light path, and everything that was a property of glass goes with it: the speed penalty, the nonlinearity, the Raman scattering, and the wavelength window you were stuck with.

for quantum specifically, the ranking flips depending on what's hurting you. **loss-limited? hollow core.** noise-limited from classical neighbours? multicore, or hollow core. encoding-limited? change the encoding before you change the fiber. and if you're building at the metre scale on a bench, it's PM fiber all the way down, and it always will be.

## where the numbers came from

most of the classical figures are standard datasheet and ITU-T values. the research results are worth reading in full if any of this is your job:

- hollow core at 0.091 dB/km: [Southampton / Microsoft, reported in Nature Photonics](https://www.networkworld.com/article/4049666/microsofts-hollow-core-fiber-delivers-the-lowest-signal-loss-ever.html)
- hollow core loss from 600 to 1100 nm: [Nature Communications, "Hollow core optical fibres with comparable attenuation to silica fibres between 600 and 1100 nm"](https://www.nature.com/articles/s41467-020-19910-7)
- four node entanglement network over NANF, with the Raman noise comparison: [npj Quantum Information](https://www.nature.com/articles/s41534-025-01125-7)
- quantum dot photons at 934 nm over hollow core, avoiding frequency conversion: [arXiv 2509.11889](https://arxiv.org/html/2509.11889)
- QKD plus 110.8 Tb/s over a field deployed 4-core fiber in L'Aquila: [Light: Science & Applications](https://www.nature.com/articles/s41377-025-01982-z)
- the MCF-versus-HCF coexistence comparison and launch power limits: [Entropy 26(7), 601](https://www.mdpi.com/1099-4300/26/7/601)
- entanglement over a 62 km polarization-stabilized aerial link: [arXiv 2601.11753](https://arxiv.org/html/2601.11753)
- multicore practicalities, FIFO, splicing, IEC 61300-3-55: [Senko multicore fiber application note](https://www.senko.com/wp-content/uploads/2026/05/Application-Note_Multicore-Fibers.pdf)