
# 🧠 I²C Master (System Verilog)

This project implements a bit-banged **I²C Master Controller** in **System Verilog**, capable of performing standard **I²C read and write operations** to slave devices using configurable timing parameters. It simulates I²C protocol using software-defined timing on the SDA and SCL lines.

## 📁 File Structure

```
i2c_master_project/
├── master.v        # I²C Master Verilog module
├── master_tb.v     # Testbench for simulation
├── README.md       # Project documentation
```

## 🔧 Module: `master`

### 📌 Description

The `master` module supports:
- START, STOP, ACK/NACK control
- 7-bit addressing
- Read/write operation (R/W bit)
- Open-drain `sda` line
- SCL generation based on system clock

### 🧮 Parameters

| Name            | Description                      | Default Value |
|----------------|----------------------------------|---------------|
| `system_freq`   | System clock frequency (Hz)      | 4_000_000     |
| `i2c_freq`      | Desired I²C bus frequency (Hz)   | 100_000       |

Internal divider:
```verilog
parameter clk_count4 = system_freq / i2c_freq;  // Full I2C period
parameter clk_count1 = clk_count4 / 4;          // 1/4 period (for 4-phase state)
```

---

## 📋 Module Overview

### Module: `master`

This module controls communication over an I²C bus as a master device. It initiates start and stop conditions, sends a 7-bit slave address with a read/write flag, and performs single-byte read or write transactions. It handles all protocol phases under internal state machine control.

---

## ⚙️ Parameters

| Parameter       | Description                      | Default       |
|----------------|----------------------------------|---------------|
| `system_freq`  | Clock frequency of system (Hz)   | `4_000_000`   |
| `i2c_freq`     | Target I²C bit rate (Hz)         | `100_000`     |
| `clk_count4`   | One full I²C clock cycle count   | `system_freq / i2c_freq` |
| `clk_count1`   | One quarter I²C clock cycle      | `clk_count4 / 4`         |

The `clk_count1` is used to divide each I²C bit into four timing phases.

---

## 🔌 I/O Interface

| Signal       | Direction | Width | Description                                     |
|--------------|-----------|--------|-------------------------------------------------|
| `clk`        | input     | 1     | System clock                                    |
| `rst`        | input     | 1     | Active-high synchronous reset                   |
| `newd`       | input     | 1     | Triggers a new I²C transaction                  |
| `addr`       | input     | 7     | 7-bit I²C slave address                         |
| `op`         | input     | 1     | Operation: 0 = Write, 1 = Read                  |
| `din`        | input     | 8     | Data byte to write                              |
| `dout`       | output    | 8     | Data byte read from slave                       |
| `sda`        | inout     | 1     | I²C Serial Data Line                            |
| `scl`        | output    | 1     | I²C Serial Clock Line                           |
| `busy`       | output    | 1     | High when transaction is in progress            |
| `done`       | output    | 1     | Pulses high for one clock when transaction ends |
| `ack_error`  | output    | 1     | High if slave did not acknowledge               |

---

## 🧠 FSM States

The design is based on a `typedef enum logic [3:0]` FSM, transitioning on quarter-bit pulse phases. Key states include:

- `idle`: Wait for new transaction
- `start`: Generate start condition (SDA falling while SCL high)
- `write_addr`: Send 7-bit address + R/W bit
- `ack_1`: Wait for ACK from slave after address
- `write_data`: Send 8-bit data
- `read_data`: Read 8-bit data
- `ack_2`: Wait for ACK after data write
- `master_ack`: Send NACK after read
- `stop`: Generate stop condition

---

## ⏱ Timing Control

An internal counter increments on every system clock edge. It is used to produce a `pulse` signal that transitions through 4 values (0 to 3) corresponding to quarter-bit edges in the I²C protocol.

This pulse timing controls all SCL/SDA changes for protocol correctness.

---

## 🔄 SDA Control Logic

Your implementation uses the following SystemVerilog logic to manage bidirectional SDA:

```systemverilog
assign sda = (sda_en == 1) ? ((sda_t == 0) ? 1'b0 : 1'b1) : 1'bz;


### 🔍 Key Waveform Observations

- START: SDA goes `1 → 0` while SCL is high
- WRITE: Address and data sent MSB-first
- ACK: Slave pulls SDA low after each byte
- STOP: SDA goes `0 → 1` while SCL is high

---

###Behavior:

    If sda_en == 1 and sda_t == 0 → Pulls SDA LOW

    If sda_en == 1 and sda_t == 1 → Drives SDA HIGH

    If sda_en == 0 → Releases SDA to high-impedance (Z)

## 📘 Example Use

Write 0xFF to slave at address 0x3C:
```verilog
addr  = 7'h3C;
op    = 0;
din   = 8'hFF;
newd  = 1'b1;
```

Read from slave:
```verilog
addr  = 7'h3C;
op    = 1;
newd  = 1'b1;
```

## ✅ License

This project is open-source and available under the MIT License.
