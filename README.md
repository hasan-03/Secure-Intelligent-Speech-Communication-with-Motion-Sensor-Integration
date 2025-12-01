# Secure-Intelligent-Speech-Communication-with-Motion-Sensor-Integration

> Real-time Android → TCP/IP pipeline that captures high-fidelity audio and motion sensor data, secures transmission using Multi-Method Encryption (Frequency Scrambling, Adaptive Masking, Watermarking), transmits synchronized fixed-size byte packets, and performs real-time authentication and decryption at the receiver.


## Project Overview

This project implements a Simulink-based system (Android target) designed for secure, synchronized communication between two mobile devices. It features:

- **Data Acquisition:** Captures stereo Audio (44.1kHz) and Motion data (Accelerometer + Gyroscope) simultaneously.
- **Multi-Mode Security:**
- *Method 1 (Frequency Scrambling):* Applies a key-derived Linear Congruential Generator (LCG) permutation to FFT bins to scramble speech intelligibility.
- *Method 2 (Adaptive Noise Masking):* Generates synthetic pink noise shaped by the speech spectral envelope to mask content. The receiver regenerates the exact noise profile using the shared key to subtract it.
- *Method 3 (Watermarking):* Embeds an inaudible, spread-spectrum noise signature (200Hz–15kHz) into the audio. The receiver validates this signature using Normalized Cross-Correlation to authenticate the source.
- **Efficient Transmission:** Syncs 6 float values (Motion) and 4410 int16 samples (Audio) into a single 8844-byte packet sent over TCP/IP (Port 25000).
- **Receiver Architecture:** Unpacks data, validates the watermark (Latching Logic), decrypts the audio based on the selected method, computes Blind SNR for quality assessment, and visualizes motion data in real-time.

## Overall Design Pipeline: 
Transmitter (Phone A):
- Record: Audio (44.1kHz) & Motion Sensors.
- Process: Convert Audio to Frequency Domain (FFT).
- Encrypt: Apply selected method (Scramble / Mask / Watermark) using Key-based Seed.
- Pack: Quantize Audio to int16, serialize with single-precision Motion data.
- Transmit: Send 8844-byte packets via TCP/IP.

Receiver (Phone B):
- Receive: Capture & Unpack byte stream.
- Restore: Convert int16 audio back to double & Apply FFT.
- Authenticate: Verify Watermark using Spread Spectrum Correlation (Latch Output).
- Decrypt: Reverse the encryption logic (Unscramble / Subtract Noise) to recover speech.
- Output: Playback Audio, Display SNR, and Plot Motion Vectors.

## Licence:
- Hasan Ali
- Krish Patel
- Kshitij Agrawal


