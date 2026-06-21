

You can treat the Digi VFO as a drop‑in PTO replacement and then choose, per‑configuration, whether RIT/split comes from the Digi VFO or from the stock TR‑7. The key is to inject the Digi VFO where the PTO used to feed the exciter, add a clean 5 V supply inside the rig, and bring a simple TX/RX sense line into the Digi VFO.[^1][^2][^3][^4]

Below is a practical procedure in four parts:

- Mechanical/electrical PTO replacement
- Powering the Digi VFO (5 V)
- TX/RX sense into Digi VFO
- RIT/split strategy (TR‑7 vs Digi VFO)

I’ll keep it high‑level so you can map to your service manual, but with enough detail to be actionable.

***

## 1. Replace PTO with Digi VFO at the correct node

Goal: remove the analog PTO from the LO chain and inject Digi VFO at exactly the same electrical point, so the rest of the TR‑7 (including its digital display and RIT) doesn’t know or care.

1. **Locate PTO output and VFO input node**
    - From the TR‑7 service manual, identify the PTO output lead/connector feeding the exciter board (often a shielded lead marked “PTO” or similar, going into the synthesizer/mixer section).[^2]
    - On the exciter PCB, mark the node where that lead lands; that becomes your *injection point*.
2. **Electrically remove the PTO**
    - Desolder or unplug the PTO RF output lead at the exciter input node.
    - Optionally remove PTO power so it no longer oscillates; you can leave the module mechanically in place or remove it entirely, as you prefer.[^5][^2]
3. **Inject Digi VFO LO**
    - Take Digi VFO Clk0 output (or the chosen RF output) and run it via a short shielded cable or twisted pair to the former PTO injection node, with the shield/return tied to the same RF ground reference the PTO used.[^3][^1]
    - Keep the cable short and route it similarly to the original PTO lead to minimize RF pickup.
4. **Configure LO frequency plan**
    - Determine the original PTO frequency range from the TR‑7 service manual (roughly a 5 MHz‑range LO feeding the synthesizer; exact numbers you can lift from the VFO section).[^2]
    - Configure Digi VFO so its *output* sweeps that same range with the desired tuning step, and use IF offset/TYPE settings so its *display* shows RF operating frequency while its LO output is the PTO‑equivalent frequency.[^6][^7][^1]
    - Calibrate Digi VFO reference so the TR‑7 digital display shows correct on‑air frequency (calibrate via WWV or a modern rig as frequency standard).[^7][^8][^9]

At this point, the TR‑7 behaves like stock but with a synthesized PTO.

***

## 2. Providing 5 V for Digi VFO inside the TR‑7

Digi VFO tolerates up to about 12 V at its V+ pin but QRP Labs strongly recommend not feeding it raw 13.8 V: the onboard regulator will run too hot. Use a local 5 V regulator instead.[^10][^1][^6]

1. **Choose a source rail**
    - The TR‑7 uses internal regulated rails (e.g. +10 V, +13.8 V, +5 V on some logic boards); you can tap either +10 V or +13.8 V at a convenient, always‑on point (e.g. the same rail that fed the PTO).[^11][^12][^2]
    - For RF cleanliness, pick a node on a regulated rail rather than the raw unregulated input, and add local decoupling.
2. **Add a small 5 V regulator for Digi VFO**
    - Mount a 78L05/7805 (or a small linear module) near the Digi VFO board.[^1][^10]
    - Input: from +10 V or +13.8 V rail (fused if you want belt‑and‑braces).
    - Output: 5 V to Digi VFO V+, with 0.1 µF and 10 µF capacitors near the regulator and near the Digi VFO board.[^13][^10][^1]
    - Ground: tie to the same RF ground the PTO injection node uses, and keep ground leads short.
3. **Current and heat considerations**
    - Digi VFO current is modest, so a 78L05 in TO‑92 or small SMD package is usually enough; if you use a 7805, give it a small tab or chassis contact if fed from 13.8 V.[^10][^1]
    - Avoid switching regulators near the VFO injection point unless you are very careful about filtering; a linear regulator keeps it simple and RF‑quiet.[^10]

***

## 3. TX/RX sense into Digi VFO

To allow Digi VFO to do split (different LO on TX vs RX) and its own RIT, it needs to know when the TR‑7 is transmitting. Digi VFO has a /TX input which responds to a low signal indicating TX mode.[^14][^4][^15]

1. **Understand Digi VFO TX input**
    - According to QRP Labs, Digi VFO has a /TX pin (active low); pulling /TX low tells Digi VFO you’re in TX, and it can then alter frequency (e.g. apply TX offset, use VFO B for split).[^4]
    - In some Si5351 VFO projects, a similar pin is documented: “Pin 5 is the RX/TX pin. Put this pin LOW for RX, open or high for TX,” or vice versa; check the exact logic in the current Digi VFO manual, but the pattern is the same: a simple binary sense.[^16][^14]
2. **Find a TX indication in the TR‑7**

You want a logic or DC line that is reliably:
    - One level (e.g. high) in RX
    - The other level (e.g. low or 0 V) in TX

Candidate signals (from TR‑7 service docs and similar mods):
    - The line that keys the PA/driver (the PTT control line from the control board).
    - A TX LED drive or control line, if it’s driven from a logic signal.[^17][^2]
    - If needed, you can derive this from a relay coil line that is only energised during TX (with appropriate isolation).

You’ll need to eyeball the control board schematic and pick the cleanest candidate; other TR‑7 digital VFO and DAFC‑style add‑ons tap into the existing RIT/PTO and control wiring without major surgery.[^18][^11][^17]
3. **Translate TR‑7 TX signal to Digi VFO /TX**

Once you have identified a TX line:
    - If the TR‑7 line swings between 0 V and +12 V or +13.8 V, use a simple interface:
        - A small NPN transistor (or MOSFET) as an open‑collector stage, with base resistor and possibly a clamp diode, so when TX is asserted the transistor pulls /TX low, and when RX it is open.[^19][^20]
        - Alternatively, a resistor divider and a transistor/optocoupler if you want galvanic isolation.
    - Tie Digi VFO /TX to this transistor’s collector, with a pull‑up inside the Digi VFO (or an external pull‑up to its 5 V) according to the QRP Labs manual.[^4][^1]

Example: suppose you pick a +12 V “TX enable” line that goes high in TX:
    - Feed that through a 47–100 kΩ resistor into the base of a small NPN, emitter to Digi VFO GND, collector to /TX.
    - Use Digi VFO’s internal pull‑up so /TX floats high in RX, and is pulled low when the TR‑7 asserts TX (driving the transistor).[^19][^4]

You now have a clean logic indication into Digi VFO without loading the TR‑7 control circuitry.

***

## 4. Choosing between Digi VFO RIT/split and TR‑7 RIT

With the above wiring in place, you can decide day‑to‑day whether you want:

- Stock TR‑7 RIT/XIT to do small offsets, or
- Digi VFO to do dual VFO, split, and RIT.

You don’t need any extra switching hardware; it’s mostly “which knobs do you touch and how you set the offsets.”

### Option A: “TR‑7‑centric” RIT, Digi VFO for PTO

- Leave the TR‑7 RIT/XIT circuitry fully connected.
- Center the Digi VFO’s own RIT offset at zero and either disable Digi RIT in firmware or just don’t use it.
- Use the TR‑7 RIT knob as you always have; the Digi VFO just provides the LO at whatever base frequency you’re tuned to.[^18][^17]

Split operation in this mode:

- You can still use Digi VFO’s dual VFO/split, driven by the /TX sense line:
    - VFO A = RX frequency
    - VFO B = TX frequency
    - Split on: Digi VFO outputs B on TX, A on RX.[^15][^21][^22]
- In practice, you’d keep TR‑7 RIT at zero most of the time and only use Digi VFO’s split/A/B when needed.


### Option B: “Digi VFO‑centric” RIT/split, TR‑7 RIT mostly idle

- Leave TR‑7 RIT controls at their electrical center; treat them as “unused” in normal operation.
- Enable Digi VFO’s RIT and split features; use its encoder and buttons (or your resurrected TR‑7 knob/encoder) to set RIT, VFO A/B, and split.[^21][^23][^15][^4]
- Because Digi VFO sees /TX and (optionally) /CW, it can:
    - Apply a receive‑only offset (RIT) by shifting LO only in RX.
    - Apply CW offset according to mode.
    - Use VFO A/B for split TX/RX frequencies.

The TR‑7 simply follows; its front‑panel RIT and XIT become “mostly decorative,” but you haven’t physically removed them, so you could always restore them by re‑wiring later.

### Hybrid: keep both available

Because the injection point is the PTO node and you haven’t removed RIT wiring, you can mix:

- Small tweaks with TR‑7 RIT in some operating situations (e.g. fine zero‑beat), and
- Larger jumps/split via Digi VFO dual VFO in contest/DX use.[^24][^25]

Practically you’ll probably settle into one pattern, but the hardware doesn’t force you.

***

If you want, I can sketch a specific example based on a particular TR‑7 schematic revision (e.g. “tap +10 V at test point X for the 5 V regulator; take TX sense from control board node Y; inject Digi VFO at exciter point Z”) once you confirm which TR‑7 version you have and whether you already have the DAFC or other mods installed.
<span style="display:none">[^26][^27][^28][^29][^30]</span>

<div align="center">⁂</div>

[^1]: https://qrp-labs.com/digivfo.html

[^2]: https://pub.deadnet.se/Manuals/ciasteczkowypotwor/%23ham_radio/Drake/Drake_TR-7_serv.pdf

[^3]: https://www.wb4hfn.com/DRAKE/DrakeArticles/DSS-VFO/DDS-VFO.htm

[^4]: https://qrp-labs.com/images/digivfo/DigiVFO_1_00.pdf

[^5]: https://www.wb4hfn.com/DRAKE/DrakeArticles/TR7-Digital_VFO_Upgrade/TR7_Digital_VFO.htm

[^6]: http://shop.qrp-labs.com/digivfo

[^7]: https://miscdotgeek.com/qrp-labs-vfo-review/

[^8]: https://www.hb9ryz.ch/downloads/tr7_manual.pdf

[^9]: https://www.youtube.com/watch?v=mprb1GQ4p-o

[^10]: https://blog.templaro.com/vfo-in-a-box/

[^11]: https://shop.elcon.ch/media/docs/drake-tr-7-ii-installation-guide_v1.0a_en.pdf

[^12]: https://groups.io/g/DRAKE-RADIO/topic/tr_7a_power_supply_board/69903048

[^13]: https://www.qrp-labs.com/images/vfo/assembly_vfo.pdf

[^14]: https://gist.github.com/la3pna/7f3ef8c9c75e035fce02

[^15]: https://qrp-labs.com/digivfo

[^16]: https://www.scribd.com/document/748231631/Si5351a-VFO-Manual-1

[^17]: https://dk4sx.darc.de/tr7neu.htm

[^18]: http://www.dl7maj.de/DAFC-deLuxe_en.pdf

[^19]: http://www.qrp.gr/supervfo/index.htm

[^20]: https://www.embeddedrelated.com/thread/7792/microcontroller-with-5v-supply-to-control-3-3v-power

[^21]: https://manuals.plus/m/4667d2bde8203ce2231f216d6194c6a79c4c6e36323f0db9a599c0be76011fa5

[^22]: https://itshamradio.com/sbitx-guide-book/split-operation/

[^23]: https://id.scribd.com/document/54383893/dds-2v0

[^24]: https://groups.io/g/DRAKE-RADIO/topic/tr7_digital_vfo/58986411

[^25]: https://www.ure.es/foros/tecnico/nuevo-vfo-nvr7-con-dds-para-el-drake-tr-7/

[^26]: http://www.dl7maj.de/Dig-VFO R7.pdf

[^27]: https://gmcotton.com/Ham_Radio/MISC%20Manuals/Drake/Drake%20R-V7%20Remote%20VFO_%20Instruction%20Manual.pdf

[^28]: https://www.scribd.com/document/502780731/dds-x-v1-6

[^29]: https://www.youtube.com/watch?v=865w7aN_Wcw

[^30]: https://www.scribd.com/document/672536103/DDSVFO2-manual-V1




Here’s a concrete, TR‑7–specific scheme for installing the Digi VFO in place of the PTO, with suggested points for LO injection, 10→5 V power, and TX sense. This assumes a “plain” TR‑7 (no DAFC/EL‑34) and the standard parent/power board and exciter layout.[^1][^2][^3]

I’ll structure it as:

- Injection at the PTO node (point Z)
- 10→5 V supply pick‑off (point X)
- TX sense pick‑off (point Y)
- How to choose between TR‑7 RIT and Digi VFO RIT/split in practice

All references are to the TR‑7 service manual and circuit diagram sets.[^2][^4][^1]

***

## 1. LO injection: replacing the PTO (point Z)

### 1.1 Identify the PTO output and exciter input

From the service manual:

- The TR‑7 PTO is a 5.05–5.55 MHz permeability‑tuned oscillator that “provides a 5.05–5.55 MHz injection signal to the synthesizer.”[^1]
- On the chassis/parent board diagram, the PTO output is brought to the parent board at the posts labeled “PTO” and from there to the exciter (synthesizer) board via a short coax or shielded lead.[^4][^1]

On your rig:

1. Remove the top cover and locate the PTO can and the coax/lead coming from it.
2. Follow that lead to the parent board; you should find a small post or connector labeled “PTO” (as in the Elcon DAFC guide, which explicitly says, “The PTO connection is via a small caliber 50 Ohm coaxial cable RG‑316 from the posts with the PTO sign of the motherboard to the RF in of the EL‑34”).[^5]
3. That PTO post is your **point Z**: the LO injection node.

### 1.2 Disconnect original PTO and attach Digi VFO

- Desolder or unplug the PTO coax from the PTO post on the parent board. Leave the board pad/post itself intact.
- Optionally, unplug or cut PTO DC power if you want it fully dead; otherwise at least detune or disable it so it doesn’t oscillate.[^1]
- Run a short piece of shielded cable or twisted pair from Digi VFO Clk0 output to the PTO post:
    - Center conductor (or one conductor of the twisted pair) to PTO post (point Z).
    - Shield/return to a nearby parent‑board ground pad or the same ground lug the PTO coax shield used.[^5][^1]

Digi VFO now directly drives the exciter at the same node the PTO used, with the expected ~5 MHz range.[^6][^1]

***

## 2. 5 V supply for Digi VFO: tap +10 V at point X

The TR‑7’s power board generates +10 V and +5 V rails; the +10 V rail is widely distributed and is exactly what the DAFC/PT0 stabilizer mods tap for their own logic.[^3][^5]

### 2.1 Identify the +10 V rail

From “Inside the TR‑7 – The Power Supply Board”:

- The power board takes the main 13.8 V and produces +10 V, +5 V, +24 V, and –5 V. The +10 V and +5 V are internally regulated on that board.[^3]
- The +10 V and +5 V rails leave the power board via its card‑edge connector and are distributed on the parent board; in some repair notes, +10 V and +5 V test points are mentioned on the parent board itself.[^7][^8]

On your rig:

1. Locate the power supply board (rear side, vertical board).
2. With the service manual’s “Power Supply Board” schematic, identify the +10 V output pin on its card‑edge connector.[^4][^3]
3. Either:
    - Tap +10 V directly on the power board at a convenient pad, or
    - Tap it on the parent board at a clearly labeled +10 V feed or test point (as done in the Elcon DAFC installation, which takes +10 V “from the respective motherboard PCB assembly of the TR‑7”).[^5]

This tapped point is your **point X**.

### 2.2 Add a local 5 V regulator

Near the Digi VFO board:

1. Mount a small linear regulator (78L05 or 7805) on a bit of perfboard or a tiny PCB.
2. Wiring:
    - Input: from point X (+10 V) via a short wire, optionally through a small fuse (e.g. 100 mA) or series resistor if you want protection.
    - Output: to Digi VFO V+, ~5.0 V.[^9][^10][^6]
    - Ground: to the same parent‑board ground area you used for the PTO shield / Digi VFO RF ground.[^3][^5]
3. Decoupling:
    - 0.1 µF and 10 µF caps at regulator input and output, close to the regulator.
    - 0.1 µF at Digi VFO V+ pins as per QRP Labs guidance.[^10][^6]

The +10→5 V conversion keeps Digi VFO cool and avoids loading any existing +5 V logic rails in the TR‑7.[^11][^6][^3]

***

## 3. TX sense: pick a keying line at point Y

You want a signal that cleanly indicates TX vs RX and is easy to interface.

The TR‑7 uses a “switching board” for T/R, and the circuit diagrams show a keying line and “FIXED Tx SW” control points.[^12][^2]

### 3.1 Find a suitable TX/RX indicator

From the TR‑7 circuit diagram PDF:

- The “Switching Board Schematic” includes the PTT keying path and fixed TX switch; the “KEY” line and “FIXED Tx SW” notation appear in the TR‑7 chassis \& parent board schematic.[^2][^12]
- Other notes (G4ALG mods etc.) reference a key line where applying +12 V keys the rig and where a series RC was added across the key jack to shape the waveform.[^12]

A practical approach:

1. Identify the **key line** that runs from the key jack / PTT logic to the driver/PA bias circuits on the “Switching Board Schematic.”[^2][^12]
2. Choose a point where the voltage clearly goes high (≈+12 V or +10 V) in TX and low (0 V) in RX, and where loading it with a high‑impedance detector won’t disturb operation.
3. Mark that node as **point Y**.

If you prefer an LED‑driven line (e.g. TX indicator LED drive from the control board) that toggles between 0 and some DC level, you can equally use that as point Y.

### 3.2 Build a simple interface to Digi VFO /TX

Assuming point Y is a +10–12 V line that goes high on TX:

- Digi VFO /TX expects a low (0 V) to indicate TX; it has (or you can add) a pull‑up to its internal 5 V.[^13][^14][^6]

Use a small NPN transistor as an open‑collector inverter:

1. Connect a 47–100 kΩ resistor from point Y to the base of a small NPN (e.g. 2N3904).
2. Emitter to Digi VFO ground.
3. Collector to Digi VFO /TX pin.
4. Ensure Digi VFO /TX has a pull‑up to 5 V (either internal as documented, or add ~10 kΩ to 5 V at the Digi VFO end).[^14][^15][^13]

Operation:

- RX: point Y ≈ 0 V → transistor off → /TX pulled high → Digi VFO “RX mode.”
- TX: point Y ≈ +10–12 V → base drive → transistor on → /TX pulled to 0 V → Digi VFO “TX mode.”[^13][^14]

This loads the TR‑7 key line with only microamps (47–100 kΩ), so it’s invisible to the rig, and gives Digi VFO a clean logic indication.

If point Y is low on TX and high on RX, you can either:

- Swap the logic in the Digi VFO config (if supported), or
- Use the transistor the other way round (different pull‑up/logic arrangement), but the principle—isolated open‑collector interface—remains the same.[^15][^16]

***

## 4. Switching between Digi VFO RIT/split and TR‑7 RIT in practice

With:

- LO injected at PTO post (Z),
- Digi VFO powered from +10→5 V (X),
- /TX sense via point Y,

you can choose at the operating level which RIT/split “layer” you use.

### 4.1 “TR‑7 RIT primary, Digi VFO split optional”

- Leave TR‑7 RIT and XIT circuitry untouched.
- Keep TR‑7 RIT centered most of the time.
- Use Digi VFO for:
    - Normal tuning (PTO replacement).
    - Dual VFO A/B and split (thanks to /TX sense), for bigger TX/RX offsets.[^17][^18][^19][^20]

In this mode:

- Your main mental model stays “TR‑7 is the radio, Digi VFO is just a smarter PTO with split.”
- When you press TR‑7 RIT and move its knob, the offset is applied inside the TR‑7 LO chain, regardless of whether Digi VFO is doing split; you can still use it for tiny corrections if you want.[^21][^22]


### 4.2 “Digi VFO RIT/split primary, TR‑7 RIT idle”

- Set TR‑7 RIT/XIT controls to electrical zero and leave them there.
- Use Digi VFO’s own RIT, split, VFO A/B, and CW offset logic; /TX and /CW inputs let Digi VFO know when to apply TX offset vs RX offset.[^18][^20][^17][^13]
- TR‑7 sees only whatever base LO Digi VFO chooses; its RIT is effectively unused, but still present and reversible.


### 4.3 Hybrid “best of both”

Because you didn’t cut any TR‑7 RIT wiring and you injected only at the PTO node:

- You can mix small RIT tweaks via TR‑7 front‑panel RIT with big split offsets via Digi VFO dual VFOs.
- You can also decide later to route the TR‑7’s RIT control voltage into Digi VFO (/RIT input or a spare analog input) and do the offset in Digi VFO firmware if you want a “DAFC‑like” integrated solution (similar in concept to what the DAFC module does by tapping the existing RIT circuit).[^22][^23][^5]

***

Given your level of comfort, the next thing I’d suggest is:

- On your actual TR‑7, locate and label:
    - PTO post on parent board (Z),
    - A convenient +10 V pad or test point (X),
    - A keying or TX‑LED line you like for point Y.
- Then we can sanity‑check those specific node IDs against the circuit diagram PDFs (e.g. parent board node numbers, switching board connector pins), and tweak the suggestions if you’d prefer, say, to use the existing +5 V rail rather than adding a new regulator, or to derive TX sense from a different board.
<span style="display:none">[^24][^25][^26][^27][^28][^29][^30]</span>

<div align="center">⁂</div>

[^1]: http://bee.mif.pg.gda.pl/ciasteczkowypotwor/%23ham_radio/Drake/Drake_TR-7_serv.pdf

[^2]: https://www.wb4hfn.com/DRAKE/DrakeManuals/PDFDOCS/TR7-Circuit-Diagrams.pdf

[^3]: https://www.wb4hfn.com/DRAKE/DrakeArticles/InsideTheTR7/Inside_The_TR7-07.htm

[^4]: https://www.wb4hfn.com/DRAKE/DrakeManuals/PDFDOCS/TR-7_Service_manual.pdf

[^5]: https://shop.elcon.ch/media/docs/drake-tr-7-ii-installation-guide_v1.0a_en.pdf

[^6]: https://qrp-labs.com/digivfo.html

[^7]: https://groups.io/g/DRAKE-RADIO/topic/tr7_with_low_power_on_15_and/102702614

[^8]: https://waves-vagues.dfo-mpo.gc.ca/Library/49392.pdf

[^9]: http://shop.qrp-labs.com/digivfo

[^10]: https://www.qrp-labs.com/images/vfo/assembly_vfo.pdf

[^11]: https://groups.io/g/DRAKE-RADIO/topic/tr_7a_power_supply_board/69903048

[^12]: http://www.zerobeat.net/drakelist/tr7moremods.html

[^13]: https://qrp-labs.com/images/digivfo/DigiVFO_1_00.pdf

[^14]: https://gist.github.com/la3pna/7f3ef8c9c75e035fce02

[^15]: http://www.qrp.gr/supervfo/index.htm

[^16]: https://www.embeddedrelated.com/thread/7792/microcontroller-with-5v-supply-to-control-3-3v-power

[^17]: https://qrp-labs.com/digivfo

[^18]: https://manuals.plus/m/4667d2bde8203ce2231f216d6194c6a79c4c6e36323f0db9a599c0be76011fa5

[^19]: https://itshamradio.com/sbitx-guide-book/split-operation/

[^20]: https://id.scribd.com/document/54383893/dds-2v0

[^21]: https://dk4sx.darc.de/tr7neu.htm

[^22]: http://www.dl7maj.de/DAFC-deLuxe_en.pdf

[^23]: https://s873ba685ce580070.jimcontent.com/download/version/1310210400/module/4320762251/name/dds-x_v1_6.pdf

[^24]: http://www.bigskyspaces.com/w7gj/station.htm

[^25]: https://es.scribd.com/document/586307158/DRAKE-TR-7-Service-Manual

[^26]: https://lse.se/hubfs/3179-81-F-Jotron-TR7750.pdf?hsLang=en

[^27]: https://www.florian-anwander.de/yahoo/tr-707/1297.html

[^28]: https://www.youtube.com/watch?v=WQtd6DDfOJA

[^29]: https://groups.io/g/DRAKE-RADIO/topic/drake_tr7_pa_problem/101304081

[^30]: https://www.youtube.com/watch?v=765OPf8xiZ4


