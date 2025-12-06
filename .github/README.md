[![English](https://img.shields.io/badge/lang-English-informational)](./README.md)
[![Português (BR)](https://img.shields.io/badge/idioma-Portugu%C3%AAs%20(BR)-blue)](./README.pt-BR.md)

# Armer Argent One + FlexASIO + RS_ASIO Setup

This repository provides configuration and instructions for using the **Armer Argent One** audio interface with **FlexASIO** (universal ASIO driver) and **RS_ASIO** (Rocksmith 2014 patch to enable ASIO).

The goal of this setup is to allow the use of the Argent One interface with low latency in Rocksmith 2014, correcting common channel mismatch errors.

* [RS_ASIO](https://github.com/mdias/rs_asio)
* [FlexASIO](https://github.com/dechamps/FlexASIO)
* [RSMods](https://github.com/Lovrom8/RSMods) (Required for Fast Load)
* [FlexASIO GUI](https://github.com/flipswitchingmonkey/FlexASIO_GUI) (Optional)

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Included Config Files](#included-config-files)
4. [Setup Instructions](#setup-instructions)
5. [Testing & Latency Tuning](#testing--latency-tuning)
6. [Troubleshooting](#troubleshooting)
7. [Argent One Hardware Details](#argent-one-hardware-details)
8. [Credits & References](#credits--references)

---

## Overview

* **Argent One** is a 2-channel audio interface by Armer.
* **FlexASIO** bridges ASIO hosts to the Windows audio backend (WASAPI).
* **RS_ASIO** injects ASIO support into **Rocksmith 2014**.

**Why this specific config?**
The Argent One presents itself to Windows as a **2-channel Stereo device**. Rocksmith requires a specific Mono input. This configuration forces FlexASIO to match the Windows stereo format (preventing crashes) and uses RS_ASIO to route the correct physical channel (Left or Right) to the game functions.

---

## Prerequisites

* Windows PC
* **Armer Argent One** interface, installed and functional
* **FlexASIO** installed (latest version)
* **RS_ASIO** files copied to Rocksmith folder
* **RSMods** installed (Crucial to skip intro videos and prevent "white screen" hangs)

---

## Included Config Files

### 1. FlexASIO.toml
*Place this file in your User folder (e.g., `C:\Users\YOUR_NAME\FlexASIO.toml`)*

**Important:** We set `channels = 2` to match the Windows Default Format. Setting this to 1 usually causes `AUDCLNT_E_UNSUPPORTED_FORMAT` errors with this interface.

```toml
backend = "Windows WASAPI"
bufferSizeSamples = 512

[input]
device = "Microfone (Armer Argent)"
# Must be 2 to match Windows Stereo default
channels = 2
# Set to 'true' for lowest latency (Game Only). 
# Set to 'false' to allow YouTube/Spotify to play in background (Higher Latency).
wasapiExclusiveMode = true
wasapiAutoConvert = false

[output]
device = "Fones de ouvido (Armer Argent)"
# Must be 2 to match Windows Stereo default
channels = 2
wasapiExclusiveMode = true
wasapiAutoConvert = false
````

### 2\. RS\_ASIO.ini

*Place this file in your Rocksmith root folder.*

This config maps the specific hardware inputs of the Argent One:

  * **Input 0 (Left/Mic)** -\> Mapped to Rocksmith Microphone.
  * **Input 1 (Right/Inst)** -\> Mapped to Rocksmith Guitar Cable.

<!-- end list -->

```ini
[Config]
EnableWasapiOutputs=0
EnableWasapiInputs=0
EnableAsio=1

[Asio]
BufferSizeMode=driver
CustomBufferSize=

[Asio.Output]
Driver=FlexASIO
BaseChannel=0
EnableSoftwareEndpointVolumeControl=1
EnableSoftwareMasterVolumeControl=1

[Asio.Input.0]
; This is the GUITAR (Player 1)
; We use Channel=1 because Argent One Input 2 (Inst) is the Right channel
Driver=FlexASIO
Channel=1
EnableSoftwareEndpointVolumeControl=1
EnableSoftwareMasterVolumeControl=1

[Asio.Input.Mic]
; This is the VOCAL MIC
; We use Channel=0 because Argent One Input 1 (Mic) is the Left channel
Driver=FlexASIO
Channel=0
EnableSoftwareEndpointVolumeControl=1
EnableSoftwareMasterVolumeControl=1
```

-----

## Setup Instructions

1.  **Windows Sound Settings**

      * Open `mmsys.cpl` (Sound Control Panel).
      * Set Argent One as Default Playback and Recording device.
      * **Crucial:** In Properties \> Advanced, ensure both are set to **2 channel, 16 bit (or 24 bit), 48000 Hz**.

2.  **Install FlexASIO**

      * Download and install the latest release.

3.  **Install RS\_ASIO**

      * Copy `RS_ASIO.dll`, `RS_ASIO.ini` (content above), and `avrt.dll` to the Rocksmith folder.

4.  **Configure FlexASIO**

      * Create the `FlexASIO.toml` file with the content above in your user home folder (`C:\Users\YOUR_USERNAME\`).

5.  **Install RSMods (Fix Long Startup)**

      * Rocksmith has unskippable intro videos that run silently in the background. With custom ASIO drivers, this often looks like the game has crashed (white screen for \~20 seconds).
      * Install **RSMods**, go to the "Set and Forget" tab, and enable **Fast Load**. This removes the intro and makes startup instant.

6.  **Launch & Play**

      * Open Rocksmith. The game should start instantly and audio should work for both Guitar and Mic.

-----

## Testing & Latency Tuning

  * **Default Buffer:** `512` samples (Safe balance).
  * **Low Latency:** Try changing `bufferSizeSamples` in the TOML file to `256` or `192`.
      * If audio crackles, increase the number.
      * If audio is delayed, decrease the number.
  * **Exclusive Mode:** The provided config uses `wasapiExclusiveMode = true` for best performance. If you need to watch YouTube or use Discord while playing, change this to `false` in `FlexASIO.toml` (this may increase latency).

-----

## Troubleshooting

| Symptom | Likely Cause | Suggested Fix |
| :--- | :--- | :--- |
| **Game hangs on white screen** | Intro videos playing invisibly | Install **RSMods** and enable "Fast Load". |
| **Error: Unsupported Format** | Channel mismatch | Ensure `channels = 2` in TOML and Windows format is 48kHz Stereo. |
| **No Guitar Sound** | Wrong Input Channel | Check `RS_ASIO.ini`. Argent One Inst input is usually Channel 1 (Right). |
| **Audio Crackling** | Buffer too low | Increase `bufferSizeSamples` to 512 or 1024. |
| **YouTube video pauses** | Exclusive Mode conflict | Set `wasapiExclusiveMode = false` in TOML. |

-----

## Argent One Hardware Details

  * [Product Page](https://armer.com.br/produtos/interface-de-audio-usb-armer-argent-one/)
  * [Official Manual (PDF)](https://drive.google.com/file/d/15iVYalthiWtdguu4y76YLcfhcIaOtVLd/view?usp=sharing)

### Key Features

  * Two combo inputs (XLR / ¼” TRS).
  * \+48V Phantom Power (Channel 1).
  * Channel 2 switchable between **Instrument (INST)** and **Line**.

### Important Behavior Notes

**A) Mic (Ch1) - Left Channel**

  * Connector type: **3‑pin XLR**.
  * \+48 V Phantom Power available.
  * In Windows Stereo Mix, this appears as the **Left** side (Channel 0 in ASIO).

**B) Instrument (Ch2) - Right Channel**

  * Connector type: **¼” TS (Guitar Cable)**.
  * **Enable the INST button** for Guitars/Bass (High-Z).
  * In Windows Stereo Mix, this appears as the **Right** side (Channel 1 in ASIO).

**C) Monitoring**

  * **Direct Monitor ON:** You hear inputs directly (Zero Latency). Good for checking signal, but you might hear the "dry" guitar signal mixed with the game amp.
  * **Direct Monitor OFF:** You only hear the processed audio from the PC (Game/DAW). Recommended for Rocksmith.

-----

## Credits & References

  * **RS\_ASIO** by mdias
  * **FlexASIO** by dechamps
  * **RSMods** by Lovrom8
  * **Armer Argent One** Hardware info based on official manual.

<!-- end list -->
