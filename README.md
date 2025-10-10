# Secure-Intelligent-Speech-Communication-with-Motion-Sensor-Integration

> Real-time Android → TCP/IP pipeline that captures audio and motion, **scrambles audio in the frequency domain with a key-based permutation**, transmits synchronized audio+motion as fixed-size byte packets, and reconstructs the original audio at the receiver only when the same key is used.


## Project Overview

This project implements a Simulink-based system (Android target) that:

- Captures audio (stereo) and motion (accelerometer + gyroscope).
- Converts audio frames to the frequency domain (FFT), applies a **key-derived linear congruential permutation** to the positive-frequency bins, restores Hermitian symmetry, and converts back (IFFT).
- Packs motion + scrambled audio into a single `uint8` packet and sends it over TCP/IP.
- The receiver unpacks, performs FFT, applies the inverse permutation (using the modular inverse of the scrambling multiplier), restores Hermitian symmetry, and IFFT to recover the original audio — only possible with the correct key.

## Overall Design Pipeline: 
Record audio & motion data → Apply FFT → Generate key-based permutation → Scramble frequency components → Apply IFFT → Transmit audio & motion data → Receiver FFT → Generate same permutation → Apply inverse permutation & IFFT → Recover audio & display motion data 

## Licence:
- Hasan Ali
- Krish Patel
- Kshitij Agrawal


