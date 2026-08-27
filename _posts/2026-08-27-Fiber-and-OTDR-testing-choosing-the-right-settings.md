---
layout: post
section-type: post
title: "Fiber and OTDR testing : how to choose the right settings"
category: 'Networking'
tags: [ 'networking', 'fiber', 'otdr' ]
---

every time i hand an OTDR to someone new, the same thing happens. they plug the fiber in, hit `AUTO`, get a trace that looks like a ski slope with some spikes on it, and then ask me the only question that matters: *is this good or not?*

auto mode is fine for a quick look. but the moment you need to certify a link, find a break, or prove to a vendor that the problem is on their side, you have to open the settings menu and make some choices. and those choices are not arbitrary, every one of them is a trade against something else.

this post is the explainer i wish someone had given me. it goes through each setting, what it actually does to the trace, and how to pick a value based on the fiber in front of you.

## first, what an OTDR is not

an OTDR is not a power meter. this trips people up constantly.

a **light source and power meter** (LSPM, or an OLTS if it's the automated flavor) measures the *real* end-to-end loss of a link. you put a known amount of light in one end and measure what comes out the other. that number is the truth, and it's what most standards actually want you to certify against.

an **OTDR** measures the link from one end only, by firing a pulse down the fiber and listening to what comes back. it gives you a map: where the events are, how much each one costs you, and what the fiber looks like between them. it's a troubleshooting and characterization tool.

they answer different questions. the OTDR tells you *where* and *why*, the power meter tells you *how much*. on a real install you usually want both.

## how the thing actually works

two physical effects do all the work.

**rayleigh backscattering** — glass isn't perfectly uniform at the molecular level, so a tiny fraction of the light scatters in every direction as it travels. a small part of that scatter goes back toward the OTDR. this is the sloped line on your trace. its slope *is* the fiber attenuation in dB/km.

**fresnel reflection** — wherever the refractive index changes abruptly (a connector, a mechanical splice, a clean break, the far end of the fiber), a chunk of light bounces straight back. these are the spikes.

the OTDR fires a pulse, times how long the light takes to come back, converts that time to distance, and plots power against distance. everything else is detail.

one thing to keep in your head from the start: **the OTDR sees the round trip, but displays one-way distance and one-way dB.** it does the halving for you. this is why some numbers feel off by a factor of two when you do the math by hand.

## anatomy of a trace

before the settings make sense you need to be able to read the picture:

- **the front spike** — the reflection off the OTDR's own connector. everything under it is blind. this is the reason launch fibers exist.
- **the slope** — backscatter from the fiber itself. a clean fiber gives you a straight line. the steeper it is, the more dB/km you're burning.
- **a step down with no spike** — a fusion splice, or a bend. loss, no reflection.
- **a spike with a step down** — a connector or a mechanical splice. loss *and* reflection.
- **a step down that's steeper than the rest but not a clean step** — usually a macrobend or a stressed section of cable.
- **the last big spike, then noise** — end of fiber. after that you're looking at the noise floor.

## setting 1 : wavelength

start here, because it changes what the rest of the trace even means.

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Wavelength</th><th>Fiber</th><th>Typical attenuation</th><th>What it's good for</th></tr>
</thead>
<tbody>
<tr><td>850 nm</td><td>multimode</td><td>~3.0 dB/km</td><td>data center / short MM links, matches how most MM transceivers run</td></tr>
<tr><td>1300 nm</td><td>multimode</td><td>~1.0 dB/km</td><td>longer MM backbone, second wavelength for MM certification</td></tr>
<tr><td>1310 nm</td><td>singlemode</td><td>0.32 - 0.40 dB/km</td><td>the default SM wavelength, least bend-sensitive of the three</td></tr>
<tr><td>1550 nm</td><td>singlemode</td><td>0.18 - 0.25 dB/km</td><td>long spans, and the wavelength that exposes bends</td></tr>
<tr><td>1625 nm</td><td>singlemode</td><td>~0.22 - 0.25 dB/km</td><td>testing live fiber out of band, and the most sensitive bend detector</td></tr>
</tbody>
</table>
</div>

**the rules:**

- **test at the wavelength the system actually runs at.** if the link carries 1310 nm traffic, a beautiful 1550 nm trace proves less than you think.
- **for singlemode, always test both 1310 and 1550.** the pair is what makes bend detection possible (more on that below).
- **for multimode, test 850 and 1300.**
- **1550 reaches further than 1310** for the same pulse and averaging, because the fiber attenuates it less. if you're hunting for a break at the far end of a long span, 1550 is your friend.
- **1625 (or 1650) is the out-of-band wavelength.** it sits above the C-band so you can test a fiber that is carrying live traffic, provided there's a filter in place. on most units it has its own dedicated port for exactly this reason.

## setting 2 : pulse width

this is the setting that matters most, and the one people get wrong.

the pulse width is how long the laser stays on for each shot. longer pulse = more energy in the fiber = more backscatter coming back = you can see further. that's the whole benefit. the cost is that a long pulse physically occupies a long stretch of glass, and anything inside that stretch gets smeared into one blob.

the conversion is easy. light moves through fiber at roughly **0.2 metres per nanosecond** (about 200,000 km/s, because the glass slows it down by the index of refraction). so:

```
    length of fiber lit up (m)  ~=  0.2  x  pulse width (ns)
    best case dead zone    (m)  ~=  0.1  x  pulse width (ns)
```

the second line is halved because of the round trip. a 100 ns pulse lights up about 20 m of glass, and the very best you can hope for is that two events 10 m apart stay distinguishable. in practice it's worse than that, because the detector also needs time to recover from a bright reflection.

here's the working table. treat these as ballpark, your unit's datasheet has the real numbers:

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Pulse width</th><th>Fiber lit up</th><th>Event dead zone</th><th>Attenuation dead zone</th><th>Practical reach</th><th>Use it for</th></tr>
</thead>
<tbody>
<tr><td>3 - 5 ns</td><td>~1 m</td><td>~1 m</td><td>~3 m</td><td>&lt; 500 m</td><td>patch panels, jumpers, closely spaced connectors</td></tr>
<tr><td>10 ns</td><td>~2 m</td><td>~2 m</td><td>~5 m</td><td>~1 km</td><td>FTTH drops, MDU risers, short MM</td></tr>
<tr><td>30 ns</td><td>~6 m</td><td>~6 m</td><td>~15 m</td><td>~3 km</td><td>in-building backbone, campus</td></tr>
<tr><td>100 ns</td><td>~20 m</td><td>~20 m</td><td>~50 m</td><td>~10 km</td><td>access network, short metro</td></tr>
<tr><td>300 ns</td><td>~60 m</td><td>~60 m</td><td>~150 m</td><td>~25 km</td><td>metro</td></tr>
<tr><td>1 us (1000 ns)</td><td>~200 m</td><td>~200 m</td><td>~500 m</td><td>~60 km</td><td>regional backbone</td></tr>
<tr><td>3 us</td><td>~600 m</td><td>~600 m</td><td>~1.5 km</td><td>~120 km</td><td>long haul</td></tr>
<tr><td>10 - 20 us</td><td>2 - 4 km</td><td>2 - 4 km</td><td>5 - 10 km</td><td>200 km +</td><td>ultra long haul, locating a break on a dead span</td></tr>
</tbody>
</table>
</div>

**the rules:**

- **start short, go longer only when you have to.** the shortest pulse that still puts the end of the fiber comfortably above the noise floor is the right one.
- **if the end of the fiber is buried in noise, go up one step.** repeat. don't jump straight to 10 us.
- **if two events are merging into one, go down one step.** if you can't go lower, you've hit the physical limit and you need to accept a combined loss number for that pair.
- **run two acquisitions.** this is the trick that makes life easy: one short pulse for the near end where all the connectors are, one long pulse to reach the far end. most units let you save both, and some do it automatically and stitch the result. don't fight to find one pulse width that does everything, because it usually doesn't exist.

## setting 3 : distance range

the range is how far out the OTDR plots. it's not the same as how far it can *measure* — that's dynamic range, which is a property of the instrument, not a setting.

set it too short and you cut off the end of your fiber. set it too long and two bad things happen: the fixed number of sampling points gets spread over a longer distance so your resolution gets coarser, and each acquisition takes longer for no benefit.

**the rule:** set the range to the first step **above** your link length, aiming for roughly **link length + 20-25%**. for a 4 km link, a 5 km range is right. a 40 km range is not.

if you don't know the length, run one fast auto acquisition first to find out, then set the range manually and run it properly.

one exception: if you're hunting **ghosts** (see below), deliberately setting the range to 1.5 - 2x the fiber length lets you see whether extra reflections show up past the real end of the fiber, which is how you confirm they're ghosts.

## setting 4 : averaging / acquisition time

each shot comes back noisy. the OTDR fires thousands of them and averages, and the noise falls away while the real signal stays. more averaging = cleaner trace = more usable dynamic range = you can see further and measure small events more accurately.

the catch is that it's a square-root relationship, so it gets expensive fast. roughly, **doubling the averaging time buys you about 0.75 dB**, and quadrupling it buys about 1.5 dB. going from 3 minutes to 6 minutes to gain three quarters of a dB is usually not worth standing in a cold manhole for.

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Averaging time</th><th>When to use it</th></tr>
</thead>
<tbody>
<tr><td>5 - 10 s</td><td>quick look, is the fiber there, where's the break</td></tr>
<tr><td>15 - 30 s</td><td>normal troubleshooting on a short or clean link</td></tr>
<tr><td>30 - 60 s</td><td>measurements you're going to write down</td></tr>
<tr><td>1 - 3 min</td><td>documentation / acceptance testing, noisy traces, small splice losses</td></tr>
<tr><td>3 min +</td><td>long spans at the edge of the instrument's range</td></tr>
</tbody>
</table>
</div>

**the rules:**

- **finding a break: short time.** you need a distance, not a pretty trace. 5-15 seconds.
- **certifying a link: long time.** if you're going to put a number in a report, average for at least a minute. a 0.05 dB splice measured on a noisy trace is a guess.
- **if the far end is marginal, averaging is cheaper than a longer pulse.** more averaging gets you reach *without* growing your dead zones. try that before you step the pulse width up.

## setting 5 : index of refraction (IOR)

the OTDR measures time. to turn time into distance it needs to know how fast light travels in *your* fiber, and that's the IOR (strictly the group index).

get it wrong and every distance on the trace is wrong by the same percentage. that's fine when you're comparing events, and very much not fine when you're telling a crew where to dig.

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Fiber</th><th>850 nm</th><th>1300 nm</th><th>1310 nm</th><th>1550 nm</th><th>1625 nm</th></tr>
</thead>
<tbody>
<tr><td>Singlemode G.652</td><td>-</td><td>-</td><td>1.4675</td><td>1.4681</td><td>1.4683</td></tr>
<tr><td>Multimode 50/125 (OM3/OM4)</td><td>1.4818</td><td>1.4790</td><td>-</td><td>-</td><td>-</td></tr>
<tr><td>Multimode 62.5/125 (OM1/OM2)</td><td>1.4960</td><td>1.4910</td><td>-</td><td>-</td><td>-</td></tr>
</tbody>
</table>
</div>

**the rules:**

- **get the real value from the cable datasheet** whenever you can. the manufacturer publishes it.
- **the defaults above are close enough for troubleshooting**, they're within a fraction of a percent for standard fiber.
- **the classic mistake is leaving a multimode IOR set while testing singlemode.** 1.4960 instead of 1.4681 is a 1.9% error. on a 10 km span that's 190 m of digging in the wrong place.
- **IOR does not affect loss**, only distance. if your losses look right but the distances are consistently off by a few percent, this is why.
- remember the OTDR reports **fiber length, not cable length**. fiber is stranded and has slack, so it's typically 1-2% longer than the cable, and the cable is longer than the trench. keep a correction factor if you care about the last few metres.

## setting 6 : sampling resolution

how far apart the measurement points are. it can be anything from a few centimetres to several metres depending on the pulse width and range you picked.

you usually don't set this directly, it falls out of the other choices. what you need to know is that **a long range spreads your points thinner**, so keeping the range tight is what keeps the resolution fine. a 100 ns pulse on a 5 km range gives a much more detailed picture than the same pulse on a 100 km range.

if your unit offers a "high resolution" or "extra points" mode, it just means more samples for more acquisition time. worth it for documentation, skip it when you're chasing a break.

## setting 7 : detection thresholds

these decide what the auto-analysis calls an "event" and puts in the table. too tight and you get a page of noise flagged as splices. too loose and you miss a marginal connector.

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Threshold</th><th>Typical default</th><th>Notes</th></tr>
</thead>
<tbody>
<tr><td>Loss (splice) threshold</td><td>0.05 dB (SM), 0.10 - 0.15 dB (general)</td><td>tighten to 0.02 dB for careful SM work, loosen on noisy traces</td></tr>
<tr><td>Reflectance threshold</td><td>-65 dB (SM), -55 dB (MM)</td><td>anything worse than this gets flagged as a reflective event</td></tr>
<tr><td>End of fiber threshold</td><td>~5 dB</td><td>a drop this big is treated as the end, not as an event</td></tr>
</tbody>
</table>
</div>

for reference, here's what reflectance values actually mean in the field:

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>What it is</th><th>Reflectance</th></tr>
</thead>
<tbody>
<tr><td>Fusion splice</td><td>no reflection (that's the point)</td></tr>
<tr><td>UPC connector, good</td><td>-45 to -55 dB</td></tr>
<tr><td>APC connector, good</td><td>-55 to -65 dB</td></tr>
<tr><td>Open / unmated UPC, or a clean break</td><td>~-14 dB (glass to air, roughly 4% straight back)</td></tr>
<tr><td>Open / unmated APC</td><td>-45 to -60 dB (the angle throws the reflection into the cladding)</td></tr>
</tbody>
</table>
</div>

that -14 dB number is worth memorising. **a big fat reflection at the end of your trace means an open connector or a clean break. a soft rounded drop into the noise with no reflection means a crushed, shattered or bent fiber.** that one observation tells you what kind of failure you're dealing with before you leave the office.

## the thing that isn't a setting : dynamic range

dynamic range is the instrument's spec, the distance in dB between the backscatter level at the start of the fiber and the noise floor. it's what determines how long a link the unit can actually test. you can't set it, but it drives every other choice you make.

two things to know:

**the usable range is lower than the number on the box.** vendors spec it at the longest pulse, 3 minutes of averaging, at SNR = 1, which is the point where the signal is *indistinguishable* from noise. for measurements you can trust, knock about 5 dB off. a 35 dB unit gives you roughly 30 dB of usable range.

**pick a unit with 5 to 8 dB more dynamic range than the worst loss you expect to see.** so:

```
    30 dB usable  at  0.20 dB/km (1550 nm)   =  24 dB of fiber  ->  120 km
    plus a splice every 2 km at 0.1 dB       =   6 dB of splices
    ------------------------------------------------------------
    total                                        30 dB, ~120 km
```

which is where the "35 dB unit certifies about 120 km" rule comes from.

## dead zones, and why you need launch and receive fibers

the front connector reflection saturates the detector. while it's recovering, it's blind. that blind stretch is the dead zone, and there are two kinds:

- **event dead zone (EDZ)** — the minimum distance after a reflection before the OTDR can *detect* another event. measured 1.5 dB down from the peak. this is the smaller number.
- **attenuation dead zone (ADZ)** — the minimum distance after a reflection before the OTDR can *accurately measure the loss* of the next event. measured where the trace settles back to within 0.5 dB of the backscatter line. always bigger than the EDZ, usually by 2-3x.

detecting an event and measuring it are two different things, and the ADZ is the one that matters when you're certifying.

both grow with pulse width, and both grow with how reflective the event is. a filthy UPC connector produces a longer dead zone than a clean APC one, which is one more reason to clean everything.

**the fix is a launch fiber** (also called a launch cable, pulse suppressor, or fiber ring). it's a spool of fiber between the OTDR and the link, long enough that the front dead zone is entirely used up inside the spool. that way your first real connector lands on clean backscatter and can actually be measured. a **receive fiber** on the far end does the same for the last connector, which otherwise sits inside the end-of-fiber reflection and can't be measured at all.

**sizing rule:** launch fiber length in metres ≈ **pulse width in ns ÷ 10**, then round up generously.

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Pulse width</th><th>Minimum launch fiber</th><th>Practical choice</th></tr>
</thead>
<tbody>
<tr><td>&le; 100 ns</td><td>~25 m</td><td>100 m spool</td></tr>
<tr><td>500 ns</td><td>~50 m</td><td>150 - 300 m (typical PON / FTTx)</td></tr>
<tr><td>1 us</td><td>~100 m</td><td>200 - 500 m</td></tr>
<tr><td>5 us</td><td>~500 m</td><td>1 km</td></tr>
<tr><td>10 us</td><td>~1 km</td><td>1 - 2 km</td></tr>
<tr><td>20 us</td><td>~2 km</td><td>2 - 4 km</td></tr>
</tbody>
</table>
</div>

two practical notes: **match the fiber type** (a 50 µm launch fiber on a 62.5 µm link will lie to you), and **match the connector type** so you're not adding an extra mated pair at the patch panel just to adapt.

if your pulse is longer than the launch fiber can absorb, the launch fiber does nothing except add length. that's the most common way people get this wrong.

## weird stuff on the trace

### ghosts

extra spikes that aren't real events. light bounces back and forth between two strongly reflective points and the echo arrives late, so the OTDR plots it further down the fiber.

**how to spot one:**

- if a strong reflection sits at distance D, the ghost shows up at **2D**, and possibly at other multiples of D
- **it shows no loss** — the trace level is the same on both sides of it
- it may appear *past* the end of the fiber, where there is no fiber
- add a length of fiber to the front of the link and re-test: real events move with it, the spacing of ghosts changes in a way that gives them away

**how to kill it:** shorter pulse width (less energy to bounce around), clean or replace the offending connector, use APC where you can, index matching gel on a mechanical splice. or just recognise it and ignore it.

### gainers

a splice that appears to *amplify* the light. the trace steps up instead of down.

nothing is being amplified. it happens when two fibers with different mode field diameters are spliced (a G.652 spliced to a G.657, for example). the OTDR infers loss from backscatter, and the second fiber returns backscatter more efficiently than the first, so the step looks positive.

**the fix is to test from both ends and average the two results.** one direction shows an exaggerated loss, the other shows a gain, and the average is the true splice loss. this is why standards ask for bidirectional testing, and it's not optional if you care about splice numbers.

### macrobends

a bend tight enough that light escapes the core. **longer wavelengths leak more**, because they have a bigger mode field.

**the test:** measure the same event at two wavelengths. a splice or connector loses roughly the same amount at any wavelength. a bend does not.

```
    loss at 1550 (or 1625)  -  loss at 1310   >  0.2 dB   ->  macrobend
    roughly equal at both wavelengths                     ->  splice or connector
```

this is the single best reason to always test singlemode at both wavelengths. a bend hiding in a cable tray looks exactly like a mediocre splice at 1310 nm alone.

### merged events

two connectors closer together than the attenuation dead zone show up as one event with a combined loss. common in patch panels and short jumpers.

you can't fix this with settings alone past a point. drop the pulse width as far as it goes, and if they still merge, report the pair as a group. an OTDR physically cannot separate two events inside its dead zone.

## putting it together

<div class="table-responsive">
<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Scenario</th><th>Wavelength</th><th>Pulse</th><th>Range</th><th>Averaging</th><th>Launch fiber</th></tr>
</thead>
<tbody>
<tr><td>Data center MM, &lt; 300 m</td><td>850 + 1300</td><td>3 - 10 ns</td><td>500 m - 1 km</td><td>30 s</td><td>100 m MM, matched core</td></tr>
<tr><td>In-building / campus SM, 1 - 3 km</td><td>1310 + 1550</td><td>10 - 30 ns</td><td>5 km</td><td>30 - 60 s</td><td>100 - 300 m</td></tr>
<tr><td>FTTH drop / PON leg</td><td>1310 + 1550</td><td>5 - 30 ns</td><td>2 - 5 km</td><td>60 s</td><td>150 - 300 m</td></tr>
<tr><td>Access / short metro, 10 - 25 km</td><td>1310 + 1550</td><td>100 - 300 ns</td><td>25 - 30 km</td><td>60 - 180 s</td><td>500 m</td></tr>
<tr><td>Metro / regional, 40 - 80 km</td><td>1550</td><td>1 - 3 us</td><td>80 - 100 km</td><td>3 min</td><td>1 km</td></tr>
<tr><td>Long haul, 100 km +</td><td>1550</td><td>3 - 20 us</td><td>150 km +</td><td>3 min +</td><td>1 - 2 km</td></tr>
<tr><td>Hunting a break, any link</td><td>1550</td><td>as long as needed</td><td>1.5x link</td><td>5 - 15 s</td><td>optional</td></tr>
</tbody>
</table>
</div>

## a checklist before you press start

1. **clean and inspect every connector.** with a scope, not with your eye. dirt is the cause of most bad OTDR results, and it will also transfer dirt into the OTDR's own port.
2. **set the fiber type and IOR** to match what's actually in the ground.
3. **connect the launch fiber**, sized for the pulse width you're about to use.
4. **set range** to link length + 20-25%.
5. **start with a short pulse.** step up only if the far end is in the noise.
6. **check the far end is above the noise floor** before you trust anything.
7. **average longer** if you're going to write the numbers down.
8. **test both wavelengths**, and compare them for bends.
9. **test from both ends** and average, if splice loss numbers matter.
10. **back it up with a power meter** for the actual end-to-end loss.

## the short version

the whole thing collapses to one trade: **pulse width buys you distance and costs you resolution.** everything else is bookkeeping around that.

short pulse to see detail near you, long pulse to see far away, averaging to buy reach without paying in dead zone, launch fiber to stop the front reflection eating your first connector, and two wavelengths so bends can't hide.

and if the trace still doesn't make sense, clean the connectors again. it's almost always the connectors.