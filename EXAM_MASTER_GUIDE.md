# CMOS VLSI Lab (ECEL23606) — Exam Master Guide

> A single study sheet for the SEE practical. It tells you (1) how to write up any
> answer, (2) how all 19 questions collapse into a small set of reusable skills, and
> (3) the exact step-by-step procedure + parameters + code for each skill.
>
> Built from your GAT lab manual and the official question bank.

---

## 0. How the exam is scored (so you know where marks come from)

Semester End Examination (SEE) is out of 100, then scaled to 50:

| Activity   | Marks |
|------------|-------|
| Write-up   | 15    |
| Conduction | 70    |
| Viva Voce  | 15    |
| **Total**  | **100** |

What this means for you:
- **Every question has two parts: (a) Analog/Layout and (b) Digital/Verilog.** You must do both.
- The **write-up (15)** is free marks if you memorise the format below.
- **Conduction (70)** is the actual tool work — this is what the "skills" sections train.
- You may change the experiment **only once**, and doing so zeroes 20% of the conduction marks. So aim to be comfortable with *all* skills rather than gambling on one.

---

## 1. The universal write-up format (use this for EVERY question)

Write parts (a) and (b) as two separate experiments using the **same skeleton**:

1. **Aim / Objective** — one or two lines, copied almost verbatim from the question.
2. **Tools / Software Required**
   - Analog (a): *Cadence Virtuoso* (Schematic + Layout XL), *ADE L* with *Spectre* simulator, *Assura* (DRC/LVS), *Quantus QRC* (extraction); technology = **gpdk180 (180 nm)**; OS = Linux.
   - Digital (b): *Cadence Incisive / NCLaunch* (ncvlog, ncelab, ncsim) for simulation; *Genus* for synthesis; HDL = **Verilog**.
3. **Theory** — 3–5 lines on what the circuit does (use the snippets in this guide).
4. **Circuit Diagram / Truth Table / Block Diagram** — draw it.
5. **Design Specifications** — the W/L table or the truth table (given in each skill below).
6. **Procedure** — the numbered steps (from the skill sections).
7. **Verilog Code + Test bench** — for part (b) only.
8. **Observations / Expected Waveforms** — describe inputs vs outputs; sketch the waveform.
9. **Calculations** — delay formulas / gain / bandwidth where asked.
10. **Result / Conclusion** — "The X was designed, simulated and verified; DRC/LVS clean; delay/gain = ...".

> Tip: For analog questions the *Procedure* is almost identical every time — only the
> circuit and parameters change. For digital questions the *Procedure* is **identical
> every time** — only the Verilog code changes.

---

## 2. The Master List — everything reduces to these skills

### The big picture

```
ANY EXAM QUESTION
        |
   +----+--------------------------+
   |                               |
 PART (a) ANALOG                 PART (b) DIGITAL
   |                               |
 pick ONE of 3 circuits         pick ONE of ~9 Verilog modules
   - Inverter                     and run the SAME flow every time
   - 2-input NAND
   - Common Source Amplifier
   |
 do ONE (or both) of 2 flows
   - Schematic flow (capture + simulate + measure)
   - Layout flow (layout + DRC + LVS + QRC + back-annotate)
```

### Part A skills (analog)

| Skill | What you master | Used for |
|-------|-----------------|----------|
| **A1** | Inverter: schematic, symbol, testbench, DC + transient, `tpHL/tpLH/td` | Inverter questions |
| **A2** | 2-input NAND: schematic, symbol, testbench, functionality + delay table | NAND questions |
| **A3** | Common Source Amplifier (PMOS current-mirror load): schematic, symbol, TB, transient/DC/AC, **gain & bandwidth** | CSA questions |
| **A4** | **Layout + DRC + LVS + QRC** — *identical procedure for all 3 circuits* | Any "draw layout / verify DRC, LVS, extract parasitics" |
| **A5** | **Post-layout simulation / back-annotation** + compare with pre-layout | Any "post-layout / back annotation / compare results" |

### Part B skills (digital)

| Skill | What you master | Used for |
|-------|-----------------|----------|
| **D1** | The **Verilog → NCLaunch → Genus** flow — *identical for every module* | Every part (b) |
| **D2** | The actual **Verilog codes**: up/down counter, full adder, D/SR/JK/T flip-flops, D/SR/JK latches | Every part (b) |

### Bottom line

If you can confidently do **A1, A2, A3, A4, A5** and **D1 + the 9 codes in D2**, you can
answer **all 19 questions**. That's three analog circuits, two analog flows, one digital
flow, and a code sheet. See Section 6 for the exact question-to-skill map.

---

## 3. One-time environment setup (do this before anything)

Open a terminal and start the Cadence environment:

```bash
csh                 # switch to C-shell
source cshrc        # load tool paths + licenses
mkdir 1ga23ec167    # your working folder (use your USN); skip if it already exists
cd 1ga23ec167
```

- For **analog** work, launch the GUI: `virtuoso &`
- For **digital** work, you stay in the terminal and use `gedit`, `nclaunch`, `genus`.

> The `cshrc` file is what attaches the **gpdk180** technology library and the digital
> standard-cell libraries. If a tool says "library not found", you forgot `source cshrc`.

---

# PART A — ANALOG SKILLS

Every analog circuit follows this generic full-custom flow (memorise the order):

```
Create Library (attach gpdk180)
  -> Schematic (Create > Instance, Wire, Pin ; Check & Save)
  -> Symbol (Create > Cellview > From Cellview)
  -> Test bench (instantiate symbol + sources)
  -> Simulate in ADE L (Spectre)  ... measure delay / gain / BW
  -> Layout XL (Generate All From Source, place, route)   [Skill A4]
  -> DRC -> LVS -> QRC extraction                          [Skill A4]
  -> Back-annotate av_extracted -> re-simulate (post-layout) [Skill A5]
```

### Common keyboard shortcuts (Virtuoso)
| Key | Action | Key | Action |
|-----|--------|-----|--------|
| `i` | Create Instance | `w` | Create Wire |
| `p` | Create Pin | `r` | Create Rectangle / path (layout) |
| `o` | Create Via/Contact (layout) | `q` | Edit Properties of selected object |
| `Shift+X` | Check & Save (schematic) | `f` | Fit view |
| `M` | Place marker (waveform) | `Esc` | Cancel current command |

### Create the library (once)
1. In the CIW: `File > New > Library`.
2. Name it (e.g. your USN). Choose **"Attach to an existing technology library"** and select **gpdk180**.

---

## Skill A1 — CMOS Inverter (schematic, symbol, testbench, delay)

**Aim:** Capture the inverter schematic with a 0.1 pF load, simulate DC + transient, and compute `tpHL`, `tpLH`, `td`.

**Theory (write-up):** A CMOS inverter is a PMOS (pull-up) and NMOS (pull-down) in series between VDD and GND with gates tied to the input. When Vin is low the PMOS conducts and Vout = VDD; when Vin is high the NMOS conducts and Vout = 0. It has near-zero static power (current only flows while switching) and high noise margins.

### Device parameters (from manual)
| Library | Cell | W (Total = Finger width) | L | Bodytie |
|---------|------|--------------------------|---|---------|
| gpdk180 | pmos | **40 µm** | 180 n | Integrated |
| gpdk180 | nmos | **20 µm** | 180 n | Integrated |

> The PMOS is ~2× the NMOS width to equalise rise/fall (PMOS mobility is lower).

### Schematic steps
1. `File > New > Cellview` → Library = yours, Cell = `inverter`, View = `schematic`.
2. Press `i`, browse **gpdk180 → pmos**, set Total width = `40u`, L = `180n`, place it. (Type `2u` style values; the tool appends the unit automatically.)
3. Press `i`, browse **gpdk180 → nmos**, set width = `20u`, L = `180n`.
4. Add power nets: press `i`, from **analogLib** place `vdd` and `gnd`.
5. Press `w` and wire: both gates → `Vin`; both drains → `Vout`; PMOS source → vdd; NMOS source → gnd.
6. Press `p` to add pins: `Vin` (direction **input**), `Vout` (direction **output**).
7. `Shift+X` (Check & Save). Fix any floating-wire warnings.

### Symbol
- `Create > Cellview > From Cellview` → confirm Library/Cell → OK. A triangle inverter symbol is generated.

### Test bench
1. `File > New > Cellview`, Cell = `inverter_tb`, View = schematic.
2. Press `i`, place your **inverter symbol**.
3. From **analogLib** add the sources:

| Source | Settings | Connect to |
|--------|----------|------------|
| `vpulse` | V1 = **0 V**, V2 = **1.8 V**, Period = **20 ns**, Rise = **1 ns**, Fall = **1 ns**, Pulse width = **10 ns** | Vin |
| `vdc` | DC voltage = **1.8 V** | vdd |
| `gnd` | 0 | gnd / reference |
| `cap` | **0.1 pF (100 fF)** | Vout → gnd (load) |

4. `Shift+X` to save.

### Simulate (ADE L)
1. From the testbench: `Launch > ADE L`.
2. `Setup > Model Libraries` → ensure `gpdk180.scs`, corner **NN (typical)** is loaded.
3. `Analyses > Choose`:
   - **tran** — Stop time = **200 ns**, accuracy = Moderate.
   - **dc** — sweep the `vpulse`/input component, Start = **0 V**, Stop = **1.8 V**.
4. `Outputs > To be plotted > Select on schematic` → click the `Vin` and `Vout` wires.
5. Click **Netlist & Run** (green play). Expected: Vout is the inverse of Vin (square wave flipped), with rounded edges from the 0.1 pF load.

### Delay calculation (this is the marks-earning bit)
Propagation delay:

```
td = (tpHL + tpLH) / 2
```
- `tpHL` = delay from input rising to output falling (high→low at output)
- `tpLH` = delay from input falling to output rising (low→high at output)
- Both measured at the **50% switching point = 0.9 V** (half of 1.8 V VDD)

**Tool steps:** `Tools > Calculator` → enable **Wave**, disable **Clip** → pick function **delay** → Signal 1 = `Vin`, Signal 2 = `Vout` → set Threshold 1 and Threshold 2 = **0.9** → choose Edge types (rising/falling) for `tpHL`, opposite for `tpLH` → **Evaluate** (the table icon). Average the two for `td`.

> The switching threshold can also be found exactly with the calculator's **intersect**
> function on the DC curve (where Vout = Vin). For 1.8 V supply it's ~0.9 V.

### Delay table to fill (Skill A1)
| Quantity | Value |
|----------|-------|
| tpHL | ___ ps |
| tpLH | ___ ps |
| td = (tpHL+tpLH)/2 | ___ ps |

---

## Skill A2 — 2-input CMOS NAND gate (schematic, symbol, TB, delay table)

**Aim:** Capture the 2-input NAND schematic, verify functionality, and find delay `td` for the input combinations. Tabulate results.

**Theory:** NAND = AND followed by NOT. Output is **0 only when both inputs are 1**, else 1. CMOS NAND uses **two PMOS in parallel** (pull-up) and **two NMOS in series** (pull-down). Series NMOS is why each NMOS is sized like the inverter's NMOS so the gate matches inverter delay.

### Device parameters (from manual, Wp/Wn = 40/20)
| Library | Cell | W | L |
|---------|------|---|---|
| gpdk180 | pmos | **40 µm** (×2, in parallel) | 180 n |
| gpdk180 | nmos | **20 µm** (×2, in series) | 180 n |

### Schematic
1. New cellview `nand2`, schematic.
2. Place 2× PMOS (sources → vdd, drains tied together → `Y`/Vout).
3. Place 2× NMOS **in series**: top NMOS drain → Vout, the two NMOS share a middle node, bottom NMOS source → gnd.
4. Gate of PMOS1 + NMOS1 → input **A**; gate of PMOS2 + NMOS2 → input **B**.
5. Add pins A, B (input), Y (output); add vdd/gnd. `Shift+X`.

### Symbol
- `Create > Cellview > From Cellview` → generate a NAND symbol.

### Test bench + functional simulation
1. New cellview `nand2_tb`, place the symbol, add `vdc` = 1.8 V for vdd and `gnd`.
2. Launch **ADE L** → `Setup > Stimuli`.
3. `Stimulus Type > Inputs` → for each of A and B: **Enabled**, **Function = bit**, One value = **1.8**, Zero value = **0**, Rise/Fall/Period as in A1, **Source type = bit**, give a **Pattern** (manual uses `11001001`) so all input combinations occur, Trigger = Internal → Apply → OK.
4. Choose **tran** analysis, plot A, B, Y. Run.

### Truth table & delay table (fill these)
| A | B | Y (= NAND) |
|---|---|------------|
| 0 | 0 | 1 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

| Input transition | tpHL | tpLH | td |
|------------------|------|------|----|
| Combination 1 | | | |
| Combination 2 | | | |
| ... | | | |

Measure delay with the **Calculator > delay** function exactly as in Skill A1 (threshold 0.9 V).

---

## Skill A3 — Common Source Amplifier with PMOS Current-Mirror Load

**Aim:** Capture the CS amplifier schematic, find Transient / DC / AC response, and calculate **Gain** and **Bandwidth (UGB)**.

**Theory:** A common-source amplifier uses an NMOS as the gain (driver) transistor. Its load is an **active load** built from a PMOS current mirror, which gives a large small-signal output resistance and therefore high voltage gain. Gain ≈ gm(NMOS) × (ro_n ∥ ro_p). The dominant pole (set by the output node and load capacitance) fixes the bandwidth.

### Device parameters (from manual)
| Library | Cell | W (Total = Finger) | L | Bodytie |
|---------|------|--------------------|---|---------|
| gpdk180 | nmos | **6 µm** | 180 n | Integrated |
| gpdk180 | pmos | **8.85 µm** | 180 n | Integrated |

### Test-bench source parameters (from manual)
| Library | Cell | Settings |
|---------|------|----------|
| analogLib | `vdc` | DC voltage = **3.3 V** (supply) |
| analogLib | `isin` | DC current = **60 µA** (bias) |
| analogLib | `vsin` | DC voltage = **592 mV**, AC magnitude = **1 V**, Amplitude = **10 µV**, Frequency = **10 kHz** |
| analogLib | `cap` | **100 fF (0.1 pF)** load |
| analogLib | `gnd` | 0 |

### Steps
1. Build the schematic: NMOS driver, PMOS current-mirror pair as load, bias current source, input at NMOS gate, output at the drain node. Add pins, vdd/gnd. `Shift+X`.
2. Create symbol (`From Cellview`).
3. Build testbench: instantiate the symbol + the sources from the table above. `Shift+X`.
4. **ADE L** → `Analyses > Choose`:
   - **tran** (transient response)
   - **dc** (operating point / DC sweep)
   - **ac** — sweep frequency over a wide range (e.g. 1 Hz to 1 GHz). AC magnitude on `vsin` = 1 V.
5. Plot input and output. **Netlist & Run**.

### Gain & Bandwidth (the marks bit)
1. Back in ADE L: `Results > Direct Plot > AC Magnitude & Phase`.
2. Click the **output net** on the schematic → press `Esc`.
3. On the Bode plot:
   - **DC gain** = flat low-frequency magnitude (place a marker with bind key `M`). Read in dB (or V/V).
   - **Unity Gain Bandwidth (UGB)** = frequency where the magnitude curve crosses **0 dB**. Place a horizontal cursor at 0 dB; the crossing frequency is the UGB.
   - **3-dB Bandwidth** = frequency where gain drops 3 dB below its DC value.

### Result table (fill)
| Quantity | Value |
|----------|-------|
| DC Gain (dB) | |
| DC Gain (V/V) | |
| 3-dB Bandwidth | |
| Unity Gain Bandwidth (UGB) | |

---

## Skill A4 — Layout + DRC + LVS + Parasitic Extraction (SAME for all 3 circuits)

> This is the single most reused analog skill. The procedure below is identical whether
> the layout is for the Inverter, NAND, or CS amplifier — only the device count changes.

### Step 1 — Create the layout
1. From the circuit's **schematic** window: `Launch > Layout XL > Create New`. In the New File window, View = `layout`, Type = `layout` → OK.
2. `Connectivity > Generate > All From Source` — this auto-imports the layout P-cells for every NMOS/PMOS/pin from the schematic.
3. When the **Assura Technology Lib Select** appears, Browse to `assura_tech.lib` at
   `/home/install/FOUNDRY/analog/180nm/` → OK.

### Step 2 — Placement & routing
- Place **PMOS on top**, **NMOS on bottom**, inside the PR boundary.
- Press `r` to draw routing paths:
  - **Metal 1 (M1)** — connect drains/sources, and the VDD (top) / VSS-GND (bottom) rails.
  - **Poly** — connect the transistor gates.
- Press `o` to drop **vias/contacts** (Poly↔M1, M1↔diffusion, M1↔pins).
- **Latch-up prevention:** place an **n-tap (tied to VDD)** near the PMOS and a **p-tap (tied to GND)** near the NMOS.
- Save the layout.

### Step 3 — DRC (Design Rule Check)
- `Assura > Run DRC` → select the **gpdk180** rules / `assura_tech.lib`, give a Run Name (no spaces) → OK.
- When done, click **Yes** to view results. A clean run reports **0 violations**. If there are errors, the Error Layer Window points to spacing/width issues — fix, save, re-run until clean.

### Step 4 — LVS (Layout vs Schematic)
- `Assura > Run LVS` → verify the **Schematic Design Source** and **Layout Design Source**, give a Run Name, Technology = **gpdk180** → OK.
- The LVS Debug window should say **"Schematic and Layout Match"** (0 violations). This proves your metal/poly wiring matches the schematic netlist.

### Step 5 — QRC / Parasitic Extraction (Quantus)
- `Assura > Run Quantus`.
- **Setup tab:** Technology = **gpdk180**, Output = **Extracted View**.
- **Extraction tab:** Extraction Type = **RC** (or R-only / C-only), Ref Node = **VSS** → OK.
- This produces the **`av_extracted`** view (open it from the Library Manager). It contains the real parasitic R and C of your physical layout.

> Quick definitions for viva: **DRC** = checks geometry against foundry rules (width,
> spacing, enclosure). **LVS** = checks that layout connectivity == schematic. **QRC/RCX**
> = extracts parasitic R/C. **ERC** = electrical rule check (e.g. floating gates).

---

## Skill A5 — Post-layout simulation (back-annotation) + comparison

> Used whenever a question says "perform post-layout simulation / back annotation /
> compare with pre-layout". It reuses the `av_extracted` view from Skill A4.

1. `File > New > Cellview` → Cell = `<circuit>_tb`, **Type = config** → "Open with → Hierarchy Editor" → OK.
2. In the New Configuration window: **Use Template → Spectre**; set **Top Cell View → schematic** → OK.
3. In the **Hierarchy Editor (Tree View)**, right-click the instance `I0` → **Set Instance View → av_extracted**. Save.
4. Open the testbench, descend into the symbol and confirm `View = av_extracted`.
5. `Launch > ADE L` → `Session > Load State` (reuse your pre-layout setup) → **Re-run**.
6. Recompute delay (or gain/BW) the same way.

### Expected observation (always state this)
> The **post-layout delay is slightly larger** than the pre-layout delay because the
> extracted interconnect parasitic resistance and capacitance add extra RC loading that
> was not present in the ideal schematic. (For the amplifier, the extra capacitance
> slightly **reduces the bandwidth**.)

### Comparison table (fill)
| Parameter | Pre-layout | Post-layout | Change |
|-----------|-----------|-------------|--------|
| tpHL / Gain | | | |
| tpLH / BW | | | |
| td | | | |

---

# PART B — DIGITAL SKILLS

## Skill D1 — The Verilog → Simulation → Synthesis flow (IDENTICAL every time)

> This is the single procedure for **every** part (b). Only the `.v` code changes.
> Learn this once and you can do all 19 digital halves.

### Phase 1 — Write the code
```bash
csh
source cshrc
mkdir 1ga23ec167          # your USN (skip if exists)
cd 1ga23ec167
gedit dff.v &             # design file (use the correct module name)
gedit dff_tb.v &          # testbench file
```
Rules: file name must end in `.v`; the testbench has **no ports** and instantiates the design.

### Phase 2 — Functional simulation in NCLaunch (GUI multi-step)
```bash
nclaunch -new
```
1. Click **Multiple Step**.
2. Click **Create cds.lib File** → choose **"Don't include any libraries (Verilog design)"** → OK → Save. (`cds.lib` maps the logical library `worklib` to a directory.)
3. The NCLaunch window appears: HDL files on the left; `worklib` and `snapshots` on the right.
4. **Compile** (`ncvlog`): select `dff.v`, click the **Verilog Compiler** icon. Then select `dff_tb.v` and compile it too. Compiled modules appear under **worklib**.
5. **Elaborate** (`ncelab`): expand `worklib`, select the **testbench** module (e.g. `dff_tb`), click the **Elaborator** icon. The elaborated design appears under **snapshots**. (Elaboration builds the hierarchy, binds instances, checks connectivity. Use `-access +rwc` so waveforms are writable.)
6. **Simulate** (`ncsim`): expand `snapshots`, select the testbench snapshot, click the **Simulator** icon. The SimVision window opens.
7. In SimVision: right-click the testbench signals → **Send to Waveform Window** → click **Run** (play). Verify the waveform matches the truth table.

**Equivalent CLI (faster):**
```bash
ncvlog dff.v dff_tb.v          # compile
ncelab -access +rwc dff_tb     # elaborate (rwc enables waveform read/write)
ncsim -gui dff_tb              # simulate with GUI
```

### Phase 3 — Logic Synthesis in Genus
1. Write a Tcl script:
```bash
gedit run.tcl &
```
A representative `run.tcl` (the structure your lab uses — `read_hdl` then map and report):
```tcl
# --- libraries (paths come from your lab's cshrc / foundry) ---
set_db init_lib_search_path  /home/install/FOUNDRY/digital/90nm/dig/lib
set_db init_hdl_search_path  ./
read_libs  slow.lib                 ;# standard-cell timing library

# --- read + elaborate the RTL ---
read_hdl   { dff.v }                ;# <-- put YOUR design file(s) here
elaborate

# --- constraints (area + timing) ---
create_clock -name clk -period 10 [get_ports clk]   ;# 100 MHz example
set_max_area 0

# --- synthesize: generic -> map to gates -> optimize ---
syn_generic
syn_map
syn_opt

# --- reports ---
report_area
report_power
report_timing

# --- write the gate-level netlist ---
write_hdl > dff_netlist.v
```
2. Run it:
```bash
genus -f run.tcl
```
3. Inside Genus you can also type the reports directly:
```tcl
report area
report power
report timing
```

### What to record from the synthesis report (this is the deliverable)
| From report | What to note |
|-------------|--------------|
| `report area` | Total **number of cells/gates** and total **area** |
| `report power` | Total **power** (leakage + dynamic) |
| `report timing` | **Critical path** and **maximum frequency** (1 / critical-path delay) |

Also note the **RTL/gate-level schematic** Genus produces, and (if asked to compare latch vs flip-flop) tabulate area/power/timing of each.

---

## Skill D2 — The Verilog code sheet

> All codes below are cleaned/corrected versions of the manual's listings (OCR fixed,
> smart-quotes removed, port widths corrected) so they compile. Encoding for SR/JK uses
> a 2-bit bus `sr`/`jk` where bit[1] = S/J and bit[0] = R/K.

### D2.1 — 4-bit Up/Down Counter (asynchronous reset)
```verilog
module counter(clk, rst, m, count);
    input  clk, rst, m;            // m = 1 -> count up, m = 0 -> count down
    output reg [3:0] count;
    always @(posedge clk or negedge rst)
    begin
        if(!rst)                   // active-low async reset
            count = 4'b0000;
        else if(m)
            count = count + 1;     // up
        else
            count = count - 1;     // down
    end
endmodule
```
```verilog
`timescale 1ns/1ps
module counter_test;
    reg  clk, rst, m;
    wire [3:0] count;
    counter counter1(clk, rst, m, count);
    initial begin
        clk = 0; rst = 0; #25;
        rst = 1;                   // release reset, counting starts
    end
    initial begin
        m = 1;                     // up first
        #600 m = 0;                // then down
    end
    always #5 clk = ~clk;          // 100 MHz clock
    initial #1400 $finish;
endmodule
```

### D2.2 — 4-bit Full Adder (structural)
```verilog
module full_adder(A, B, CIN, S, COUT);
    input  A, B, CIN;
    output S, COUT;
    assign S    = A ^ B ^ CIN;
    assign COUT = (A & B) | (B & CIN) | (CIN & A);
endmodule

module adder(A, B, C0, S, C4);
    input  [3:0] A, B;
    input  C0;
    output [3:0] S;
    output C4;
    wire   C1, C2, C3;
    full_adder fa0(A[0], B[0], C0, S[0], C1);
    full_adder fa1(A[1], B[1], C1, S[1], C2);
    full_adder fa2(A[2], B[2], C2, S[2], C3);
    full_adder fa3(A[3], B[3], C3, S[3], C4);
endmodule
```
```verilog
`timescale 1ns/1ps
module adder_tb;
    reg  [3:0] A, B;
    reg  C0;
    wire [3:0] S;
    wire C4;
    adder dut(A, B, C0, S, C4);
    initial begin
        A = 4'b0011; B = 4'b0011; C0 = 1'b0; #10;
        A = 4'b1011; B = 4'b0111; C0 = 1'b1; #10;
        A = 4'b1111; B = 4'b1111; C0 = 1'b1; #10;
    end
    initial #50 $finish;
endmodule
```

### D2.3 — D Flip-Flop (edge-triggered)
```verilog
module dff(d, clk, rst, q, qb);
    input  d, clk, rst;
    output reg q;
    output qb;
    assign qb = ~q;
    always @(posedge clk)
    begin
        if(rst == 1'b1)
            q <= 1'b0;
        else
            q <= d;
    end
endmodule
```
```verilog
`timescale 1ns/1ps
module dff_tb;
    reg  clk, d, rst;
    wire q, qb;
    dff U1(d, clk, rst, q, qb);
    initial begin clk = 1'b0; forever #10 clk = ~clk; end
    initial begin
        #20 d = 1'b1; rst = 1'b1;
        #20 d = 1'b1; rst = 1'b0;
        #20 d = 1'b0; rst = 1'b0;
        #20 d = 1'b1; rst = 1'b1;
        #300 $finish;
    end
endmodule
```

### D2.4 — D Latch (level-sensitive, enable `en`)
```verilog
module dlatch(d, en, rst, q, qb);
    input  d, en, rst;
    output reg q;
    output qb;
    assign qb = ~q;
    always @(en or d or rst)
    begin
        if(rst == 1'b1)
            q = 1'b0;
        else if(en)        // transparent when enabled, holds when en = 0
            q = d;
    end
endmodule
```
*(Testbench: same as the D-FF but toggle `en` instead of a clock — drive `en`, `d`, `rst` with `#20` steps.)*

### D2.5 — SR Flip-Flop (sr[1]=S, sr[0]=R)
```verilog
module srff(sr, rst, clk, q, qb);
    input  [1:0] sr;
    input  clk, rst;
    output reg q, qb;
    reg    ff;
    always @(posedge clk)
    begin
        if(rst == 1'b1)
            ff = 1'b0;
        else
            case(sr)
                2'b01: ff = 1'b0;   // S=0,R=1 -> reset
                2'b10: ff = 1'b1;   // S=1,R=0 -> set
                2'b11: ff = 1'bz;   // forbidden
                default: ff = ff;   // 2'b00 -> hold
            endcase
        q  = ff;
        qb = ~ff;
    end
endmodule
```
```verilog
`timescale 1ns/1ps
module srff_tb;
    reg  [1:0] sr;
    reg  clk, rst;
    wire q, qb;
    srff U1(sr, rst, clk, q, qb);
    initial begin clk = 1'b0; forever #10 clk = ~clk; end
    initial begin
        sr = 2'b01; rst = 1'b1;
        #20 sr = 2'b10; rst = 1'b0;
        #20 sr = 2'b11;
        #20 sr = 2'b00;
        #20 sr = 2'b01;
        #300 $finish;
    end
endmodule
```

### D2.6 — SR Latch (enable `en`)
```verilog
module srlatch(sr, rst, en, q, qb);
    input  [1:0] sr;
    input  en, rst;
    output reg q, qb;
    reg    ff;
    always @(en or sr or rst)
    begin
        if(rst == 1'b1)
            ff = 1'b0;
        else if(en)
            case(sr)
                2'b01: ff = 1'b0;
                2'b10: ff = 1'b1;
                2'b11: ff = 1'bz;
                default: ff = ff;
            endcase
        q  = ff;
        qb = ~ff;
    end
endmodule
```

### D2.7 — JK Flip-Flop (jk[1]=J, jk[0]=K)
```verilog
module jkff(jk, clk, rst, q, qb);
    input  [1:0] jk;
    input  clk, rst;
    output reg q, qb;
    reg    ff;
    always @(posedge clk)
    begin
        if(rst == 1'b1)
            ff = 1'b0;
        else
            case(jk)
                2'b01: ff = 1'b0;   // reset
                2'b10: ff = 1'b1;   // set
                2'b11: ff = ~ff;    // toggle
                default: ff = ff;   // hold
            endcase
        q  = ff;
        qb = ~ff;
    end
endmodule
```
```verilog
`timescale 1ns/1ps
module jkff_tb;
    reg  [1:0] jk;
    reg  clk, rst;
    wire q, qb;
    jkff U1(jk, clk, rst, q, qb);
    initial begin clk = 1'b0; forever #2 clk = ~clk; end
    initial begin
        jk = 2'b01; rst = 1'b1;
        #20 jk = 2'b10; rst = 1'b0;
        #20 jk = 2'b11;
        #20 jk = 2'b00;
        #20 jk = 2'b01;
        #300 $finish;
    end
endmodule
```

### D2.8 — JK Latch (enable `en`)
```verilog
module jklatch(jk, en, rst, q, qb);
    input  [1:0] jk;
    input  en, rst;
    output reg q, qb;
    reg    ff;
    always @(en or rst or jk)
    begin
        if(rst == 1'b1)
            ff = 1'b0;
        else if(en)
            case(jk)
                2'b01: ff = 1'b0;
                2'b10: ff = 1'b1;
                2'b11: ff = ~ff;
                default: ff = ff;
            endcase
        q  = ff;
        qb = ~ff;
    end
endmodule
```

### D2.9 — T Flip-Flop (single bit)
```verilog
module tff(t, clk, rst, q, qb);
    input  t, clk, rst;
    output reg q;
    output qb;
    assign qb = ~q;
    always @(posedge clk)
    begin
        if(rst)
            q <= 1'b0;
        else if(t)
            q <= ~q;        // toggle
        // else hold
    end
endmodule
```

### D2.10 — 4-bit T Flip-Flop / T-FF register
> "4-bit T flip-flop" is usually a 4-bit register of T flip-flops (each bit toggles when
> its T = 1). If your examiner means a T-FF based 4-bit counter, tie all `t` bits to 1.
```verilog
module tff_4bit(t, clk, rst, q);
    input  [3:0] t;
    input  clk, rst;
    output reg [3:0] q;
    always @(posedge clk)
    begin
        if(rst)
            q <= 4'b0000;
        else
            q <= q ^ t;     // toggle each bit whose t = 1
    end
endmodule
```

### Truth-table cheat sheet (for write-ups & viva)
| Type | Inputs | Behaviour |
|------|--------|-----------|
| **D** | D | Q follows D (FF: on clock edge; Latch: while EN=1) |
| **SR** | S R | 00 hold, 01 reset(Q=0), 10 set(Q=1), 11 forbidden |
| **JK** | J K | 00 hold, 01 reset, 10 set, 11 **toggle** |
| **T** | T | T=0 hold, T=1 **toggle** |

**Latch vs Flip-flop:** a **latch** is level-sensitive (transparent while enable is high);
a **flip-flop** is edge-triggered (changes only at a clock edge). That single difference
is why each latch module uses `always @(en or ...)` while each flip-flop uses
`always @(posedge clk)`.

---

# 6. Per-question map — which skills each of the 19 questions needs

> Read this as: "For Q*n*, do these skill sections." Nothing here is new — it's just
> the same A1–A5 / D1+D2 building blocks recombined.

| Q | Part (a) — Analog | Skills (a) | Part (b) — Digital | Skills (b) |
|---|-------------------|-----------|--------------------|-----------|
| 1  | CS Amplifier: schematic **+ layout**, DRC/LVS/extract | A3 (schematic) + **A4** | SR **Latch** | D1 + D2.6 |
| 2  | 2-i NAND: schematic **+ layout**, DRC/LVS/extract | A2 + **A4** | 4-bit up/down **Counter** | D1 + D2.1 |
| 3  | 2-i NAND: schematic+symbol+TB, functionality + delay td (table) | **A2** | **D flip-flop** | D1 + D2.3 |
| 4  | CMOS **Inverter**: schematic+symbol+TB, compute tpHL/tpLH/td | **A1** | **SR flip-flop** | D1 + D2.5 |
| 5  | CS Amplifier: schematic+symbol+TB, transient + AC, gain & BW | **A3** | **JK Latch** | D1 + D2.8 |
| 6  | **Inverter**: schematic **+ layout**, DRC/LVS/extract | A1 + **A4** | **D Latch** | D1 + D2.4 |
| 7  | 2-i NAND: schematic+symbol+TB, functionality + delay td | **A2** | **D flip-flop** | D1 + D2.3 |
| 8  | **Inverter**: schematic+symbol+TB **+ layout**, transient + DC, delay | A1 + **A4** | **Inverter layout**: DRC/LVS/extract + **post-layout vs pre-layout** | **A4 + A5** |
| 9  | CS Amplifier: schematic+symbol+TB, transient + AC, gain & BW | **A3** | **JK flip-flop** | D1 + D2.7 |
| 10 | **Inverter**: schematic+symbol+TB, tabulate delay | **A1** | **D flip-flop** | D1 + D2.3 |
| 11 | 2-i NAND: schematic+symbol+TB, delay | **A2** | **NAND layout**: DRC/LVS/extract + **post-layout vs pre-layout** | **A4 + A5** |
| 12 | CS Amplifier: **layout**, DRC/LVS/extract | A3 (as source) + **A4** | SR **Latch** | D1 + D2.6 |
| 13 | 2-i NAND: schematic+symbol+TB, functionality + delay td | **A2** | **D flip-flop** | D1 + D2.3 |
| 14 | CS Amplifier: schematic+symbol+TB, transient + AC, gain & BW | **A3** | CSA **layout** + DRC/LVS/extract + **back-annotation** | **A4 + A5** |
| 15 | **Inverter**: schematic **+ layout**, DRC/LVS/extract | A1 + **A4** | **4-bit T flip-flop** | D1 + D2.10 |
| 16 | CS Amplifier: **layout**, DRC/LVS/extract | A3 (as source) + **A4** | **JK Latch** | D1 + D2.8 |
| 17 | **Inverter**: schematic **+ layout**, DRC/LVS/extract | A1 + **A4** | **4-bit full adder** | D1 + D2.2 |
| 18 | 2-i NAND: schematic+symbol+TB, compute delay | **A2** | **D Latch** | D1 + D2.4 |
| 19 | **Inverter**: schematic+symbol+TB, calculate delay | **A1** | **D flip-flop** | D1 + D2.3 |

### Frequency count (study priority)
- **Part (a):** NAND appears ~6×, Inverter ~7×, CS Amplifier ~6×, Layout/DRC/LVS (A4) ~9×, Post-layout (A5) ~3×.
- **Part (b):** **D flip-flop is the most common (≈6 questions)** → learn D2.3 cold first. Then SR/JK latch, then the rest.

**Smartest study order:** D-FF (D2.3) → the analog **A4 layout/DRC/LVS** flow → Inverter (A1) → NAND (A2) → CS Amplifier (A3) → remaining codes → A5. That sequence covers the highest-frequency items first.

---

# 7. Viva quick-reference (the 15 marks people forget to study)

**Technology / tools**
- *gpdk180* = Generic Process Design Kit at **180 nm**; supply used = **1.8 V** (3.3 V for the CS amplifier). `analogLib` = ideal sources/passives; `gpdk180` = real transistors.
- *Virtuoso* = schematic + layout. *ADE L* = Analog Design Environment (runs the **Spectre** simulator). *Assura* = DRC/LVS. *Quantus* = parasitic (RC) extraction. *Incisive/NCLaunch* = Verilog simulation. *Genus* = synthesis. *Innovus* = place & route (physical design).

**Physical verification**
- **DRC** — does the layout obey foundry geometry rules (min width, spacing, enclosure)?
- **LVS** — does the layout's connectivity match the schematic netlist?
- **RCX / QRC** — extracts parasitic R and C → `av_extracted` view.
- **ERC** — electrical rule check (floating gates, shorts).
- **Latch-up** — parasitic SCR between PMOS/NMOS; prevented with **substrate/well taps** (n-tap to VDD, p-tap to GND) and guard rings.

**Inverter / delay**
- `tpHL`, `tpLH` measured at the **50% point (0.9 V for a 1.8 V supply)**; `td = (tpHL + tpLH)/2`.
- PMOS is wider than NMOS (40/20) because hole mobility < electron mobility; this equalises rise/fall times.
- **Why post-layout delay is higher:** extracted interconnect parasitic R and C add loading not present in the ideal schematic.

**NAND**
- Pull-down = **2 NMOS in series**; pull-up = **2 PMOS in parallel**. Output = 0 only when A = B = 1.

**CS amplifier**
- NMOS = driver/gain device; PMOS current mirror = **active load** giving high output resistance and high gain. Gain ≈ gm·(ro_n ∥ ro_p). **UGB** = frequency where |gain| = 0 dB; **3-dB BW** = where gain falls 3 dB from DC.

**Digital flow**
- **Compile (ncvlog)** → syntax check; **Elaborate (ncelab)** → build hierarchy, bind instances, check connectivity; **Simulate (ncsim)** → apply test vectors, view waveforms. `-access +rwc` lets you probe/dump waveforms.
- **Synthesis (Genus)** = RTL → gate-level netlist mapped to a standard-cell library, optimised for **area, power, timing**. `cds.lib` maps the logical library `worklib` to a physical directory.
- **Latch vs flip-flop:** latch is level-sensitive (transparent on enable); flip-flop is edge-triggered (changes only at the clock edge).
- **Testbench** has **no ports**; it generates stimulus (`initial`/`always`), instantiates the DUT, and ends with `$finish`.

**Synthesis report — what to quote**
- `report area` → number of cells + total area; `report power` → dynamic + leakage power; `report timing` → critical path delay → **fmax = 1 / critical-path delay**.

---

# 8. 60-second pre-flight checklist (in the exam)

1. `csh` → `source cshrc` → `cd` into your USN folder. (Nothing works without `source cshrc`.)
2. **Analog:** library attached to **gpdk180**? Right W/L from the table? Pins named and not floating (`Shift+X`)? Model library `gpdk180.scs` loaded in ADE L?
3. **Delay/gain:** thresholds at **0.9 V**; for the amplifier use `Results > Direct Plot > AC Magnitude & Phase`.
4. **Layout:** `Generate > All From Source` first; route M1 + Poly; drop taps; then **DRC → LVS → QRC** in that order.
5. **Digital:** file ends in `.v`; testbench module name matches what you elaborate; instantiate the DUT correctly; `$finish` present.
6. **Synthesis:** `read_hdl` points to YOUR file in `run.tcl`; run `genus -f run.tcl`; capture `report area/power/timing`.
7. Write the **Result** line with actual numbers (delay/gain/BW or area/power/fmax) — examiners look for it.

> Report drawings (schematic / stick diagram / layout) for the analog experiments are
> kept separately as ready-to-compile LaTeX in the **`latex/`** folder — see
> `latex/README.md`.
