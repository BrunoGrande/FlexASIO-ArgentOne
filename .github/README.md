[![English](https://img.shields.io/badge/lang-English-informational)](./README.md)
[![Português (BR)](https://img.shields.io/badge/idioma-Portugu%C3%AAs%20(BR)-blue)](./README.pt-BR.md)

# Argent One + FlexASIO + RS_ASIO Setup

This repository provides configuration and instructions for using the **Armer Argent One** audio interface with **FlexASIO** (universal ASIO driver) and **RS_ASIO** (Rocksmith 2014 patch to enable ASIO).

The goal of this setup is to allow **Simultaneous Audio**: running low-latency ASIO applications (Guitar Rig, Rocksmith) while keeping standard Windows audio (YouTube, Discord, Spotify) functioning properly.

* [RS_ASIO](https://github.com/mdias/rs_asio)
* [FlexASIO](https://github.com/dechamps/FlexASIO)
* [RSMods](https://github.com/Lovrom8/RSMods) (Recommended for faster startup)
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
The Argent One presents itself to Windows as a **2-channel Stereo device**. Rocksmith requires a specific Mono input. This configuration forces FlexASIO to match the Windows stereo format (preventing crashes) and uses RS_ASIO to route the correct channel (Left or Right) to the game.

---

## Prerequisites

* Windows PC
* **Armer Argent One** interface, installed and functional
* **FlexASIO** installed
* **RS_ASIO** files copied to Rocksmith folder
* **RSMods** (Highly recommended to skip the long intro video/wait time)

---

## Included Config Files

### 1. FlexASIO.toml
*Place this in your User folder (e.g., `C:\Users\BrunoGrande\FlexASIO.toml`)*

**Important:** We set `channels = 2` to match the Windows Default Format. Setting this to 1 usually causes `AUDCLNT_E_UNSUPPORTED_FORMAT` errors.

```toml
backend = "Windows WASAPI"
bufferSizeSamples = 512

[input]
device = "Microfone (Armer Argent)"
# Must be 2 to match Windows Stereo default
channels = 2
# Set to false to allow YouTube/Spotify to play while playing
wasapiExclusiveMode = false
wasapiAutoConvert = true

[output]
device = "Fones de ouvido (Armer Argent)"
# Must be 2 to match Windows Stereo default
channels = 2
wasapiExclusiveMode = false
wasapiAutoConvert = true
