

<!-- Start of picture text -->
Core sanityTE Corex  1A SoG corewithout Full ieeeSoC with<br><!-- End of picture text -->



<!-- Start of picture text -->
Unit Testbench<br>scoreboard<br>(Veritog/S¥})<br>Driver _ Responder<br>(Verilog/SVj) (Verilog/SV}<br><!-- End of picture text -->

Most ISA bugs originate here: wrong immediate, wrong opcode mapping, wrong funct decode leads to silent failures, which can lead to wild goose chase later. 

Potential Functional Targets: 

- All RV32I + M decode coverage 

- Immediate correctness: I/S/B/U/J 

- Control outputs: 

`o` “writeback_select” correctness (ALU vs LOAD vs PC+4) 

- reg write enable correctness 

- load/store size and sign 

- branch compare op mapping 

- M-extension detection via funct7=0000001 

Signoff 

- Directed tests for each opcode/funct combination 

- Coverage: 

`o` Cross coverage: opcode x funct3 x funct7 

`o` For loads/stores: size x sign x address_offset 

## 2.3Regfile 

Why needed 

Regfile read/write hazards and x0 behaviour must be verified. 

Potential Functional Targets: 

- x0 hardwired to 0 even if written 

- Same-cycle write/read behavior (define it: write-first or read-first) 

### Signoff 

- All directed tests passing as per verification/testplan. 

## 2.4ALU 

Why needed 

ALU ops are core ISA compliance. 

Target 

- ADD/SUB overflow wrap behavior 

- Shifts: logical vs arithmetic, shift amount masking 

- SLT/SLTU sign correctness 

### Signoff 

- Comparison vs a reference model in unit TB with all directed tests passing ● Coverage: 

`o` Signed corner values: 0x8000_0000, 0x7FFF_FFFF, 0, -1 

- Shift amounts: 0, 1, 31, >31 masked 

## 2.5Branch Unit 

Why needed 

Branch compare errors cause control flow failures and random-looking crashes. 

Target 

- BEQ/BNE 

- BLT/BGE signed 

- BLTU/BGEU unsigned 

- Compare corner cases around sign-bit boundaries 

Signoff 

- Directed compare matrix 

- Coverage: 

`o` rs1/rs2 equal, rs1<rs2, rs1>rs2 for both signed/unsigned 

### 2.6 MDU (M-extension) 

Why needed 

Multi-cycle units are where handshake + latency bugs hide. 

Target 

- MUL/DIV/REM semantics, including divide-by-zero 

- req/gnt/rvalid sequences under backpressure 

- Correct stalling behavior of Execute stage 

- Deterministic latency (if fixed) or bounded latency (if iterative) 

Signoff 

- All directed tests passing. 

- Coverage: 

   - All funct3 M ops 

   - Edge values and divide-by-zero 

   - `o` 

## 2.7Memory Interface Unit (MIU) 

Why needed 

Load/store alignment, byte enables, sign extension, and ordering which are classic silicon escape bugs. 

Potential Functional Targets: 

- Byte/half/word stores: 

   - BE correctness for address [1:0] 

   - write data alignment/shift correctness 

- Loads: 

`o` read data extraction and sign/zero extension 

- Outstanding transaction semantics: one at a time 

- Store completion definition and load completion 

### Signoff 

- All directed tests passing. 

- Coverage: 

   - all size x offset combos 

   - signed/unsigned load coverage 

## 2.8Hazard / Control (Hazard policy modes) 

Why needed 

Hazard handling can get complex and let’s you decide the different kinds of hazards your design 



<!-- Start of picture text -->
Core Sanity TB<br>IF<br>stimulus<br>Tests I-MEM Core DUT D-MEM<br>} writeback & mem<br>Checker /<br><!-- End of picture text -->

### Why needed 

A core-level TB catches integration issues: staging, writeback timing, flush interaction. A quick Testbench which acts as a sanity before getting into the “software” world. 

### Potential Functional Targets: 

- Basic program execution with memory models for I-Memory and D-Memory. 

- Reset sequence: fetch from RESET_PC (Have PC parameterized) 

- Simple instruction sequences and self-checking tests: 

   - arithmetic, branches, jumps, loads/stores, M-extension ops 

   - At least one instruction of each kind. 

- Interrupt entry & handling flow: 

   - irq asserted mid-stream 

   - trap takes at a clean boundary 

   - mtvec jump, mepc saved (based on spec decision) 

- Hazard tests: 

   - Data Hazards: RAW, RAR, WAW, WAR 

   - Control Hazards 

   - Structural Hazards 

### Signoff (core sanity) 

- All directed tests pass 

- Minimum coverage achieved: 

   - 100% opcode coverage for RV32I 

   - M ops each hit at least once 

`o` all load/store sizes hit 

### Infrastructure Needed: 

|Verification Components|Requirements|
|---|---|
|Testbench|Verilog or SV Testbench|
|Stimulus|Tasks/Function calls in TB|
|Checkers|Checking via tasks/function calls in TB|
|Tests|Tasks/Function based Testcases in TB|



# **4. CORE LEVEL VERIFICATION (SW PROGRAMS + RANDOM INSTRS)** 



<!-- Start of picture text -->
Core SW TB<br>I-MEM<br>model<br>Spike/Whisper<br>Ore<br>IRQ C Re D-MEM (ISS -Ref. Model)<br>Stimulus DUT model<br>Core Logs 55<br>Execution Lag Comparator Execution Log<br>| Test Pass/Fail. |<br><!-- End of picture text -->

### 4.4 Standard Benchmarks 

- Industry standard benchmarks like Dhrystone , CoreMark etc. (one will be identified) ● Compliance tests 

### Signoff (core-level) 

- Pass all official ISA tests for supported ISA (eg. RV32IM) 

- riscv-dv regressions: 

   - N seeds with stable pass rate (define N based on project scale; typical 1k–10k) 

- Functional Coverage closure: 

   - instruction coverage (opcode/funct cross) 

   - hazard coverage (dependency patterns) 

   - protocol coverage (gnt/rvalid delay buckets) 

- Code Coverage – 95% with waivers. 

- Clean regression of identified benchmark and compliance tests from RISC-V Standard. 

### Infrastructure Needed: 

|Verification Components|Requirements|
|---|---|
|Testbench|Verilog or SV Testbench|
|ISS Model|Spike ISS*|
|Manual Tests|C & Assembly – RISC-V GNU Toolchain**|
|Random Test<br>Generator<br>Compliance/Benchmar<br>ks|Chip Alliance RISC-V DV***<br>TBD|



*SPIKE ISS: https://github.com/riscv-software-src/riscv-isa-sim 

- This needs to be compiled as per Core’s architecture 

**RISC-V GNU ToolChain: https://github.com/riscv-collab/riscv-gnu-toolchain/releases 

- Pick precompiled toolchain to match the linux version. (-elf version is good for bare-metal) - E.g: riscv32-elf-ubuntu-20.04-nightly-2023.05.14-nightly.tar.gz 

***RISC-V Instruction Generator: https://github.com/chipsalliance/riscv-dv 

# **5. SOC VERIFICATION – WITHOUT CORE** 



<!-- Start of picture text -->
SoC UVM TB<br>wxt-Lite |<br>APS |<br>SPI Boot<br>Master Rak<br>SPI Flash<br>Model<br><!-- End of picture text -->



<!-- Start of picture text -->
SoC TB<br>SoC<br>Perera re ere ee eres Lore<br>AX |<br>AMI to AXI-Lite |<br>axi-ite ||<br>APB |<br>fa) (BR) Comm) (re<br>w= [oe Co) (mm) (ate)SPI Boot<br>GPOVIF UARTVIP | IncVIP | SPIModel Flash<br><!-- End of picture text -->

- Randomized bus arbitration + peripheral activity 

- Power-on reset sequences 

- Backpressure and wait states on interconnect 

- Software-driven random tests (riscv-dv in “bare-metal SoC mode” if feasible) 

### Signoff (Full SoC) 

- Boot success across all supported boot modes (or minimum required modes) 

- Full regression clean: 

   - directed + random + software test suites 

- Coverage closure: 

   - code coverage: statement/branch/toggle for RTL (target e.g., >95% where 

      - meaningful with waivers) 

   - functional coverage: address map + interrupt sources + boot modes 

- Bug closure: 

   - all P0/P1 closed 

`o` no known data corruption issues 

- Performance sanity: 

   - no deadlocks in long runs 

   - no protocol stalls beyond bounded expectations 

### Infrastructure Needed: 

|Verification Components|Requirements|
|---|---|
|Testbench|Verilog or SV Testbench|
|VIPs|https://mbits-mirafra.github.io/projects/|
|Tests|C & Assembly – RISC-V GNU Toolchain|



### RISC-V GNU ToolChain: https://github.com/riscv-collab/riscv-gnu-toolchain/releases 

- Pick precompiled toolchain to match the linux version. (-elf version is good for bare-metal) - Eg: riscv32-elf-ubuntu-20.04-nightly-2023.05.14-nightly.tar.gz 

# **7. DV MATERIALS FOR REFERENCE** 

SystemVerilog Basics: 

- <u>https://github.com/mbits mirafra/SystemVerilogCourse/wiki https://verificationacademy.com/forums/c/systemverilog/7</u> 

UVM Basics <u>https://verificationacademy.com/forums/c/uvm/5</u> 

VIPs: - <u>https://mbits mirafra.github.io/projects/</u> 

