# BFSK_sim
Simulate communication network on a noise free channel with BFSK modulation using a header for simple protocol implementation

# 📡 FSK Audio Transmission & Reception System

## Overview
This project implements a complete **2-FSK (Frequency Shift Keying)** digital communication chain in **GNU Radio**, capable of transmitting audio as binary data over an FSK-modulated link and recovering it at the receiver. The channel is noiseless.

It covers key digital communications concepts:
- Symbol mapping for modulation
- Passband to baseband conversion
- Low-pass and matched filtering
- Bit and frame synchronization
- Protocol framing
- Real-time audio reconstruction

---

## Why the Design Works
This system is robust because it:
1. **Uses Orthogonal FSK Tones** – Bits `0` and `1` are mapped to distinct carrier frequencies (`f0`, `f1`), minimizing inter-symbol interference.
2. **Downconverts to Baseband** – Simplifies processing by bringing signals to a lower frequency range.
3. **Filters Effectively** – Removes high-frequency artifacts with an LPF, and maximizes SNR with a matched filter.
4. **Synchronizes via Headers** – Fixed 48-bit headers allow precise byte/frame alignment.
5. **Integrates in a Reusable Block** – The modulator/demodulator chain is encapsulated in a **Hierarchical Block** (`new_hier`) for modularity.

---
# 📂 Module Breakdown

## 1. Audio Input & Bit Conversion
- **WAV File Source** – Reads 16 kHz mono audio samples from a `.wav` file.  
- **Float to Char** – Converts float samples (−1 to +1) into 8-bit integers (−128 to +127).  
- **Unpack K Bits** – Expands each byte into its 8 individual bits for transmission.  

---

## 2. Protocol Framing
- **Header Generator** – Adds a fixed **48-bit synchronization header** to each frame for byte alignment and error recovery.  
- **Tagged Stream Mux** – Combines headers and payload bits into a continuous tagged stream.  

---

## 3. FSK Modulation (Inside `new_hier`)
- **Bit-to-Tone Mapping**:  
  - `0` → cosine at **f0** = 8 cycles/symbol  
  - `1` → cosine at **f1** = 9 cycles/symbol  
- **Symbol Duration** – 72 samples/symbol.
- **Sampling Rate calculation** : 16000 symbols/sec * 8 bits / symbol * 72 samples / bit = 9.216 Mega Samples / sec
- **Passband Sampling Rate** i.e. **pb_rate = 9.216 MHz**.  

---

## 4. FSK Demodulation (Inside `new_hier`)
- **Mixing Stage** – Downconverts **f0** tone to DC.  
- **Low-Pass Filter (LPF)** – Removes high-frequency mixing artifacts.  
- **Matched Filter** – Maximizes SNR by correlating with expected tone shape.  
- **Decimation** – Outputs one decision per symbol period.  
- **Delay Compensation** – Aligns symbol sampling with matched filter peaks.  

---

## 5. Header Detection & Payload Extraction
- **Correlator** – Detects the unique 12-bit sync pattern in each header(2728 = 101010101000 thus it should detect the reverse of it i.e. 000101010101)  
- **Tag Share + Header/Payload Demux** – Splits the header and payload streams for separate processing.  

---

## 6.  Audio Reconstruction
- **Pack K Bits** – Groups recovered every 8 bits into bytes.  
- **Char to Float** – Converts bytes back to floating-point audio samples.  
- **Audio Sink** – Plays the reconstructed audio in real time.  
- **QT GUI Time Sink** – Visualizes the recovered waveform. 
