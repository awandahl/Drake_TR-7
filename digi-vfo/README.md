

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


