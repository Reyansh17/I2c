
# 🧠 I²C Master (Verilog HDL)

This project implements a bit-banged **I²C Master Controller** in **Verilog**, capable of performing standard **I²C read and write operations** to slave devices using configurable timing parameters. It simulates I²C protocol using software-defined timing on the SDA and SCL lines.

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

### 🔌 Port Description

| Port        | Dir | Width | Description                                  |
|-------------|-----|-------|----------------------------------------------|
| `clk`       | In  | 1     | System clock                                 |
| `rst`       | In  | 1     | Synchronous reset                            |
| `newd`      | In  | 1     | Start a new transaction                      |
| `addr`      | In  | 7     | 7-bit I²C slave address                      |
| `op`        | In  | 1     | Operation: `0` = Write, `1` = Read           |
| `din`       | In  | 8     | Data to write to slave                       |
| `dout`      | Out | 8     | Data read from slave                         |
| `sda`       | IO  | 1     | Serial Data Line (open-drain)                |
| `scl`       | Out | 1     | Serial Clock Line                            |
| `busy`      | Out | 1     | Transaction in progress                      |
| `done`      | Out | 1     | Transaction completed                        |
| `ack_error` | Out | 1     | ACK not received from slave                  |

## 🧪 Simulation

### 🔧 Testbench

Testbench (`master_tb.v`) provides stimulus:
- Resets the system
- Sends I²C write and read commands
- Observes `sda`, `scl`, `dout`, and status signals

### 🔍 Key Waveform Observations

- START: SDA goes `1 → 0` while SCL is high
- WRITE: Address and data sent MSB-first
- ACK: Slave pulls SDA low after each byte
- STOP: SDA goes `0 → 1` while SCL is high

## ⚠️ Important Notes

- SDA is **open-drain**, so output logic must be:
  ```verilog
  assign sda = (sda_en && ~sda_t) ? 1'b0 : 1'bz;
  ```
- ACK detection requires **sampling** `sda` after each byte.
- Timing is software-controlled — not using hardware I²C IP.

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
