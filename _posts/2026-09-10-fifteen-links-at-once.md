---
layout: post
section-type: post
title: "Fifteen links at once : when a quantum network needs a scheduler"
category: 'networking'
tags: [ 'networking', 'fiber', 'quantum' ]
---

at the end of [the eight-users post]({% post_url 2026-05-29-eight-users-on-one-chip-maximizing-the-source-side-of-a-flex-grid-quanntum-network %}) i wrote down the experiment i wanted somebody to run:

> eight users, 50 GHz channels, deployed metro fiber, all links live at once, finite-key accounting. that single measurement is what separates a network from a demonstration, and i suspect the result would be the useful kind of disappointing: positive key on the short links, nothing on the long one, and an operator who now needs a policy for that.

six weeks later a group in Bristol posted most of it. [Wang et al., "Dynamic Entanglement Distribution for Multi-User and Multi-Protocol Quantum Networking," arXiv:2607.15262](https://arxiv.org/abs/2607.15262). six users instead of eight, 100 GHz channels instead of 50, deployed campus and metropolitan fiber, all fifteen links live at once, for 157 hours.

i was wrong about the disappointing part. every one of the fifteen links produced key, including the long one. what i got right, and did not expect to get right in this particular shape, is the last clause. **the operator does now need a policy**, and the paper contains the first experiment i have seen that puts a number on what the policy is worth.

below: the box, the 157 hours, the scheduling question, and the parts that are still missing.

## the box

[the flex grid post]({% post_url 2026-04-04-Flex-grid-for-entanglement %}) was about replacing a tree of fixed filters with one wavelength selective switch, so that who-talks-to-whom becomes a config push. this paper calls the resulting thing a **q-ROADM**, which is the right name for it, and builds it out of parts a transport engineer would recognize on sight.

- a 30-channel 100 GHz DWDM demultiplexer, on the ITU grid
- a Polatis 192×192 optical fiber switch
- two 4×16 wavelength selective switches, Finisar WaveShaper 16000S, about 5 dB each
- two 1×16 passive multiplexers, about 1.3 to 2.7 dB
- a fiber polarization controller per DWDM channel

the source is a type-0 SPDC Sagnac loop, MgO:PPLN, pumped continuous-wave at 775.06 nm, emitting |Φ⁺⟩ centred at 1550.12 nm with roughly 70 nm of bandwidth across S, C and L. the 30 channels are taken as 15 conjugate pairs, `[λᵢ, λ₋ᵢ]`, one pair per user pair. six users make fifteen pairs. the arithmetic closes exactly, which is the tidiest possible version of the allocation problem and worth noticing, because it means this network never has to decide who goes without.

### not every user gets the same box

the detail i want to pull out is the asymmetry. Alice and Bob each hang off a passive 1×16 multiplexer. Chloe and David share one WaveShaper; Faye and Grant share the other.

the paper gives the reason plainly. a passive mux costs you 1.3 to 2.7 dB and a WSS costs you about 5 dB, so the passive path is better on loss. but a passive mux is hardwired, so every channel it might ever carry needs its own port on the fiber switch. the WSS is programmable, so it collapses many potential channels onto one port.

so this is a port-count versus insertion-loss trade, made per user, and it is the most classically-networking thing in the paper. it also means **Alice and Bob are structurally better-served than Chloe through Grant**, by two to four dB, forever, because of how they were cabled. that is a service tier, arrived at by accident of hardware. i doubt the paper intends it as one, and i expect it becomes a real design question the moment somebody has to write a service catalogue for one of these.

### where the fiber actually goes

the source sits at the Centre for Nanoscience and Quantum Information and reaches the q-ROADM at the Smart Internet Lab over 0.8 km of campus fiber. the users hang off the q-ROADM over deployed fiber: 1.6 km each for five of them, and 5.6 km for David, whose path runs out through a 4.8 km metropolitan loop via Watershed and back.

per-user loss, from the paper's table:

<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>User</th><th>Fiber</th><th>Loss</th></tr>
</thead>
<tbody>
<tr><td>Alice</td><td>1.6 km</td><td>8.9 &ndash; 10.0 dB</td></tr>
<tr><td>Bob</td><td>1.6 km</td><td>8.1 &ndash; 11.1 dB</td></tr>
<tr><td>Chloe</td><td>1.6 km</td><td>10.6 &ndash; 13.0 dB</td></tr>
<tr><td>David</td><td>5.6 km</td><td>14.1 &ndash; 15.1 dB</td></tr>
<tr><td>Faye</td><td>1.6 km</td><td>10.6 &ndash; 11.9 dB</td></tr>
<tr><td>Grant</td><td>1.6 km</td><td>11.0 &ndash; 11.9 dB</td></tr>
</tbody>
</table>

that word "loop" is doing work. the fiber is real, installed, and outdoors. the users are all still in one building, and so is every detector. twelve SNSPDs, two per user, all feeding one Swabian Time Tagger Ultra.

which is the same scoping note as [the quantum LAN post]({% post_url 2026-09-07-a-quantum-lan-on-deployed-fiber %}) had to make about its own testbed, and i will come back to it, because a single central time tagger quietly removes the hardest problem in the building.

## 157 hours

the headline run is the full mesh, all fifteen links, for **157.3 hours**, about a week.

secret key rates ran from **9.8 bps** on the worst link (David to Grant, the long one) to 153.2 bps on the best (Alice to Bob, the two passive-mux users), averaged over the whole run. instantaneous values spanned 4.8 to 190 bps.

two interruptions, both reported:

- **T1 = 18.43 h.** several links started losing key and the fiber polarization on every user was re-neutralized. the paper does not report what QBER did on either side of that intervention, which i think is the single most useful number it leaves out. more below.
- **T2 = 58.03 h.** the SNSPD cryostat went through its regular cycle. that is a fridge cycling, and the paper labels it as one rather than folding it into an availability figure.

a week of fifteen simultaneous entanglement links on installed fiber with two logged interventions is, as far as i can tell, the longest continuously-reported multi-user entanglement service anybody has published. the 2021 Oak Ridge network ran for hours. this ran for a work week.

## the scheduling question

here is the part i did not see coming.

six users on a full mesh means every user's detector is looking at **five channels at once**. those channels are all on the same detector, so the singles rate at each user is five times what it would be with one link lit. accidental coincidences between two users go as the product of their singles rates, so on a full mesh the accidental floor on any given link is twenty-five times what it would be if that link were the only one running.

accidentals are unpolarized noise, so they push QBER up, and QBER enters the key rate through a function that falls off a cliff. so there is an actual choice:

- **run the full mesh.** every link gets 100% of the time and pays the full accidental penalty.
- **time-share partial meshes.** split the fifteen links into two sets that between them cover everything, run each set for half the time. every link now gets 50% of the time and a much lower accidental floor.

this is statistical multiplexing versus TDM, and it is the same argument transport people have had for forty years, except that here the thing being contended for is a *noise budget* rather than bandwidth.

the paper runs it as an experiment. forty minutes of full mesh against two twenty-minute partial meshes, same total time, swept across pump power, under two operating conditions.

**favourable** is about 15% heralding efficiency and under 100 ps of detector jitter. full mesh wins, at every pump power tested. more links lit is simply more key.

**challenging** is about 3% heralding efficiency and 300 to 350 ps of jitter. time-shared partial mesh wins, on many of the links, because cutting the number of simultaneous channels cuts the accidentals faster than halving the duty cycle costs you.

and there is a turnover inside the favourable case too: the full mesh generates *less* total key at maximum pump power than it does at 0.8 of it, because the extra pairs arrive with more than their share of extra accidentals.

<div class="qw" id="qw-mesh" data-qw><div class="qw-hd"><span class="qw-t">full mesh or time-shared halves</span><span class="qw-s">secret key on one link, against pump power, for both scheduling strategies. hover or tap to read values.</span><span class="qw-lg"><i style="background:#32c29e"></i>full mesh, 5 channels per user, all the time<i style="background:#f2a03d"></i>time-shared, 2 channels per user, half the time</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">link</span><span class="qw-b" data-grp="lk"><button type="button" data-v="0" class="on" aria-pressed="true">A&ndash;B, best</button><button type="button" data-v="1">A&ndash;D</button><button type="button" data-v="2">D&ndash;G, worst</button></span></div><div class="qw-g"><span class="qw-l">condition</span><span class="qw-b" data-grp="cd"><button type="button" data-v="fav" class="on" aria-pressed="true">favourable</button><button type="button" data-v="chal">challenging</button></span></div></div><div class="qw-sl"><label class="qw-sr"><span class="qw-l">heralding efficiency <b id="qwm-hev">15%</b></span><input type="range" id="qwm-he" min="2" max="20" step="0.5" value="15"></label><label class="qw-sr"><span class="qw-l">detector jitter <b id="qwm-jv">80 ps</b></span><input type="range" id="qwm-j" min="40" max="400" step="5" value="80"></label></div><svg class="qw-svg" viewBox="0 0 640 340" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Secret key rate versus pump power for full mesh and time-shared partial mesh"><g id="qwm-grid"></g><g id="qwm-curves"></g><g id="qwm-cross"></g></svg><div class="qw-out"><div class="qw-oi"><span class="qw-ok">pump</span><span class="qw-ov" id="qwm-P">0.80</span></div><div class="qw-oi"><span class="qw-ok qw-bs">full mesh</span><span class="qw-ov" id="qwm-f">&mdash;</span></div><div class="qw-oi"><span class="qw-ok qw-fs">time-shared</span><span class="qw-ov" id="qwm-p">&mdash;</span></div><div class="qw-oi qw-grow"><span class="qw-ok">verdict</span><span class="qw-ov qw-txt" id="qwm-v">&mdash;</span></div></div><p class="qw-n" id="qwm-note"></p><noscript><p class="qw-n">Under favourable conditions (15% heralding efficiency, 80 ps jitter) the full mesh produces more key than the time-shared halves at every pump power up to about 0.9, where the two cross because accidentals are growing faster than pairs. Under challenging conditions (3% heralding, 325 ps jitter) the full mesh crosses the QBER threshold above about half pump power and produces no key at all, while the time-shared configuration keeps working on every link.</p></noscript></div>

the model behind the widget is the paper's own arithmetic, not a fit to its figures: pairs and accidentals from the emitted rate, the channel count and the coincidence window, then the standard `1 − f_EC H₂(Q) − H₂(Q)` key fraction. i calibrated the single free constant, the emitted pair rate per channel, so that the best link lands near 153 bps and the worst near 10. it then reproduces both of the paper's regimes and the high-pump turnover without any further tuning, which is the main reason i believe the mechanism as stated.

what falls out of it is a rule an operator could actually use. **the full mesh is the right default when your QBER has headroom, and the wrong one the moment it does not.** heralding efficiency and detector jitter are the two things that decide which side of that line a network is on, and both of them are properties of the users' own equipment, not of the network. so the scheduling decision belongs to whoever knows the endpoints, and right now nothing in the architecture knows the endpoints.

## slicing, and joining

two more things the box does, both of which are ordinary in a classical network and new here.

**slicing.** the six users get partitioned into two independent three-user networks, and the paper does it three ways. then an optional Alice-to-Bob link is added back across the boundary by reprogramming the q-ROADM, with no physical change and no significant performance cost. that is a working demonstration of a logical topology decoupled from the physical one, which is the thing the flex-grid post was really asking for and did not get in 2021.

**joining.** the other protocol on the network is onboarding. Alice wants into an existing five-user mesh. she pre-shares an authentication key with one trusted neighbour, and then the network floods key material along multiple paths and concatenates it to authenticate her end to end.

the interesting result is that the *order* matters. two strategies, one going Alice to Bob and then around, the other going Alice to Grant and back the other way:

<table class="table table-bordered table-condensed" style="text-align:left">
<thead>
<tr><th>Condition</th><th>Order 1</th><th>Order 2</th><th>Difference</th></tr>
</thead>
<tbody>
<tr><td>favourable</td><td>1356 s</td><td>1298 s</td><td>58 s</td></tr>
<tr><td>challenging</td><td>6958 s</td><td>4371 s</td><td>2587 s</td></tr>
</tbody>
</table>

when the links are healthy, the order barely matters. when they are not, picking the wrong order costs you 43 minutes on a two-hour operation, because the sequence is serial and one slow link stalls everything behind it.

that is a routing problem with a well-known shape. it is scheduling a chain of tasks whose service times you can measure in advance, and the fix is to order by expected completion. nobody needs to invent anything here, and the paper does not claim to have; it just shows the cost of not doing it, which is the useful contribution.

## what is still missing

the paper is a large piece of work and most of what follows is scope rather than error. but four of these are things i asked for by name in earlier posts, and it is worth being straight about which ones arrived.

**the key rates are asymptotic.** the formula in the paper is `N_key ≥ N_sifted[1 − f_EC H₂(Q) − H₂(Q)]`, with no finite-size correction and no security parameter. so this is not yet the composed experiment i asked for; it is the composed experiment minus the accounting. and the accounting is not cosmetic on links running at 9.8 bps, where a day of integration is under a megabit of sifted key and the finite-size penalty is the difference between a positive number and zero. **[open question #3 from the eight-users post]({% post_url 2026-05-29-eight-users-on-one-chip-maximizing-the-source-side-of-a-flex-grid-quanntum-network %}) stands.**

**the reconfiguration time is still not published.** i asked for this in the flex-grid post in April and i am asking for it again, because this is the paper that should have it. there is a Polatis switch, two WaveShapers, and a polarization controller per channel, and the service-change latency of the whole thing is the sum of the switch time and however long the polarization compensation takes to re-settle afterward. that number decides whether slicing is an operational tool or a demo, and it appears nowhere. the hardware to measure it is already on the bench.

**one time tagger.** every detector is in one building on one clock, so the synchronization problem does not appear. that is not a criticism of a first q-ROADM paper, but it does mean the slicing and scheduling results are about a network whose hardest distributed problem has been removed. put the six users in six buildings and every reconfiguration now has to re-establish timing as well as polarization, and the reconfiguration time i just complained about gets a second term.

**no classical traffic.** the fiber is dark. everything in [the coexistence post]({% post_url 2026-03-23-coexistence-running-a-quantum-channel-and-its-clock-down-the-same-fiber %}) is therefore absent, and this is the network where it would bite hardest: the whole argument of the scheduling section is about an accidental floor, and Raman from a lit classical channel is another term in exactly that floor, one that varies per wavelength. the allocator would have to know it.

**what happened at 18.43 hours.** several links degraded, everything was re-neutralized, the run continued. the paper does not say which links, by how much, or over what timescale the degradation built up. that is the closest thing in the dataset to a real operational fault on a real quantum network, and it is one sentence.

## what i'd build next

**1. publish the drift, not only the intervention.** the 157 hours contain a week of per-link QBER on installed fiber. that time series is more valuable than the key rates, because it is the only thing that tells an operator how often a technician has to touch the network. a plot of QBER per link against time, with T1 marked, would answer a question nobody currently has data for.

**2. make the scheduler an actual scheduler.** the paper compares two fixed configurations. the result it demonstrates, that the right topology depends on heralding efficiency and jitter, is an argument for a controller that measures both and picks. the inputs are already being logged. this is a control loop that could be closed with the equipment in the photograph.

**3. price the port.** the passive-mux versus WSS choice is a real cost function: dB against fiber-switch ports. write it down. for a given user count and a given switch radix there is an optimal mix, and right now the mix in this paper looks like whatever hardware was in the lab.

**4. six buildings.** the loop-back topology is the honest limit of this result. the same experiment with the users actually distributed, on White Rabbit rather than one time tagger, would turn the reconfiguration time into a measured quantity and would tell us whether polarization re-neutralization is a per-network event or a per-user one.

## the short version

- the composed experiment mostly arrived: **six users, fifteen simultaneous links, deployed campus and metro fiber, 157 hours**. the missing piece is finite-key accounting.
- the box is a q-ROADM built from a Polatis 192×192 fiber switch, two Finisar WaveShapers and two passive muxes, driving 15 conjugate 100 GHz channel pairs from a 70 nm type-0 SPDC source.
- users on passive muxes are two to four dB better off than users on WSSs, permanently. that is a service tier created by cabling.
- key rates ran 9.8 bps on the worst link to 153.2 bps on the best. two interruptions in a week, one polarization re-neutralization and one fridge cycle.
- **a full mesh multiplies your accidental floor by (N−1)².** with six users that is 25x on every link.
- so scheduling matters: full mesh wins when QBER has headroom, and time-shared partial meshes win when it does not. the deciding variables are heralding efficiency and detector jitter, which are properties of the users, not the network.
- slicing works and costs nothing measurable. a logical topology decoupled from the physical one, by config push.
- onboarding a new user is a serial chain, so the order matters: 2587 seconds between the good order and the bad one when links are poor, 58 seconds when they are healthy.
- still missing: the reconfiguration time, which i have now asked for twice; distributed clocks; classical traffic in the same glass; and what actually happened at hour 18.

most of what this paper had to decide was *policy*. which links to light, in what order, for what fraction of the time, and who gets the low-loss port. none of those have a quantum answer. they have operator answers, and this is the first entanglement network i have read about that got big enough, and ran long enough, to be forced to give them.

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
.qw-l b{text-transform:none;color:#32c29e;font-weight:600;font-variant-numeric:tabular-nums}
.qw-b{display:inline-flex;flex-wrap:wrap;gap:4px}
.qw-b button{font:inherit;font-size:12px;line-height:1;padding:7px 11px;background:#242424;color:#c8c8c8;border:1px solid #3a3a3a;border-radius:4px;cursor:pointer;transition:background .12s,color .12s,border-color .12s}
.qw-b button:hover{background:#2e2e2e;color:#fff}
.qw-b button.on{background:#32c29e;border-color:#32c29e;color:#10231f;font-weight:600}
.qw-b button:focus-visible{outline:2px solid #32c29e;outline-offset:2px}
.qw-sl{display:flex;flex-wrap:wrap;gap:16px;margin-bottom:12px}
.qw-sr{flex:1 1 230px;display:flex;flex-direction:column;gap:6px;cursor:pointer}
.qw-sr input[type=range]{-webkit-appearance:none;appearance:none;width:100%;height:4px;background:#3a3a3a;border-radius:2px;outline:none;margin:4px 0}
.qw-sr input[type=range]::-webkit-slider-thumb{-webkit-appearance:none;appearance:none;width:15px;height:15px;border-radius:50%;background:#32c29e;cursor:grab;border:0}
.qw-sr input[type=range]::-moz-range-thumb{width:15px;height:15px;border-radius:50%;background:#32c29e;cursor:grab;border:0}
.qw-sr input[type=range]:focus-visible{outline:2px solid #32c29e;outline-offset:4px}
.qw-svg{display:block;width:100%;height:auto;overflow:visible;cursor:crosshair}
.qw-out{display:flex;flex-wrap:wrap;gap:8px;margin-top:12px}
.qw-oi{flex:0 1 auto;min-width:126px;background:#212121;border:1px solid #303030;border-radius:4px;padding:7px 10px}
.qw-grow{flex:1 1 220px}
.qw-ok{display:block;font-size:10px;text-transform:uppercase;letter-spacing:.08em;color:#8f8f8f}
.qw-ok.qw-bs{color:#32c29e}
.qw-ok.qw-fs{color:#f2a03d}
.qw-ov{display:block;font-family:ui-monospace,SFMono-Regular,Menlo,Consolas,monospace;font-size:14px;color:#fff;margin-top:2px;font-variant-numeric:tabular-nums}
.qw-ov small{font-family:inherit;font-size:10.5px;color:#9a9a9a}
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

  var GREEN='#32c29e', AMBER='#f2a03d', AXIS='#3a3a3a', TICK='#7d7d7d';

  /* =======================================================================
     WIDGET - full mesh against time-shared halves

     Standard BBM92 accounting, run twice with different channel counts.

       eta_X   = 10^(-loss_X/10) . HE . 0.35
                 fibre transmission, heralding efficiency, and one lumped
                 factor for SNSPD efficiency and the polarisation analyser's
                 coupling (the paper quotes 52.8-76.1% for the latter).
       C       = R . eta_X . eta_Y                      true coincidences
       S_X     = m . R . eta_X                          singles, m channels live
       A       = S_X . S_Y . tau  =  C . m^2 . R . tau  accidentals

     That last identity is the whole point of the widget: the accidental-to-
     true ratio depends on the number of live channels SQUARED, on the emitted
     pair rate and on the window, and NOT on either endpoint's efficiency.  A
     full mesh of six users puts m = 5 on every detector, so every link carries
     25x the accidental floor it would carry alone.

       tau     = 2.5 x detector jitter
       Q       = (0.02 C + 0.5 A) / (C + A)
       SKR     = duty . 0.5 (C + A) [ 1 - 1.16 H2(Q) - H2(Q) ]

     Full mesh: m = 5, duty = 1.  Time-shared halves: m = 2, duty = 0.5.

     One free constant, the emitted pair rate per 100 GHz channel pair at full
     pump, set to R0 = 2.0e7 /s so that A-B lands near the paper's 153 bps and
     D-G near its 9.8.  With that fixed, the model reproduces the paper's two
     regimes and its high-pump turnover on its own.
     ==================================================================== */
  (function(){
    var root=document.getElementById('qw-mesh'); if(!root) return;

    var R0=2.0e7, SYS=0.35, EOPT=0.02, FEC=1.16;
    var LINKS=[{n:'A–B',a:8.9,b:8.1},{n:'A–D',a:8.9,b:15.1},{n:'D–G',a:15.1,b:11.9}];
    var lk=0, HE=15, JIT=80, hover=null;

    function H2(q){ return (q<=0||q>=1)?0:(-q*Math.log(q)-(1-q)*Math.log(1-q))/Math.LN2; }
    function calc(P,m,duty){
      var L=LINKS[lk], tau=2.5*JIT*1e-12, R=R0*P, h=HE/100;
      var ex=Math.pow(10,-L.a/10)*h*SYS, ey=Math.pow(10,-L.b/10)*h*SYS;
      var C=R*ex*ey, ratio=m*m*R*tau, A=C*ratio;
      var Q=(EOPT*C+0.5*A)/(C+A);
      var kf=1-FEC*H2(Q)-H2(Q);
      return { skr: kf>0 ? duty*0.5*(C+A)*kf : 0, Q:Q };
    }
    function full(P){ return calc(P,5,1); }
    function part(P){ return calc(P,2,0.5); }

    var X0=56, X1=624, Y0=18, Y1=274, P0=0.05, P1=1.0, LG0=0, LG1=3;
    function px(P){ return X0+(P-P0)/(P1-P0)*(X1-X0); }
    function py(v){
      var g=Math.log(Math.max(v,1e-6))/Math.LN10;
      g=Math.max(LG0,Math.min(LG1,g));
      return Y1-(g-LG0)/(LG1-LG0)*(Y1-Y0);
    }
    function unpx(x){ return P0+(x-X0)/(X1-X0)*(P1-P0); }

    var gGrid=document.getElementById('qwm-grid'),
        gCur=document.getElementById('qwm-curves'),
        gCr=document.getElementById('qwm-cross'),
        svg=root.querySelector('svg');

    var SUP=['⁰','¹','²','³','⁴','⁵','⁶','⁷','⁸','⁹'];
    function supStr(n){
      var t=''+Math.abs(n), o=n<0?'\u207b':'', i;
      for(i=0;i<t.length;i++) o+=SUP[+t[i]];
      return o;
    }
    function decLbl(lg){ return lg===0?'1':'10'+supStr(lg); }
    function fmtBps(v){
      if(v<=0) return 'no key';
      if(v<1) return v.toFixed(2)+' bps';
      if(v<10) return v.toFixed(1)+' bps';
      return Math.round(v)+' bps';
    }
    function setOv(id,r){
      var n=document.getElementById(id);
      clear(n);
      n.appendChild(document.createTextNode(fmtBps(r.skr)));
      var s=document.createElement('small');
      s.appendChild(document.createTextNode('  Q '+(r.Q*100).toFixed(1)+'%'));
      n.appendChild(s);
    }

    function autoscale(){
      /* three decades, topped just above whichever strategy is doing better,
         so the plot fills the same way in every condition */
      var vmax=0,P,v;
      for(P=P0;P<=P1+1e-9;P+=0.01){
        v=Math.max(full(P).skr,part(P).skr);
        if(v>vmax) vmax=v;
      }
      if(vmax<=0) vmax=1;
      LG1=Math.ceil(Math.log(vmax)/Math.LN10);
      LG0=LG1-3;
    }

    function grid(){
      clear(gGrid);
      var lg,y,d,x;
      for(lg=LG0;lg<=LG1;lg++){
        y=py(Math.pow(10,lg));
        gGrid.appendChild(el('line',{x1:X0,y1:y,x2:X1,y2:y,stroke:AXIS,'stroke-width':1,opacity:0.45}));
        gGrid.appendChild(el('text',{x:X0-7,y:y+3.5,fill:TICK,'font-size':9.5,'text-anchor':'end'},decLbl(lg)));
      }
      for(d=0.1;d<=1.001;d+=0.1){
        x=px(d);
        gGrid.appendChild(el('line',{x1:x,y1:Y0,x2:x,y2:Y1,stroke:AXIS,'stroke-width':1,opacity:0.22}));
        gGrid.appendChild(el('text',{x:x,y:Y1+15,fill:TICK,'font-size':9.5,'text-anchor':'middle'},d.toFixed(1)));
      }
      gGrid.appendChild(el('text',{x:(X0+X1)/2,y:Y1+31,fill:TICK,'font-size':10.5,'text-anchor':'middle'},'pump power, normalised'));
      gGrid.appendChild(el('text',{x:X0-40,y:(Y0+Y1)/2,fill:TICK,'font-size':10.5,'text-anchor':'middle',transform:'rotate(-90 '+(X0-40)+' '+((Y0+Y1)/2)+')'},'secret key rate (bps)'));
      gGrid.appendChild(el('line',{x1:X0,y1:Y1,x2:X1,y2:Y1,stroke:AXIS,'stroke-width':1.5}));
    }

    function pathOf(f){
      var p='',P,v,pen=false,FL=Math.pow(10,LG0);
      for(P=P0;P<=P1+1e-9;P+=0.005){
        v=f(P).skr;
        if(v<FL){ pen=false; continue; }
        p+=(pen?'L':'M')+px(P).toFixed(1)+' '+py(v).toFixed(1);
        pen=true;
      }
      return p;
    }

    function crossing(){
      var prev=null,P;
      for(P=P0;P<=P1+1e-9;P+=0.005){
        var d=full(P).skr-part(P).skr;
        if(prev!==null && ((prev>0&&d<0)||(prev<0&&d>0))) return P;
        prev=d;
      }
      return null;
    }

    function curves(){
      clear(gCur);
      gCur.appendChild(el('path',{d:pathOf(part),fill:'none',stroke:AMBER,'stroke-width':2,'stroke-linejoin':'round'}));
      gCur.appendChild(el('path',{d:pathOf(full),fill:'none',stroke:GREEN,'stroke-width':2,'stroke-linejoin':'round'}));
      var xc=crossing();
      if(xc!==null){
        var x=px(xc);
        gCur.appendChild(el('line',{x1:x,y1:Y0,x2:x,y2:Y1,stroke:'#dedede','stroke-width':1,'stroke-dasharray':'2 3',opacity:0.5}));
        gCur.appendChild(el('text',{x:x-5,y:Y0+11,fill:'#cfcfcf','font-size':9.5,'text-anchor':'end'},'they swap at '+xc.toFixed(2)));
      }
    }

    function cross(){
      clear(gCr);
      var P=hover===null?0.8:hover;
      var f=full(P), p=part(P), x=px(P);
      gCr.appendChild(el('line',{x1:x,y1:Y0,x2:x,y2:Y1,stroke:'#dedede','stroke-width':1,opacity:0.4}));
      if(f.skr>0) gCr.appendChild(el('circle',{cx:x,cy:py(f.skr),r:4,fill:GREEN,stroke:'#111','stroke-width':1.2}));
      if(p.skr>0) gCr.appendChild(el('circle',{cx:x,cy:py(p.skr),r:4,fill:AMBER,stroke:'#111','stroke-width':1.2}));
      document.getElementById('qwm-P').textContent=P.toFixed(2);
      setOv('qwm-f',f); setOv('qwm-p',p);

      var v,note;
      if(f.skr<=0){
        v='full mesh is over the QBER threshold';
        note='Five live channels per detector put this link’s accidental floor 25x above what it would carry alone, and at this jitter that is enough to push QBER past the point where any key survives error correction and privacy amplification. <b>The time-shared halves still work</b>, because two channels per detector is a factor of six less accidental noise and half the duty cycle is a cheap price for it. This is the paper’s challenging condition.';
      } else if(f.skr>p.skr){
        v='full mesh wins, by '+(f.skr/p.skr).toFixed(2)+'×';
        note='QBER has headroom here, so the extra accidentals the full mesh brings cost less than the half of the clock that time-sharing gives up. Lighting every link is simply more key. <b>This is the regime the paper reports for its favourable condition</b>, 15% heralding efficiency and under 100 ps of jitter, and it holds across almost the whole pump range.';
      } else {
        v='time-shared wins, by '+(p.skr/f.skr).toFixed(2)+'×';
        note='Past the crossing, pairs are arriving faster than the key fraction can absorb the accidentals that come with them, so the extra brightness is buying QBER rather than key. <b>Turning links off is now worth more than turning the pump up.</b> Note that the full-mesh curve turns over while the time-shared one is still climbing, which is the same effect the paper sees at its highest pump power.';
      }
      document.getElementById('qwm-v').textContent=v;
      document.getElementById('qwm-note').innerHTML=note;
    }

    function render(){ autoscale(); grid(); curves(); cross(); }

    function pt(e){
      var r=svg.getBoundingClientRect(), vb=svg.viewBox.baseVal;
      var cx=(e.touches?e.touches[0].clientX:e.clientX)-r.left;
      var x=cx/r.width*vb.width;
      if(x<X0||x>X1) return null;
      return unpx(x);
    }
    function move(e){
      var P=pt(e); if(P===null) return;
      hover=P; cross();
      if(e.touches) e.preventDefault();
    }
    svg.addEventListener('mousemove',move);
    svg.addEventListener('touchstart',move,{passive:false});
    svg.addEventListener('touchmove',move,{passive:false});
    svg.addEventListener('mouseleave',function(){ hover=null; cross(); });

    var slHE=document.getElementById('qwm-he'), slJ=document.getElementById('qwm-j');
    function syncLabels(){
      document.getElementById('qwm-hev').textContent=HE.toFixed(HE%1?1:0)+'%';
      document.getElementById('qwm-jv').textContent=JIT+' ps';
    }
    btnGroup(root,'lk',function(v){ lk=+v; render(); });
    btnGroup(root,'cd',function(v){
      if(v==='fav'){ HE=15; JIT=80; } else { HE=3; JIT=325; }
      slHE.value=HE; slJ.value=JIT; syncLabels(); render();
    });
    slHE.addEventListener('input',function(){ HE=+slHE.value; syncLabels(); render(); });
    slJ.addEventListener('input',function(){ JIT=+slJ.value; syncLabels(); render(); });
    syncLabels();
    render();
  })();
})();
</script>{% endraw %}