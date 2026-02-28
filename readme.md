# RP2350 HUB75 AGP (Active WIP)

> Autonomous HUB75 graphics pipeline for RP2350. [Public Tracker & Documentation]

![Status: Active Development](https://img.shields.io/badge/Status-Active_Development-orange)
![MCU: RP2350](https://img.shields.io/badge/MCU-RP2350-blue)

A next-generation, high-performance HUB75 LED matrix pipeline built from the ground up for the Raspberry Pi RP2350.

This engine maximizes panel refresh rates and color depth through the use of highly optimized ASM bit-shuffling and a targeted RAM footprint. At the hardware level, it utilizes cascaded PIO state machines linked via a DMA bridge, using ping-pong signaling to guarantee perfect synchronization.

## 🚀 Key Features

* **Multi-Channel Drive:** Designed to push up to 2 parallel HUB75 chains simultaneously.
* **Zero-Bottleneck Pipeline:** Employs a highly optimized data flow that sidesteps traditional memory bottlenecks, keeping the hardware fed at the maximum physical speeds the panels can handle.
* **Advanced Sub-Frame Dithering:** Utilizes a custom hybrid modulation scheme to bypass the physical Output Enable (OE) limits of standard panels, achieving superior color depth without the typical global refresh rate penalties.
* **Ultra-High Refresh:** Maintains rock-solid, flicker-free updates even under heavy rendering loads.
* **Smart PIO Orchestration:** Uses a tightly coupled, multi-state-machine architecture to handle routing, precise microsecond timing, and dynamic masking autonomously.

## 🎨 Supported Color Formats

The pipeline natively handles multiple pixel formats, automatically mapping them through a high-precision gamma correction curve:

* **8-bit Palette**
* **RGB332**
* **RGB565:** Resolves up to **RGB101010** (30-bit color) post-gamma.
* **RGB888:** Resolves up to **RGB121212** (36-bit color) post-gamma.

## 🪟 Viewport & Buffer Management

The engine decouples the physical display size from the memory buffer size, allowing for zero-overhead hardware panning:

* **Oversized Buffers:** Natively supports RGB backing buffers that are physically larger than the active display matrix.
* **Relative Offset Addressing:** Move the display window across the larger buffer by simply changing the offset coordinates.
* **Seamless Wrapping:** If the display window coordinates exceed the physical boundaries of the RGB buffer, the addressing automatically wraps to the start of the row or column.

## 📐 Hardware Support & Wiring Topologies

This engine supports typical indoor HUB75 panels with a **SCAN = HEIGHT / 2** multiplexing ratio (e.g., 1:16 for 32px height, 1:32 for 64px height).

> **Note on Compatibility:** I am currently developing exclusively for the panels I have in stock. Need support for 1/4 scan or other exotic multiplexing? Send me the panels.

### ⚙️ Matrix Configuration Rules

* **Daisy-Chaining:** Panels can be chained in both horizontal and vertical directions.
* **Channel A (Primary):** The starting point is always the **top-left** corner of the display.
* **Channel B (Secondary):** Must have an identical panel configuration to Channel A. It can be mapped either directly to the right (horizontal alignment) or directly below (vertical alignment) Channel A.

### 🚥 The Channel Rules (Legend)

* 🟦 **1 Channel Mandatory:** Asymmetric panel counts or low-bandwidth configurations. The entire matrix must be daisy-chained and driven by a single channel (Channel A).
* 🟪 **1 or 2 Channels (Optional):** Symmetrical splits with moderate bandwidth. You can daisy-chain all panels to Channel A, or wire half to Channel A and half to Channel B to double your refresh rate.
* 🟩 **2 Channels Mandatory (The Heavyweights):** Maximum bandwidth reached. To meet VSYNC deadlines and prevent matrix flickering, you **must** split the workload perfectly in half across Channel A and Channel B.

---

### 🧩 Base Panel: 32x16px
| Height \ Width | 32px | 64px | 96px | 128px | 160px | 192px | 256px |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **16px** | 🟦 1 Ch | 🟪 Optional | 🟦 1 Ch | 🟪 Optional | 🟩 2 Ch | 🟩 2 Ch | 🟩 2 Ch |
| **32px** | 🟪 Optional | 🟩 2 Ch | 🟩 2 Ch | 🟩 2 Ch | - | - | - |
| **48px** | 🟦 1 Ch | 🟩 2 Ch | - | - | - | - | - |
| **64px** | 🟪 Optional | 🟩 2 Ch | - | - | - | - | - |
| **80px** | 🟩 2 Ch | - | - | - | - | - | - |
| **96px** | 🟩 2 Ch | - | - | - | - | - | - |
| **128px**| 🟩 2 Ch | - | - | - | - | - | - |

### 🧩 Base Panel: 32x32px
| Height \ Width | 32px | 64px | 96px | 128px | 192px | 256px |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **32px** | 🟦 1 Ch | 🟪 Optional | 🟦 1 Ch | 🟪 Optional | 🟩 2 Ch | 🟩 2 Ch |
| **64px** | 🟪 Optional | 🟪 Optional | 🟩 2 Ch | 🟩 2 Ch | - | - |
| **96px** | 🟦 1 Ch | 🟩 2 Ch | - | - | - | - |
| **128px**| 🟪 Optional | 🟩 2 Ch | - | - | - | - |
| **192px**| 🟩 2 Ch | - | - | - | - | - |
| **256px**| 🟩 2 Ch | - | - | - | - | - |

### 🧩 Base Panel: 64x32px
| Height \ Width | 64px | 128px | 256px |
| :--- | :--- | :--- | :--- |
| **32px** | 🟦 1 Ch | 🟪 Optional | 🟩 2 Ch |
| **64px** | 🟪 Optional | 🟩 2 Ch | - |
| **128px**| 🟩 2 Ch | - | - |
| **256px**| 🟩 2 Ch | - | - |

### 🧩 Base Panel: 64x64px
| Height \ Width | 64px | 128px | 256px |
| :--- | :--- | :--- | :--- |
| **64px** | 🟦 1 Ch | 🟪 Optional | 🟩 2 Ch |
| **128px**| 🟪 Optional | 🟩 2 Ch | - |
| **256px**| 🟩 2 Ch | - | - |

### 🧩 Base Panel: 96x48px
| Height \ Width | 96px | 192px |
| :--- | :--- | :--- |
| **48px** | 🟦 1 Ch | 🟩 2 Ch |
| **96px** | 🟩 2 Ch | - |

## 🛠️ Current Status

The core rendering engine, advanced temporal dithering math, and PIO timing logic are currently in active development in a private repository. 

Source code and API documentation will be published here upon completion.

---
*Developed by scLLptr*