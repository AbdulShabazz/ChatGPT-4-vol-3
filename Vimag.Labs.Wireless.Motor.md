## OpenAI ChatGPT [Highest-Thinking] + v1.2026.213.1825 Vimag.Labs.Wireless.Motor.md

<b>Q: Explain the wireless motor technology at Vimag Labs!</b>

**A:** Vimag Labs’ “wireless motor” is best understood as a **brushless wound-field synchronous motor whose rotor receives its excitation power inductively through a rotating transformer**. The clever part is eliminating both permanent magnets and the brushes/slip rings normally needed to energize an electromagnetic rotor. ([patents.google.com][1])

### How it works

The power path is approximately:

```text
Battery / DC bus
      │
      ▼
Main traction inverter ──► Main stator windings
                                  │
                                  │ magnetic torque
                                  ▼
                           Electromagnetic rotor
                                  ▲
                                  │ DC excitation
                           Rotor rectifier
                                  ▲
                                  │ AC
                      Rotating transformer rotor
                           )) air gap ((
                      Rotating transformer stator
                                  ▲
                                  │
                      Excitation inverter
```

The **main stator** operates much like the stator of a conventional Permanent Magnet Synchronous Motor (PMSM), producing a rotating magnetic field. But instead of neodymium magnets providing the rotor field, Vimag puts **DC field windings on the rotor**. ([Google Patents][1])

The problem is then: how do you deliver electrical current onto something spinning at thousands of revolutions per minute without brushes?

Vimag adds a **rotating transformer**. One transformer winding remains stationary and another rotates with the shaft. Alternating Current (AC) applied to the stationary winding crosses a small physical air gap through magnetic induction, producing AC in the rotating winding. A rotor-mounted three-phase rectifier converts that AC into DC and feeds the rotor field coils. There is therefore **no electrical contact across the stationary-to-rotating interface**. ([Google Patents][1])

That is what Vimag means by **wireless power transfer**.

### The “virtual magnet”

Once DC flows through the rotor winding, the rotor becomes an electromagnet:

[
\text{rotor field strength} \propto N I_f
]

where (N) is the field-winding turns and (I_f) is the rotor excitation current.

A permanent-magnet rotor effectively gives you a relatively fixed magnetic flux. Vimag instead makes (I_f) controllable.

So software can command something conceptually like:

```text
Low torque demand       → weak rotor field
High torque demand      → stronger rotor field
High-speed operation    → reduce field
Thermal constraint      → reduce/reshape excitation
Regenerative braking    → select appropriate field
```

This variable electromagnetic rotor field is what the company calls its **“Virtual Magnet.”** The software is not literally synthesizing magnetism computationally; it is controlling power electronics that create a physical magnetic field in copper windings. Vimag's patent specifically describes controlling rotor DC current through the rotating-transformer stator, including closed-loop estimation without requiring a rotor current sensor. ([Google Patents][1])

### Why this is interesting

| Architecture                              | Rotor field source      | Rotor electrical contact           | Rare-earth magnets |
| ----------------------------------------- | ----------------------- | ---------------------------------- | ------------------ |
| PMSM                                      | Permanent magnets       | None                               | Usually yes        |
| Traditional wound-field synchronous motor | Electromagnetic winding | Brushes/slip rings                 | No                 |
| Induction motor                           | Induced rotor current   | None                               | No                 |
| **Vimag VMSM**                            | Electromagnetic winding | **Inductive rotating transformer** | **No**             |

So Vimag is trying to combine two attractive properties:

**PMSM:** synchronous operation, strong controllable torque and potentially high power density.

**Wound-field motor:** no expensive permanent magnets and complete control over rotor flux.

The rotating transformer removes one of the traditional disadvantages of wound-field machines: **brushes and slip rings**. Vimag's patent explicitly identifies replacement of the brush/slip-ring system as a principal feature. ([Google Patents][1])

There is another important advantage. A permanent magnet is always magnetized. An electromagnet can be weakened when maximum field strength is unnecessary. That gives the controller another degree of freedom:

[
T = f(I_d,I_q,I_f)
]

rather than essentially

[
T = f(I_d,I_q,\psi_{PM})
]

where (\psi_{PM}) is largely fixed by the permanent magnets.

That extra (I_f) control variable could be particularly useful for **field weakening, high-speed efficiency, thermal management, and optimization over an EV drive cycle**.

### What is actually novel

The fundamental physics is **not new**. Wound-field synchronous machines, transformer coupling, brushless excitation and contactless rotor excitation all have substantial prior history.

Vimag's intellectual property is therefore more specific: its particular **rotating-transformer geometry, excitation architecture, rotor rectification, control strategy, sensorless methods and integrated motor/controller implementation**. Its U.S. application US20240178726A1, for example, describes fixed-frequency excitation of the rotating transformer, a three-phase rectifier mounted with the shaft, Pulse Width Modulation (PWM) control, and techniques for estimating/controlling rotor field current from the stationary side. ([Google Patents][1])

So I would characterize the engineering as:

> **A highly integrated, electronically excited synchronous motor with contactless rotor excitation, rather than an entirely new electromagnetic motor principle.**

That distinction gets obscured by the “software-defined magnet” terminology.

### The engineering tradeoff

There is no free replacement for the permanent magnet.

A PMSM gets its rotor magnetic field essentially without continuously supplying electrical excitation power. Vimag must instead pay for:

[
P_\text{field}
==============

P_\text{transformer loss}
+
P_\text{rectifier loss}
+
I_f^2R_\text{rotor}
]

So its difficult engineering problem is making those additional losses and components small enough that eliminating permanent magnets remains worthwhile.

The upside is substantial: no rare-earth magnets, no demagnetization problem, no brushes, variable rotor flux, and potentially lower material cost. The downside is additional copper, rotor electronics, excitation hardware, heat generation and control complexity.

Volektra/Vimag currently reports approximately **95% efficiency for its contactless power-transfer subsystem, 94% drive-cycle efficiency, and 3–7 kW/kg power density**. Those figures are company-reported rather than independent production-vehicle validation, so they should be treated as performance claims at this stage. ([Volektra][2])

**The rotating transformer is the part I find technically most consequential:** it effectively gives the controller an electrically accessible rotor **without physically wiring the stationary world to the rotating world**. Once you have that, the permanent magnet becomes an electronically controllable variable rather than a fixed material property. ([Google Patents][1])

[1]: https://patents.google.com/patent/US20240178726A1/en "US20240178726A1 - Magnet-less and brush-less rotating transformer excited synchronous machine and method of its control - Google Patents"
[2]: https://www.volektra.com/technology.html?utm_source=chatgpt.com "Technology - Volektra"

<b>Q: How is this different than Tesla/Lucid induction motors?</b>

**A:** The fundamental difference is **where the rotor magnetic field comes from and whether it can be controlled independently of torque production**.

| Motor                                  | How rotor field is created                                          | Rotor speed                                     | Independent rotor-field control |
| -------------------------------------- | ------------------------------------------------------------------- | ----------------------------------------------- | ------------------------------- |
| **Induction motor**                    | Main stator field induces current directly into rotor conductors    | Slightly different from synchronous field speed | No                              |
| **Vimag wound-field motor**            | Separate rotating transformer sends power into rotor field windings | Synchronous with stator field                   | **Yes**                         |
| **Permanent-magnet synchronous motor** | Permanent magnets provide rotor flux                                | Synchronous                                     | Mostly fixed                    |

### Tesla induction motor

A conventional Tesla-style induction motor does something particularly elegant:

```text
Stator AC
   ↓
Rotating magnetic field
   ↓
Relative motion versus rotor
   ↓
Current induced in rotor cage
   ↓
Rotor magnetic field
   ↓
Torque
```

There is **no separate rotor power-transfer system at all**.

The stator simultaneously:

1. Produces the rotating magnetic field.
2. Induces current into the rotor.
3. Interacts with that induced rotor field to generate torque.

The rotor must therefore experience some relative motion with respect to the rotating field. This is **slip**:

[
s=\frac{\omega_s-\omega_r}{\omega_s}
]

Without slip, there is no induced rotor electromotive force, essentially no rotor current, and consequently no induction torque.

### Vimag

Vimag separates those functions:

```text
Main inverter
      ↓
Main stator
      ↓
Torque-producing rotating field
      ↕
Electromagnetic rotor

Separate excitation inverter
      ↓
Stationary transformer winding
      ))
   magnetic coupling
      ))
Rotating transformer winding
      ↓
Rectifier
      ↓
Rotor field winding
```

The rotor's magnetic field therefore **does not depend on slip**.

The controller can independently command:

[
I_f
]

for rotor excitation, while separately controlling the stator's:

[
I_d,\ I_q
]

components.

That gives the controller approximately three useful electromagnetic control variables:

[
T=f(I_d,I_q,I_f)
]

An induction motor instead has much tighter coupling between stator excitation, rotor current, slip, rotor flux, and torque.

### Why Vimag would bother

The Vimag architecture gives it something an induction machine fundamentally lacks: **direct control of rotor flux from outside the rotor**.

For example, at one operating point it could command:

[
I_f=100%
]

for maximum field strength.

At high speed it might use:

[
I_f=40%
]

to reduce back electromotive force and losses.

At light cruise:

[
I_f=20%
]

might be sufficient.

The exact optimum values depend on the motor design, but the important point is that **rotor magnetization becomes an independently controllable actuator**.

In an induction motor, reducing rotor flux means manipulating the stator field and slip conditions that create that flux in the first place.

### Efficiency trade

Induction avoids the Vimag transformer and rotor excitation electronics, but its rotor necessarily carries induced current:

[
P_{\text{rotor copper}}\approx I_r^2R_r
]

Those rotor losses are intrinsic to producing induction torque.

Vimag also has rotor copper losses:

[
P_{\text{field}}=I_f^2R_f
]

plus losses in its rotating transformer and rectifier.

The difference is **control**. Vimag can deliberately choose the amount of rotor excitation independently of the torque-producing stator current.

That can potentially improve the efficiency map across a broad range of speed and load, although whether it beats a well-designed induction or permanent-magnet machine overall depends on the implementation.

### One correction concerning Tesla and Lucid

Modern Tesla vehicles are **not uniformly induction-motor vehicles**. For example, Tesla currently specifies the Model 3 and Model Y rear motor as an AC permanent-magnet synchronous motor, while the front motor on All-Wheel Drive (AWD) versions is an AC induction motor. Current Model S documentation specifies permanent-magnet synchronous motors. ([Tesla][1])

Lucid's current production Air and Gravity vehicles are also primarily **permanent-magnet machines**, not induction motors. The 2026 Gravity, for example, uses dual permanent-magnet motors. Lucid does, however, advertise both permanent-magnet and induction variants of its newer Atlas drive-unit family. ([Lucid Motors][2])

So the more revealing comparison is actually:

**Tesla induction:** electromagnetized rotor, but rotor excitation is passively induced.

**Lucid/Tesla PMSM:** rotor field is supplied by permanent magnets and is essentially always present.

**Vimag:** electromagnetized rotor whose field is **actively and independently commanded without electrical contacts**.

That last property is the distinctive feature. In conceptual terms, Vimag is trying to obtain the controllability of a separately excited DC machine while retaining the packaging, speed capability, and brushless construction of a modern synchronous AC traction motor.

Keywords: induction slip, squirrel-cage rotor, wound-field synchronous motor, rotating transformer, contactless excitation, rotor flux control, (I_d/I_q/I_f), field weakening.

[1]: https://www.tesla.com/ownersmanual/modely/en_us/GUID-E414862C-CFA1-4A0B-9548-BE21C32CAA58.html?utm_source=chatgpt.com "Subsystems"
[2]: https://media.lucidmotors.com/attachment/d0010342-f57a-4ded-bab4-17f5e3f28d74?utm_source=chatgpt.com "2026 Lucid Gravity Grand Touring"

<b>Q: So an electric motor torque curve is not truly flat -- this is what Vimag tries to address?</b>

**A:** Correct that an electric motor's torque curve is **not truly flat**, but **Vimag is not primarily trying to make torque flat across the entire speed range**. It is trying to make the rotor magnetic field independently controllable, which can improve the torque-speed envelope and efficiency.

A typical traction motor has roughly two regions:

```text
Torque
  │
  │───────────────┐   Constant-torque region
  │               │
  │               └─────\
  │                       \
  │                        \   Constant-power region
  │                         \
  └──────────────────────────── RPM
                  Base speed
```

### Why torque eventually falls

At low and moderate speed, the inverter can supply enough current to maintain approximately maximum torque:

[
T \approx \text{constant}
]

Therefore mechanical power rises with speed:

[
P=T\omega
]

Eventually the motor reaches its voltage/back-electromotive-force limit. Above this **base speed**, power becomes approximately constant, so:

[
T \approx \frac{P}{\omega}
]

Torque must therefore decline approximately inversely with rotational speed.

For example, a 300 kW motor operating in the constant-power region would theoretically produce:

|      Speed |  Torque |
| ---------: | ------: |
|  5,000 rpm | 573 N·m |
| 10,000 rpm | 286 N·m |
| 20,000 rpm | 143 N·m |

That falling portion is fundamental unless you increase available electrical power or change gearing.

### Where Vimag helps

A permanent-magnet motor has another complication: its rotor magnetic field is essentially fixed.

At high speed, that permanent field produces increasingly large back electromotive force. The inverter consequently has to perform **field weakening**, typically by applying negative direct-axis current. Some stator current is then being used to oppose the permanent magnets rather than produce useful torque.

Vimag can instead tell the rotor itself:

> Produce less magnetic flux.

Its independently controlled rotor current (I_f) lets it reduce excitation directly.

Conceptually:

```text
Low RPM / high torque
Rotor field: strong

Cruise / light load
Rotor field: optimized downward

Very high RPM
Rotor field: deliberately weakened
```

That is potentially cleaner than electrically fighting a permanent magnet with the stator.

### So the distinction is important

Vimag does **not** eliminate:

[
P=T\omega
]

and therefore cannot magically provide maximum torque at arbitrarily high RPM without correspondingly enormous power.

Instead, it potentially makes the transition between operating regions **more controllable and efficient**:

**Conventional PMSM**

fixed rotor flux → field weakening through stator → increasing compromises at high speed

**Vimag**

variable rotor flux → directly weaken rotor excitation → optimize (I_f), (I_d), and (I_q) together

So I would characterize Vimag's opportunity as **broadening and optimizing the usable torque-speed envelope**, rather than producing a genuinely flat torque curve.

There is an interesting implication: with sufficiently fast rotor-field control, Vimag can treat **motor magnetization itself as another continuously variable transmission parameter**. That is arguably more significant than simply flattening the torque curve.

<b>Q: Ahh, for smoother operation and less torque spin?</b>

**A:** Partly, but **not primarily**. If by “torque spin” you mean wheelspin from the abrupt low-speed torque of an EV, Vimag's controllable rotor field could help shape torque delivery, but conventional EVs already control wheel torque extremely quickly through the inverter.

There are three separate effects:

* **Wheelspin:** Mainly controlled by traction control and stator current. Vimag adds rotor-field strength as another control variable, but this is not necessary for excellent traction control.
* **Torque ripple / smoothness:** Vimag could potentially optimize electromagnetic operation, but torque ripple is largely determined by motor geometry, winding design, inverter control, harmonics, and commutation.
* **Torque-versus-speed behavior:** This is where Vimag is more interesting. The rotor magnetic field can be strengthened or weakened as operating conditions change, potentially extending the efficient operating envelope and improving high-speed behavior.

So instead of:

> accelerator request → fixed-strength rotor magnet → adjust only stator current

Vimag can approximately do:

> accelerator request → select rotor magnetization + select stator current → desired torque

That gives the controller **another degree of freedom**, but the strongest case for it is **efficiency and flux management across speed/load**, rather than simply preventing tire spin.

A useful analogy is that a conventional permanent-magnet motor has a fixed-strength magnet with sophisticated electronic throttle control. Vimag effectively adds an electronically adjustable **magnet-strength control** as well.
