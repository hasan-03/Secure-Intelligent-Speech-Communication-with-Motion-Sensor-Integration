# Secure-Intelligent-Speech-Communication-with-Motion-Sensor-Integration

> Real-time Android → TCP/IP pipeline that captures audio and motion, **scrambles audio in the frequency domain with a key-based permutation**, transmits synchronized audio+motion as fixed-size byte packets, and reconstructs the original audio at the receiver only when the same key is used.

---

## Table of Contents
- [Project Overview](#project-overview)  
- [Features](#features)  
- [Processing Pipeline](#processing-pipeline)  
- [Packet Format](#packet-format)  
- [Scrambling: Algorithm & Math](#scrambling-algorithm--math)  
- [Unscrambling (Inverse) & Modular Inverse (Bézout)](#unscrambling-inverse--modular-inverse-b%C3%A9zout)  
- [Hermitian Symmetry (Why we enforce it)](#hermitian-symmetry-why-we-enforce-it)  
- [Worked Example (N = 10)](#worked-example-n--10)  
- [Key Functions & Code Snippets](#key-functions--code-snippets)  
- [How to Run (Quick Start)](#how-to-run-quick-start)  
- [Security Notes & Limitations](#security-notes--limitations)  
- [Authors & License](#authors--license)

---

## Project Overview

This project implements a Simulink-based system (Android target) that:

- Captures audio (stereo) and motion (accelerometer + gyroscope).
- Converts audio frames to the frequency domain (FFT), applies a **key-derived linear congruential permutation** to the positive-frequency bins, restores Hermitian symmetry, and converts back (IFFT).
- Packs motion + scrambled audio into a single `uint8` packet and sends it over TCP/IP.
- The receiver unpacks, performs FFT, applies the inverse permutation (using the modular inverse of the scrambling multiplier), restores Hermitian symmetry, and IFFT to recover the original audio — only possible with the correct key.

---

## Features

- Real-time Android audio capture at **44.1 kHz**, frame size **4410 samples** (0.1 s frames).  
- Hann window → `FFT` → key-based frequency permutation → `IFFT` pipeline.  
- Preserves **Hermitian symmetry** so `IFFT` yields a real signal.  
- Packs synchronized motion (6 × `single`) + audio (`int16`) into fixed **8844-byte** packets for TCP/IP.  

---

## Processing Pipeline

1. **Capture**: Android Audio Capture (stereo, `int16`, frame size 4410) + motion sensors (6 × `single`).  
2. **Select**: Use Selector to extract one channel (mono) — `u = u(:,1)`.  
3. **Window**: Apply Hann window to frame.  
4. **FFT**: `X = fft(double(u))`.  
5. **Scramble (sender)**: Compute `(a,b)` from integer `key`, permute positive-frequency bins, enforce Hermitian symmetry.  
6. **IFFT**: `y = real(ifft(Y))`.  
7. **Pack & Transmit**: Pack motion + audio into `uint8` packet and send via TCP/IP.  
8. **Receive & Unpack**: Unpack packet into `single` motion and `int16` audio.  
9. **Unscramble (receiver)**: `FFT` → compute `a^{-1}` (mod K) → apply inverse permutation → restore conjugate symmetry → `IFFT`.

---

## Packet Format

- Motion: `6 × single` → **24 bytes**.  
- Audio: `4410 × int16` → **8820 bytes**.  
- Total packet size: **8844 bytes** (24 + 8820).  
- Packet is transmitted as `uint8`:

## Unscrambling — Modular Math, Bézout & `a^{-1}` (Detailed Explanation)

When the scrambler applies the linear congruential permutation
\[
p(i) \equiv (a\cdot i + b)\ \bmod\ K,\qquad i\in\{0,\dots,K-1\},
\]
the receiver must compute the inverse mapping to recover the original index `i` from the scrambled index `j`. The inverse mapping is

\[
i \equiv a^{-1}\cdot (j - b)\ \bmod\ K,
\]

where \(a^{-1}\) is the **modular multiplicative inverse** of \(a\) modulo \(K\); i.e. the integer satisfying
\[
a\cdot a^{-1} \equiv 1 \pmod K.
\]

### Why the modular inverse exists
The modular inverse \(a^{-1}\) exists **iff** \(\gcd(a,K)=1\). This is guaranteed in the scrambler by incrementing `a` until it is coprime with `K`. When \(\gcd(a,K)=1\), Bézout’s identity ensures integers \(u,v\) exist such that
\[
a\cdot u + K\cdot v = 1.
\]
Reducing both sides modulo \(K\) gives \(a\cdot u \equiv 1 \pmod K\), so \(u\) is a modular inverse of \(a\). The extended Euclidean algorithm (or MATLAB’s `gcd` with Bézout outputs) is used to compute such a coefficient `u`. The positive representative is then `inv_a = mod(u, K)`.

## Licence:
- Hasan Ali
- Krish Patel
- Kshitij Agrawal


