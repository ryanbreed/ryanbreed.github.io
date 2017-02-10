---
layout: post
date: 2017-02-08
published: true
title: "Mapping out Adversary COAs in Breached Network Operations"
description: "Developing Areas of Interest in the Operational Environment"
tags:
  - doctrine
  - COA
  - TTP
  - JIPOE
  - JCEO

excerpt_separator: <!--more-->

---
{% include JB/setup %}
I have been doing some reevaluation of my own methods for developing models
of instrumentation and analysis. My motivation for this has been driven
largely because existing mental models like the [Cyber Kill Chain] and the
[Diamond Model] have done more to confuse me than help me to sharpen the
conceptual approach. I'm looking for something better than a metaphor and
a metadata organizational structure when establishing a meaningful mapping
between semantic sensor data and the analytical activation function. This is
my first attempt at articulating this formulation, so please bear with me.

A defender has a limited budget of time, effort, and attention to spend on
instrumenting assets in her operational environment. She can maximize this
budget by anticipating adversary maneuvers within the operational environment
and focusing on some Areas of Interest where the adversary must operate.
Anticipating a particular advanced actor in the short term is impossible
without a tactical intelligence and surveillance capability, so we'll have to
do our best by deriving a likely operational picture from a higher-level
concept. Since hostile threat actors do a poor job of documenting and
publishing their doctrine, we'll use DoD doctrine as a proxy.

<!--more-->

Specifically, we're going to start from the perspective offered by the Joint
Concept for Entry Operations [JCEO]. I think there is strong external validity
as a proxy model for an adversary, specifically because it was developed to
address entry operations in the face of a pervasive [A2/AD] capability. This is
a key assumption (or a leap of faith) on the part of the defender: that
the marginal costs of our standard control package are low, and they deter or
deny the vast majority of threats from undetermined and non-strategic actors.
Also, that the defenses already deployed raise the cost of using sophisticated
offensive capabilities (from the loss of equities), and we want to ensure their
strategic effectiveness in the face of increasingly sophisticated operations.

So let's put on that black hat and think like an operations bureaucrat. I'd
actually recommend going a couple levels deep into the [JOAC] and [JIPOE] if
you can stomach it. There's a logical flow that gets us to computer stuff, but
I want to make sure the framing is comprehensive and it produces correct results
before inflicting it on anyone, so hang on for the payoff.

The JCEO has a handy flow chart for entry operations, so let's start with that
picture in our head:

 <a href="/assets/images/jceo_flowchart.png">
   <img height="800" width="600" src="/assets/images/jceo_flowchart.png"/>
 </a>

Here's as much as I can boil down in one go:

* we are a strategic actor with a specific objective and target
* we have surveillance and intelligence on the target that lets us decide
  1. we can achieve insertion at the objective and then exit
  2. we will need to secure lodgement and then maneuver to the objective
  3. we will need to secure lodgement and then transition to a separate
     long-term operating force before determining the next action on the
     objective
* we're going to manage risk by
  1. minimizing loss of logistical basing (or minimizing the cost of its loss)
  2. using a minimum of force in offensive maneuver
  3. exploiting cross-domain effects to achieve offensive dominance
  4. protecting critical enablers by operating from concealment
* we understand the time-value-loss tradeoff for escalating force. once
   lodgement is established at least one time:
  1. time is less costly than loss of a strategic capability
  2. we have a direct tactical surveillance and intelligence capability that
     lets us anticipate strategic shifts by the defender
  3. we can iterate on subsequent operations, even without guaranteed reentry
     via an established lodgement or a previously-exploited vulnerability
* we understand the defenders reserve capacity to disrupt operations
  1. once alerted, the defender will surge indefinitely until depletion
  2. the defender will satisfice a decision to stop surging
* it is the *strategic capacity to overwhelm the defender* that will guarantee
   future reentry and follow-on operations

Here are some choice excerpts and commentary, slightly mangled to improve
readability:

> *Entry forces will envelop, infiltrate and penetrate in and/or across
> multiple domains at select points of entry to place the enemy at an
> operational disadvantage*.

Takeaway: Once targeted, you're not just fucked, you're *really fucked*.

> Maneuver capabilities in multiple domains present many potential threats to
> the adversary, overloading his decision cycle

Takeaway: You can expect simultaneous operation across different surfaces as
the rule and not the exception. The aggressor's capacity to dominate is
dependent on the defender's tendency to fixate and satisfice.

> In response to a review of Joint Force strengths and weaknesses, an adversary
> will be compelled to consider which investments in technology and time afford
> it the best opportunity for success. It may prepare for all possible forms of
> entry, or it may focus on just one.
> ... the adversary can be made vulnerable by exposing weakened defenses in
> one or more domains, allowing the Joint Force to achieve local domain
> superiority at one or more points of our choosing.

Takeaway: It is the hyperextension or over-concentration of defense that causes vulnerability.

> *For example, if the enemy focuses on the periphery, the Joint Force will
> attack in-depth. If the enemy defends in-depth, the Joint Force will
> concentrate at critical points. When the enemy focuses in any domain,
> the Joint Force will capitalize on defeating him in another where he is
> weaker.*

Corollary: The best defense is an absence of something defensible. Downsize before adding capability.

> ... the Joint Force will maximize surprise through deception, stealth, and
> ambiguity to counter adversary ISR and complicate targeting. ... *Where
> surprise is not possible ... the Joint Force will seek to overwhelm the
> enemy’s targeting capability.*

Takeaway: your monitoring and alerting are actually doing double duty to
conceal adversary maneuver and provide operational camouflage.

> *In general, Follow-on Forces require some form of RSO&I activities before
> they are able to conduct operations*

Takeaway: remote c2 and logistics are critical vulnerabilities in a sustained
campaign.

> A Joint Force that uses an enterprise approach to standardize DOD C2 protocols
> and systems will enable better interoperability. This interoperability allows
> rapid expansion and synchronization of C2

Takeaway: re-use, modularity, and patterns of composition are the
hallmarks of offensive c2 capabilities.

> An agile network can expand and contract when needed as well as mitigate the
> loss of nodes or commanders within the network.

Takeaway: patterns of composition in TTPs reduce risks and lower the marginal
costs of each operation. When you find one of these, you can expect its users
to be prolific operators.


* The Joint Operational Access Concept (JOAC)
* Joint Forcible Entry Operations(JP 3-18)
* Joint Intelligence Preparation of the Operational Environment (JP 2-01.3)
* Joint Concept for Entry Operations (JCEO)
