---
layout: post
section-type: post
title: "The fourth plane : what a quantum testbed runs instead of a script"
category: 'networking'
tags: [ 'networking', 'quantum' ]
---

disclosure first, same as [the aerial fiber post]({% post_url 2026-07-20-holding-entanglement-together-on-a-wire-in-the-wind %}): i am on this one. [Amlou et al., "A Measurement Plane for Quantum Networking," arXiv:2607.13291](https://arxiv.org/abs/2607.13291), NIST and Université Grenoble Alpes. i am one of nine authors and not the first, so most of the design credit belongs elsewhere, but i am inside it and cannot review it from outside. what does not change is the format. there is still a section at the end about what i would push on, and this time most of it is aimed at my own paper.

the reason to write it up here is that this blog has spent two posts complaining about the thing it is about.

[the quantum LAN post]({% post_url 2026-09-07-a-quantum-lan-on-deployed-fiber %}) ended by arguing that ebits per second deserves to be the metric of a distribution network and is losing, because nobody states an integration time, a basis set, or a position on accidental subtraction. [the flex grid post]({% post_url 2026-04-04-Flex-grid-for-entanglement %}) asked, twice now, for somebody to publish a reconfiguration time. and [the post on the Bristol q-ROADM]({% post_url 2026-09-10-fifteen-links-at-once %}) last week ran into the same wall from the other side: a week of per-link data on installed fiber, and the interesting time series is not in the paper.

those are all the same complaint. the measurements exist. what does not exist is a way to say what was measured, in a form somebody else can run.

## what a plane is

this word comes from telecom and it is worth ten seconds if you have not met it.

a network element is usually described as three *planes*, which are not physical parts but separations of concern.

- the **data plane** moves user traffic. it is the fast path, the thing the ASIC does.
- the **control plane** decides where traffic goes. routing protocols, label distribution, the thing that computes the forwarding table the data plane uses.
- the **management plane** configures and monitors the box. SNMP, NETCONF, the CLI you log into, the counters you scrape.

the split is the reason a router is buildable. each plane has its own protocols, its own failure modes, and its own rate of change, and a person can work on one without holding the other two in their head. SDN was, among other things, an argument about where the line between the first two should sit.

quantum networking has been importing this vocabulary for a few years. there is a data plane, which is the entanglement itself. there are control planes: [QUANT-NET](https://quantnet.lbl.gov/) has a two-level one, ArQNet has a coordinator built on SDN principles, and QNodeOS proposes an operating-system abstraction for the node. this is a healthy amount of work and the vocabulary transferred cleanly.

the paper's claim is that measurement does not fit in any of the three, and should be its own plane alongside them rather than a feature bolted onto one.

## why measurement does not fit

the argument has two halves and i think the second one is the interesting one.

the first half is that everybody is writing scripts. measurement in quantum testbeds today lives inside device-specific control software and ad hoc scripts. that couples the experiment to the hardware it happened to run on, makes anything reusable a rewrite, and makes a distributed multi-step procedure into a person with four terminal windows and a lab notebook. this is a familiar complaint from classical networking, and classical networking answered it: perfSONAR and RIPE Atlas deploy measurement as distributed services, and mPlane wrote down a capability model for describing what a measurement device can do.

so far, so ordinary. if that were the whole argument the answer would be "run perfSONAR."

**the second half is why you cannot.** the classical platforms are built around *independent observations*. a probe pings a target, records a latency, ships the record. probes do not need to agree with each other about anything except roughly what time it is, and no probe's result depends on what another probe was doing at that instant.

a quantum measurement is not shaped like that. to measure a polarization fringe you must set an analyzer at Alice, set another at Bob, have both time taggers agree to nanoseconds, acquire in the same window, and then compute a quantity that **only exists jointly**. there is no such thing as Alice's half of a coincidence. the paper puts this as needing "synchronized state preparation, multiple-device participation, and phase-dependent feedback," and the shortest version is that the unit of measurement spans nodes.

that is the gap. classical measurement infrastructure has the distribution and the programmability and assumes the observations are independent. quantum testbeds have the joint observations and no infrastructure. the paper is an attempt to take the first and add the second.

## the four layers

the architecture is four layers, and they are worth naming carefully because the useful idea is in the middle two.

**resource agents** sit next to the hardware. one per device, containerized, translating a specific instrument's API and data format into a common message interface. a time tagger agent hands out timestamps; a polarization controller agent takes an angle and moves a motor. this is a driver layer, and its job is to be the only place in the system that knows what brand anything is.

**capabilities** turn device access into named operations. a capability is described by a label, the endpoint that provides it, a name, **a schema for its parameters, a schema for its results**, and metadata. crucially they compose: a coincidence capability consumes timestamps from two remote time-tagger agents, and a polarization-analysis capability wraps the coincidence capability and adds the analyzer settings.

**experiment coordination** turns capabilities into procedures. which capability, in what order, with what parameters, carrying state and results between steps. a polarization-fringe workflow is a loop that sweeps an analyzer angle and invokes polarization-analysis at each setting.

the *application* layer is where a person says what they want: a Python client library and a web GUI.

the implementation is Docker containers talking over NATS, with JSON schemas on the messages, Python underneath, and REST plus messaging for the GUI. none of that is exotic and all of it is the point: this is deliberately a boring distributed system, which is the correct thing for a layer whose job is to still be running in three years.

### the part i would underline

the capability descriptor is the load-bearing idea, and it is easy to read past because it looks like plumbing.

a capability publishes the schema of what it accepts and the schema of what it returns. that means "coincidence" stops being a word in a methods section and becomes a thing with declared inputs. what integration time. what window. what basis. what it does about accidentals.

hold that thought until the last section, because it is the closest anything has come to the reference procedure the LAN post was asking for, and the paper does not claim it.

## one fringe, all the way down

the clearest way to see what the layers buy you is to watch a single experiment go down through them and come back.

<div class="qw" id="qw-mp" data-qw><div class="qw-hd"><span class="qw-t">a polarization fringe, layer by layer</span><span class="qw-s">one workflow submitted at the top, decomposed into capability calls and device operations, with the fringe assembling as each point returns.</span></div><div class="qw-ctl"><div class="qw-g"><span class="qw-l">Alice's fixed analyzer</span><span class="qw-b" data-grp="al"><button type="button" data-v="-45">&minus;45&deg;</button><button type="button" data-v="0" class="on" aria-pressed="true">0&deg;</button><button type="button" data-v="45">45&deg;</button><button type="button" data-v="90">90&deg;</button></span></div><div class="qw-g"><span class="qw-l">run</span><span class="qw-b"><button type="button" id="qwmp-play">play</button><button type="button" id="qwmp-step">step</button><button type="button" id="qwmp-reset">reset</button></span></div></div><div class="qw-two"><div class="qw-pane"><span class="qw-pl">the measurement plane</span><svg class="qw-svg" viewBox="0 0 320 250" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Four-layer measurement plane with a message moving between layers"><g id="qwmp-stack"></g><g id="qwmp-msg"></g></svg></div><div class="qw-pane"><span class="qw-pl">what comes back</span><svg class="qw-svg" viewBox="0 0 320 250" preserveAspectRatio="xMidYMid meet" role="img" aria-label="Polarization fringe assembling point by point"><g id="qwmp-plot"></g></svg></div></div><div class="qw-out"><div class="qw-oi"><span class="qw-ok">Bob's analyzer</span><span class="qw-ov" id="qwmp-tb">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">coincidences</span><span class="qw-ov" id="qwmp-cc">&mdash;</span></div><div class="qw-oi"><span class="qw-ok">visibility so far</span><span class="qw-ov" id="qwmp-vis">&mdash;</span></div><div class="qw-oi qw-grow"><span class="qw-ok">current message</span><span class="qw-ov qw-txt" id="qwmp-step-lbl">idle</span></div></div><p class="qw-n" id="qwmp-note">Press play. One workflow description goes in at the application layer; everything below it is the coordinator invoking a capability once per analyzer setting, that capability calling two resource agents in different buildings, and a coincidence being computed from two timestamp streams that only mean something together.</p><noscript><p class="qw-n">A polarization-fringe workflow is submitted once at the application layer. The coordinator then loops over Bob's analyzer angles; at each one it invokes a polarization-analysis capability, which sets both rotators through their resource agents, acquires timestamps from both time taggers, and computes coincidences in a 1 ns window. Each returned point is one dot on the fringe. In the paper's run the fringe came back with 98 percent visibility.</p></noscript></div>

the thing i want to draw attention to is how little of that sequence is about physics. one workflow description goes in. what comes back is a fringe. everything between is dispatch, schema validation, two motor moves, two timestamp acquisitions and one join, and in the version this replaces, all of it was a script that only ran on that bench.

## what it actually ran

the testbed is two nodes in separate buildings on the NIST campus, about 1.4 km of fiber between them. an SPDC source at Alice, single photon detectors at both ends, motorized polarization analyzers at both ends, and time taggers synchronized over White Rabbit. each node runs a local server hosting its resource agents.

two experiments.

**coincidence detection.** pull synchronized timestamps from both time-tagger agents, histogram them with a 1 ns coincidence window, find the peak. the raw streams sat about **7.1 µs** apart from path and system delays, corrected in software, and after that there is a clean peak at zero delay. the claim attached to this is not the peak, which is unremarkable; it is that the analysis ran **online**, during the experiment, instead of on files afterwards.

**a polarization fringe.** Alice fixed at each of −45°, 0°, 45° and 90°; Bob swept. the coordinator walked the settings, invoking the polarization-analysis capability at each pair, and the fringe came back at **98% visibility**.

and the number the paper puts on the improvement is that experiment-specific logic went from **hundreds of lines of script to a short workflow specification**.

that is the whole result. it is a feasibility paper and it says so: "this work demonstrates feasibility rather than scale limits."

## what i'd push on next

so, my own paper. five things, roughly in order of how much i think they matter.

**1. a measurement plane that does not measure itself.** the headline improvement is "hundreds of script lines to a short workflow specification," and that is not a measurement. it is the kind of claim the paper exists to make unnecessary. what is the end-to-end latency of one capability invocation. how long did the fringe sweep take, against how long the same sweep took by hand. what is the overhead of NATS plus JSON schema validation against a direct device call. those numbers are three afternoons of work with the system already running, and their absence is conspicuous in this particular paper in a way it would not be in somebody else's.

**2. close a loop, and pick the loop by its deadline.** the paper says the architecture supports online measurement and feedback, and both demonstrations are open-loop sweeps. that gap is honest but it is the gap. and there is a specific loop to close, sitting in the group's own data: on the [62 km aerial link]({% post_url 2026-07-20-holding-entanglement-together-on-a-wire-in-the-wind %}), daytime polarization drift takes fidelity below 95% in **under 20 seconds**. so a controller that corrects polarization from a live measurement has a budget of a few seconds for the whole path, measure to actuate. does this architecture fit inside that? nobody knows, because of item 1. **the latency number is not a benchmark, it is a feasibility criterion for the thing the plane is for.**

**3. make the capability descriptor into the reference procedure.** this is the one i would actually build, and it is why i wanted to write this post.

the LAN post's complaint was that ebits per second is losing as a metric because there is no stated procedure behind it: no integration time, no basis set, no declared position on accidental subtraction. the fix i asked for was a spec.

a capability descriptor is already 80% of that spec. it has a name, a parameter schema and a result schema. if the coincidence capability's schema **required** an integration time, a basis set and an accidentals policy, and if the capability carried a version, then a paper could report "ebits/s, coincidence capability v1.2, parameters as attached" and a reader could run the identical procedure on different hardware. the metric would stop being a word and become an executable.

that costs a registry and a versioning convention, both of which the paper explicitly does not have. it is a small amount of work with a much larger payoff than another supported instrument.

**4. three nodes, because two is not a network.** two nodes is the smallest thing on which a joint measurement exists at all, and it does prove the point. but the coordination problems that motivated the whole design turn up at three: an entanglement swap at a middle node is a workflow where one participant's result gates what the other two do, and the coordinator has to carry a herald between them rather than a parameter. that is qualitatively harder than a sweep and it is the case the architecture claims to be for.

<!-- **5. say what happens when a node goes away.** the lifecycle in the paper runs submitted, accepted, running, completed, interrupted. "interrupted" is doing a lot of work there. a fringe sweep is nineteen dependent steps across two buildings, and the failure question is not academic: on a week-long run like [the Bristol one]({% post_url 2026-09-10-fifteen-links-at-once %}), an SNSPD cryostat cycles on schedule and a fiber gets touched. what does a workflow do when a resource agent stops answering halfway through, and does the partial result survive with enough metadata to be worth keeping? the classical measurement platforms this borrows from all had to answer that, and it is usually the least glamorous and most-used part of the system. -->

## the short version

- telecom splits a network element into data, control and management planes. quantum networking has imported all three, and has control-plane work in QUANT-NET, ArQNet and QNodeOS.
- **measurement does not fit in any of them**, because it is currently ad hoc scripts glued to specific instruments, and because a quantum measurement spans nodes in a way a classical probe does not.
- there is no such thing as Alice's half of a coincidence. that single fact is why perfSONAR's model does not transfer, and it is the argument for a fourth plane.
- four layers: resource agents next to the hardware, capabilities as named composable operations with schemas, a coordinator that sequences them, and an application layer where a person describes what they want.
- Docker, NATS, JSON schemas, Python. deliberately boring, correctly so.
- validated on two NIST buildings, 1.4 km of fiber, White Rabbit timing, a 1 ns coincidence window and a 7.1 µs offset corrected in software. the polarization fringe came back at 98% visibility.
- the reported win is hundreds of lines of script becoming a short workflow specification, which is a real win and is not a number.
- **the capability descriptor is the interesting object.** parameter and result schemas are most of what a citable measurement procedure needs, and adding a version and a registry would turn ebits per second from a word into something reproducible.
- it is a feasibility paper and says so. two nodes, no closed loop, no latency figures, and no story about what happens when a node stops answering.

writing about a paper you are on is uncomfortable in a specific way: the things you would criticise are the things you know are on somebody's list. so the fair summary is that the architecture is right and the evidence is thin, and the thinness is fixable with the system that already exists. the part i would defend against anybody is the diagnosis. every complaint this blog has made about missing numbers in the last six months has the same root, which is that a measurement in this field is a paragraph in a methods section rather than a thing you can invoke. that is worth a plane.

{% raw %}<style>
.qw{text-align:left;background:#191919;border:1px solid #333;border-radius:6px;padding:18px 18px 14px;margin:28px 0;font-family:"Open Sans","Helvetica Neue",Helvetica,Arial,sans-serif;font-size:13px;line-height:1.5;color:#e6e6e6;touch-action:manipulation;-webkit-tap-highlight-color:transparent}
.qw *{box-sizing:border-box}
.qw-hd{margin-bottom:14px}
.qw-t{display:block;font-size:14px;font-weight:600;color:#fff;letter-spacing:.01em}
.qw-s{display:block;font-size:12px;color:#8f8f8f;margin-top:3px}
.qw-ctl{display:flex;flex-wrap:wrap;gap:18px;margin-bottom:12px}
.qw-g{display:flex;flex-direction:column;gap:5px}
.qw-l{font-size:11px;text-transform:uppercase;letter-spacing:.08em;color:#8f8f8f}
.qw-b{display:inline-flex;flex-wrap:wrap;gap:4px}
.qw-b button{font:inherit;font-size:12px;line-height:1;padding:7px 11px;background:#242424;color:#c8c8c8;border:1px solid #3a3a3a;border-radius:4px;cursor:pointer;transition:background .12s,color .12s,border-color .12s}
.qw-b button:hover{background:#2e2e2e;color:#fff}
.qw-b button.on{background:#32c29e;border-color:#32c29e;color:#10231f;font-weight:600}
.qw-b button:focus-visible{outline:2px solid #32c29e;outline-offset:2px}
.qw-svg{display:block;width:100%;height:auto;overflow:visible}
.qw-two{display:flex;flex-wrap:wrap;gap:14px}
.qw-pane{flex:1 1 270px;min-width:0;margin-bottom:6px}
.qw-pl{display:block;font-size:11px;text-transform:uppercase;letter-spacing:.08em;color:#8f8f8f;margin-bottom:4px}
.qw-out{display:flex;flex-wrap:wrap;gap:8px;margin-top:12px}
.qw-oi{flex:0 1 auto;min-width:118px;background:#212121;border:1px solid #303030;border-radius:4px;padding:7px 10px}
.qw-grow{flex:1 1 240px}
.qw-ok{display:block;font-size:10px;text-transform:uppercase;letter-spacing:.08em;color:#8f8f8f}
.qw-ov{display:block;font-family:ui-monospace,SFMono-Regular,Menlo,Consolas,monospace;font-size:14px;color:#fff;margin-top:2px;font-variant-numeric:tabular-nums}
.qw-ov.qw-txt{font-family:inherit;font-size:12.5px;font-weight:600}
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

  var GREEN='#32c29e', AMBER='#f2a03d', AXIS='#3a3a3a', TICK='#7d7d7d',
      BOX='#212121', BOXON='#1d3b34', EDGE='#3a3a3a', EDGEON=GREEN;

  /* =======================================================================
     WIDGET - one polarization fringe, layer by layer

     The left pane is the paper's four layers.  The right pane is the fringe
     the workflow produces.  Each measurement point costs four messages:

        coordinator -> capability      invoke polarization-analysis(tA, tB)
        capability  -> resource agents set both rotators, acquire
        agents      -> capability      two timestamp streams
        capability  -> coordinator     one coincidence count

     The counts come from the standard |Phi+> correlation,

        C(tA, tB) = C0 . [ 1 + V cos(2(tB - tA)) ] / 2

     with V = 0.98, the visibility the paper's fringe came back at, plus
     Poisson noise from a seeded generator so the same run repeats.  The
     point of the animation is the message sequence, not the physics: the
     coincidence at the end exists only because two agents in two buildings
     acquired inside the same window.
     ==================================================================== */
  (function(){
    var root=document.getElementById('qw-mp'); if(!root) return;

    var C0=800, VIS=0.98, STEPDEG=15, TA=0;
    var svgL=root.querySelectorAll('svg')[0], svgR=root.querySelectorAll('svg')[1];
    var gStack=document.getElementById('qwmp-stack'), gMsg=document.getElementById('qwmp-msg'),
        gPlot=document.getElementById('qwmp-plot');

    var LAYERS=[
      {id:'app',   y:14,  t:'application',            s:'a person describes the run'},
      {id:'coord', y:62,  t:'experiment coordination',s:'sequences the capability calls'},
      {id:'cap',   y:110, t:'capability',             s:'polarization-analysis → coincidence'}
    ];
    var AG=[
      {id:'alice', x:20,  t:'Alice agents', s:'rotator · time tagger'},
      {id:'bob',   x:168, t:'Bob agents',   s:'rotator · time tagger'}
    ];
    var BW=280, BH=34, AGW=132, AGY=160, AGH=42;
    var POS={app:[160,31],coord:[160,79],cap:[160,127],alice:[86,181],bob:[234,181]};

    function rng(seed){ return function(){ seed|=0; seed=seed+0x6D2B79F5|0;
      var t=Math.imul(seed^seed>>>15,1|seed); t=t+Math.imul(t^t>>>7,61|t)^t;
      return ((t^t>>>14)>>>0)/4294967296; }; }
    function gauss(r){ var u=Math.max(1e-9,r()), v=r();
      return Math.sqrt(-2*Math.log(u))*Math.cos(2*Math.PI*v); }

    var angles=[], steps=[], pts=[], idx=0, frac=0, timer=null, rand=null;

    function ideal(tb){ return C0*(1+VIS*Math.cos(2*(tb-TA)*Math.PI/180))/2; }

    function build(){
      angles=[]; for(var a=0;a<=180;a+=STEPDEG) angles.push(a);
      rand=rng(1337+TA);
      steps=[{f:'app',to:'coord',lbl:'submit workflow: Alice '+TA+'°, sweep Bob 0–180°',k:'submit'}];
      angles.forEach(function(tb,i){
        steps.push({f:'coord',to:'cap', lbl:'invoke polarization-analysis(A='+TA+'°, B='+tb+'°)',k:'invoke',tb:tb});
        steps.push({f:'cap',  to:'both',lbl:'set both rotators, acquire in a common window',k:'acquire',tb:tb});
        steps.push({f:'both', to:'cap', lbl:'two timestamp streams return',k:'stream',tb:tb});
        steps.push({f:'cap',  to:'coord',lbl:'coincidences in a 1 ns window',k:'result',tb:tb,pt:i});
      });
      steps.push({f:'coord',to:'app',lbl:'fringe complete',k:'done'});
      pts=[]; idx=0; frac=0;
    }

    function box(g,x,y,w,h,title,sub,on){
      g.appendChild(el('rect',{x:x,y:y,width:w,height:h,rx:4,
        fill:on?BOXON:BOX,stroke:on?GREEN:'#333','stroke-width':on?1.6:1}));
      g.appendChild(el('text',{x:x+9,y:y+15,fill:on?'#eafff8':'#d6d6d6','font-size':10.5,'font-weight':600},title));
      g.appendChild(el('text',{x:x+9,y:y+27,fill:'#8f8f8f','font-size':9},sub));
    }

    function edge(g,a,b,on){
      g.appendChild(el('line',{x1:a[0],y1:a[1],x2:b[0],y2:b[1],
        stroke:on?EDGEON:EDGE,'stroke-width':on?1.8:1,opacity:on?0.95:0.5}));
    }

    function activeSet(){
      var s=steps[idx]; if(!s) return {};
      var m={}; m[s.f]=1;
      if(s.to==='both'){ m.alice=1; m.bob=1; } else m[s.to]=1;
      if(s.f==='both'){ m.alice=1; m.bob=1; }
      return m;
    }

    function drawStack(){
      clear(gStack); clear(gMsg);
      var act=activeSet(), s=steps[idx];
      var live={};
      if(s){
        if(s.f==='cap'&&s.to==='both'){ live['cap-alice']=1; live['cap-bob']=1; }
        else if(s.f==='both'&&s.to==='cap'){ live['cap-alice']=1; live['cap-bob']=1; }
        else live[s.f+'-'+s.to]=1;
      }
      edge(gStack,[160,48],[160,62],live['app-coord']||live['coord-app']);
      edge(gStack,[160,96],[160,110],live['coord-cap']||live['cap-coord']);
      edge(gStack,[160,144],[86,AGY],live['cap-alice']);
      edge(gStack,[160,144],[234,AGY],live['cap-bob']);

      LAYERS.forEach(function(L){ box(gStack,20,L.y,BW,BH,L.t,L.s,act[L.id]); });
      AG.forEach(function(A){ box(gStack,A.x,AGY,AGW,AGH,A.t,A.s,act[A.id]); });

      gStack.appendChild(el('line',{x1:152,y1:AGY+AGH/2,x2:168,y2:AGY+AGH/2,
        stroke:'#5b8def','stroke-width':1.2,'stroke-dasharray':'3 3'}));
      gStack.appendChild(el('text',{x:160,y:AGY+AGH+16,fill:'#7d7d7d','font-size':9,'text-anchor':'middle'},
        '1.4 km of fibre · White Rabbit timing'));

      if(!s) return;
      function dot(a,b){
        var f=Math.min(1,frac);
        gMsg.appendChild(el('circle',{cx:a[0]+(b[0]-a[0])*f,cy:a[1]+(b[1]-a[1])*f,
          r:4.5,fill:s.k==='result'||s.k==='stream'?AMBER:GREEN,stroke:'#111','stroke-width':1}));
      }
      if(s.to==='both'){ dot(POS.cap,POS.alice); dot(POS.cap,POS.bob); }
      else if(s.f==='both'){ dot(POS.alice,POS.cap); dot(POS.bob,POS.cap); }
      else dot(POS[s.f],POS[s.to]);
    }

    var PX0=48, PX1=306, PY0=18, PY1=206;
    function px(a){ return PX0+a/180*(PX1-PX0); }
    function py(c){ return PY1-c/(C0*1.1)*(PY1-PY0); }

    function drawPlot(){
      clear(gPlot);
      var i,x,y;
      for(i=0;i<=4;i++){
        y=PY1-(i/4)*(PY1-PY0);
        gPlot.appendChild(el('line',{x1:PX0,y1:y,x2:PX1,y2:y,stroke:AXIS,'stroke-width':1,opacity:0.35}));
        gPlot.appendChild(el('text',{x:PX0-6,y:y+3.5,fill:TICK,'font-size':9,'text-anchor':'end'},
          Math.round(i/4*C0*1.1)+''));
      }
      for(i=0;i<=180;i+=45){
        x=px(i);
        gPlot.appendChild(el('line',{x1:x,y1:PY0,x2:x,y2:PY1,stroke:AXIS,'stroke-width':1,opacity:0.25}));
        gPlot.appendChild(el('text',{x:x,y:PY1+14,fill:TICK,'font-size':9,'text-anchor':'middle'},i+'°'));
      }
      gPlot.appendChild(el('text',{x:(PX0+PX1)/2,y:PY1+30,fill:TICK,'font-size':10,'text-anchor':'middle'},
        "Bob's analyzer angle"));
      gPlot.appendChild(el('text',{x:PX0-36,y:(PY0+PY1)/2,fill:TICK,'font-size':10,'text-anchor':'middle',
        transform:'rotate(-90 '+(PX0-36)+' '+((PY0+PY1)/2)+')'},'coincidences'));

      if(pts.length===angles.length){
        var d='',a;
        for(a=0;a<=180;a+=2) d+=(a?'L':'M')+px(a).toFixed(1)+' '+py(ideal(a)).toFixed(1);
        gPlot.appendChild(el('path',{d:d,fill:'none',stroke:GREEN,'stroke-width':1.2,
          'stroke-dasharray':'4 3',opacity:0.45}));
      }
      var s=steps[idx];
      if(s&&s.tb!==undefined){
        gPlot.appendChild(el('line',{x1:px(s.tb),y1:PY0,x2:px(s.tb),y2:PY1,
          stroke:'#dedede','stroke-width':1,opacity:0.35}));
      }
      pts.forEach(function(p){
        gPlot.appendChild(el('circle',{cx:px(p.a),cy:py(p.c),r:3.2,fill:GREEN,stroke:'#111','stroke-width':0.8}));
      });
      gPlot.appendChild(el('line',{x1:PX0,y1:PY1,x2:PX1,y2:PY1,stroke:AXIS,'stroke-width':1.4}));
    }

    var NOTES={
      submit:'One description goes in at the top: which analyzer settings, what to sweep, what to return. Everything below this line is the plane doing the work, and none of it is in the user’s script.',
      invoke:'The coordinator picks the capability and the parameters for this point. A capability publishes <b>a schema for what it accepts and a schema for what it returns</b>, so this call is validated rather than hoped for.',
      acquire:'The capability reaches two resource agents in two different buildings: set a rotator, start acquiring. The agents are the only components that know what brand the hardware is.',
      stream:'Two timestamp streams come back. Neither is a measurement on its own. <b>There is no such thing as Alice’s half of a coincidence</b>, which is exactly why the classical measurement platforms this borrows from do not transfer unchanged.',
      result:'The coincidence is computed from both streams in a 1 ns window and returned as one point. In the paper this ran online, during the experiment, instead of on files afterwards.',
      done:'The fringe is complete. The paper’s came back at <b>98% visibility</b>, and the reported win is that the experiment-specific logic went from hundreds of lines of script to a short workflow specification.',
      idle:'Press play. One workflow description goes in at the application layer; everything below it is the coordinator invoking a capability once per analyzer setting, that capability calling two resource agents in different buildings, and a coincidence being computed from two timestamp streams that only mean something together.'
    };

    function readouts(){
      var s=steps[idx];
      document.getElementById('qwmp-step-lbl').textContent=s?s.lbl:'run complete';
      document.getElementById('qwmp-tb').textContent=(s&&s.tb!==undefined)?s.tb+'°':'—';
      var last=pts.length?pts[pts.length-1]:null;
      document.getElementById('qwmp-cc').textContent=last?Math.round(last.c):'—';
      var v='—';
      if(pts.length>=4){
        var mx=-1,mn=1e9;
        pts.forEach(function(p){ if(p.c>mx)mx=p.c; if(p.c<mn)mn=p.c; });
        v=(mx+mn>0)?((mx-mn)/(mx+mn)*100).toFixed(1)+'%':'—';
      }
      document.getElementById('qwmp-vis').textContent=v;
      document.getElementById('qwmp-note').innerHTML=NOTES[s?s.k:'done']||NOTES.idle;
    }

    function render(){ drawStack(); drawPlot(); readouts(); }

    function commit(){
      var s=steps[idx];
      if(s&&s.pt!==undefined){
        var mu=ideal(s.tb), c=Math.max(0,mu+gauss(rand)*Math.sqrt(mu));
        pts.push({a:s.tb,c:c});
      }
      idx++;
      if(idx>=steps.length){ idx=steps.length-1; stop(); }
      frac=0;
    }
    function advance(){
      frac+=0.25;
      if(frac>=1) commit();
      render();
    }
    function stop(){ if(timer){ clearInterval(timer); timer=null; } setPlay(false); }
    function setPlay(on){ document.getElementById('qwmp-play').textContent=on?'pause':'play'; }
    function play(){
      if(timer){ stop(); return; }
      if(idx>=steps.length-1) reset();
      setPlay(true);
      timer=setInterval(advance,55);
    }
    function reset(){ stop(); build(); render(); }

    document.getElementById('qwmp-play').addEventListener('click',play);
    document.getElementById('qwmp-step').addEventListener('click',function(){
      stop(); frac=1; commit(); render();
    });
    document.getElementById('qwmp-reset').addEventListener('click',reset);
    btnGroup(root,'al',function(v){ TA=+v; reset(); });

    build(); render();
  })();
})();
</script>{% endraw %}