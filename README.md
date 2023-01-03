# SDR FM Radio Demodulator
## Introduction
An implementation of a FM demodulator that supports
- Mono audio using L+R signal
- Stereo audio using L-R signal
- RDS decoding 

![Screenshot](docs/window_screenshot.png)

As much of the internal state of the demodulator is shown for academic purposes. This program is intended to be an educational tool. 

SIMD is employed to improve DSP performance on x86 processors. 

## Instructions
1. Download program from releases page.
2. Unzip and extract folder.
3. Run <code>./bin/rtl_sdr.exe -f [fm_frequency] -s 1.024e6 -g 20 | ./main.exe</code>

## Explanation
FM radio contains many data components which are present after FM demodulation.
| Frequencies | Description |
| --- | --- |
| 0 - 15kHz  | L+R audio |
| 19kHz      | Pilot tone |
| 38 ± 15kHz | L-R audio |
| 57 ± 2kHz  | RDS (radio data system) |

![FM Spectrum](docs/fm_spectrum.png)

To compensate for frequency and phase offsets due to errors in the receiver or transmitter we need to lock onto the pilot tone. This is done with a PLL.

To downconvert the L-R and RDS signals we use harmonics of the PLL output. The L-R signal is downconverted using the second harmonic, and the RDS signal is downconverted by the third harmonic. In an analogue circuit this can be easily done by squaring the 19kHz PLL output and filtering out the desired harmonics from the square wave.

### Stereo Audio
Stereo audio can be generated by combining the mono L+R audio with the L-R audio.
```c
L = (L+R) + (L-R) = 2L
R = (L+R) - (L-R) = 2R
```

### Radio Data System (RDS)
The RDS signal is a binary phase shift keyed signal. This is passed through a BPSK symbol synchroniser to get constellation points in the range of -1 to 1.

The RDS signal contains the following data described [here](https://en.wikipedia.org/wiki/Radio_Data_System).

The document used to decode the standard (partially) is located [here](docs/EN50067_RDS_Standard.pdf).

It usually contains the programme identifier code and additional descriptive text and metadata.