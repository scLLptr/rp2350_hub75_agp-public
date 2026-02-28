# RP2350 HUB75 AGP (Active WIP)

> Autonomous HUB75 graphics pipeline for RP2350. [Public Tracker & Documentation]

![Status: Active Development](https://img.shields.io/badge/Status-Active_Development-orange)
![MCU: RP2350](https://img.shields.io/badge/MCU-RP2350-blue)

A next-generation, high-performance HUB75 LED matrix pipeline built from the ground up for the Raspberry Pi RP2350.

This engine maximizes panel refresh rates and color depth through the use of highly optimized ASM bit-shuffling and a targeted RAM footprint. At the hardware level, it utilizes cascaded PIO state machines linked via a DMA bridge, using ping-pong signaling to guarantee perfect phase synchronization.

## 📜 Project Evolution & History

The **RP2350 HUB75 AGP** is my second iteration tackling the HUB75 problem on the Pico platform - hopefully a less naive attempt than the first.

### 📅 2024: The RP2040 Precursor
Developed during a 2024 New Year's break, the first-gen driver established the "Single Stream" philosophy for the RP2040:
* **Interlaced Framebuffer:** A single DMA stream handled the entire matrix by interleaving RGB data, row selection, and timing markers into a single memory block.
* **BCM Engine:** 8-to-10 bitplane Binary Code Modulation.
* **Cascaded PIO:** Proved the concept of linking state machines via a DMA bridge for zero-CPU rendering.

### 📺 Demonstration of the 2024 Engine (96x96 Panel)

| Link |
| :--- |
| [Watch v1.mp4](https://i.imgur.com/oINd6zx.mp4) |
| [Watch v2.mp4](https://i.imgur.com/piWeAsn.mp4) |
| [Watch v3.mp4](https://i.imgur.com/8Q26J8F.mp4) |
| [Watch v4.mp4](https://i.imgur.com/vfu7Pvi.mp4) |

### 🚀 2026: The RP2350 Autonomous Pipeline (Key Features)

* **Multi-Channel Drive:** Designed to drive up to 2 parallel HUB75 chains simultaneously.
* **Zero-Bottleneck Pipeline:** Employs a highly optimized data flow that sidesteps traditional memory bottlenecks, keeping hardware fed at maximum physical panel speeds.
* **Advanced Sub-Frame Dithering:** Utilizes a custom hybrid modulation scheme to bypass physical Output Enable (OE) limits, achieving superior color depth without global refresh rate penalties.
* **Core Isolation:** Resides on Core 1, utilizing 3 dedicated 64kB SRAM banks for maximum throughput and zero bus contention.
* **Smart PIO Orchestration:** Inherits the DMA-bridged PIO architecture and adds a novel autonomous rendering solution.

## 🎨 Supported Color Formats

The pipeline natively handles multiple pixel formats, mapping them through a high-precision gamma correction curve:

* **8-bit Palette / RGB332** Resolves up to **RGB101010** (30-bit color) post-gamma.
* **RGB565:** Resolves up to **RGB101010** (30-bit color) post-gamma.
* **RGB888:** Resolves up to **RGB121212** (36-bit color) (RGB212121 - 63 bit theoretical) post-gamma.

Gamma coefficient is configurable in range 2.0 to 3.0.

## 🪟 Viewport & Buffer Management

The engine decouples physical display size from memory buffer size, allowing for zero-overhead hardware panning:

* **Oversized Buffers:** Natively supports RGB backing buffers physically larger than the active display matrix.
* **Relative Offset Addressing:** Pan the display window across the buffer by simply updating offset coordinates.
* **Seamless Wrapping:** Automatic row/column wrapping when the viewport exceeds physical buffer boundaries.
* **Mandatory Registration of RGB Buffers:** The engine validates color format, dimensions, and memory size prior to use; specifically, it enforces strict byte alignment to guarantee the atomic and high-speed operation of LDRD and LDMIA instructions within the ASM pipeline.

## 📐 Hardware Support & Wiring Topologies

Supports typical indoor HUB75 panels with a **SCAN = HEIGHT / 2** multiplexing ratio (e.g., 1:16 for 32px height, 1:32 for 64px height).

> **Note on Compatibility:** Development is currently limited to the panels I have in stock. 

### ⚙️ Matrix Configuration Rules

* **Daisy-Chaining:** Panels can be chained in both horizontal and vertical directions.
* **Channel A (Primary):** The starting point is always the **top-left** corner of the display.
* **Channel B (Secondary):** Must have an identical panel configuration to Channel A. It can be mapped either directly to the right (horizontal alignment) or directly below (vertical alignment) Channel A.

### 🚥 The Channel Rules (Legend)

* 🟦 **1 Channel Mandatory:** Asymmetric panel counts or low-bandwidth. Driven by Channel A.
* 🟪 **1 or 2 Channels (Optional):** Symmetrical splits with moderate bandwidth. You can daisy-chain all panels to Channel A, or wire half to Channel A and half to Channel B to double your refresh rate.
* 🟩 **2 Channels Mandatory (The Heavyweights):** Maximum bandwidth reached. To meet VSYNC deadlines and prevent matrix flickering, you **must** split the workload perfectly in half across Channel A and Channel B.

---

### 🧩 Topology Reference Tables

#### Base Panel: 32x16px
| Height \ Width | 32px | 64px | 96px | 128px | 160px | 192px | 256px |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **16px** | 🟦 1 Ch | 🟪 Optional | 🟦 1 Ch | 🟪 Optional | 🟩 2 Ch | 🟩 2 Ch | 🟩 2 Ch |
| **32px** | 🟪 Optional | 🟩 2 Ch | 🟩 2 Ch | 🟩 2 Ch | - | - | - |
| **48px** | 🟦 1 Ch | 🟩 2 Ch | - | - | - | - | - |
| **64px** | 🟪 Optional | 🟩 2 Ch | - | - | - | - | - |
| **80px** | 🟩 2 Ch | - | - | - | - | - | - |
| **96px** | 🟩 2 Ch | - | - | - | - | - | - |
| **128px**| 🟩 2 Ch | - | - | - | - | - | - |

#### Base Panel: 32x32px
| Height \ Width | 32px | 64px | 96px | 128px | 192px | 256px |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **32px** | 🟦 1 Ch | 🟪 Optional | 🟦 1 Ch | 🟪 Optional | 🟩 2 Ch | 🟩 2 Ch |
| **64px** | 🟪 Optional | 🟪 Optional | 🟩 2 Ch | 🟩 2 Ch | - | - |
| **96px** | 🟦 1 Ch | 🟩 2 Ch | - | - | - | - |
| **128px**| 🟪 Optional | 🟩 2 Ch | - | - | - | - |
| **192px**| 🟩 2 Ch | - | - | - | - | - |
| **256px**| 🟩 2 Ch | - | - | - | - | - |

#### Base Panel: 64x32px
| Height \ Width | 64px | 128px | 256px |
| :--- | :--- | :--- | :--- |
| **32px** | 🟦 1 Ch | 🟪 Optional | 🟩 2 Ch |
| **64px** | 🟪 Optional | 🟩 2 Ch | - |
| **128px**| 🟩 2 Ch | - | - |
| **256px**| 🟩 2 Ch | - | - |

#### Base Panel: 64x64px
| Height \ Width | 64px | 128px | 256px |
| :--- | :--- | :--- | :--- |
| **64px** | 🟦 1 Ch | 🟪 Optional | 🟩 2 Ch |
| **128px**| 🟪 Optional | 🟩 2 Ch | - |
| **256px**| 🟩 2 Ch | - | - |

#### Base Panel: 96x48px
| Height \ Width | 96px | 192px |
| :--- | :--- | :--- |
| **48px** | 🟦 1 Ch | 🟩 2 Ch |
| **96px** | 🟩 2 Ch | - |

## 🛠️ Current Status

The core rendering engine, advanced temporal dithering math, and PIO timing logic are currently in active development in a private repository. Source code and API documentation will be published here upon completion.

### 📂 Historical Archive: 2024 C-Shuffling Logic

<details>
<summary>Expand Legacy Code</summary>

```c
void __no_inline_not_in_flash_func(conv2hub75)(rgb* drgbi, uint32_t* fb, bool half){
    //assert HUB75_PARALLEL_CHANS == 1 || HUB75_PARALLEL_CHANS == 2  
    const uint32_t r1g1b1_width = 3;
    const uint32_t column_step = 4 / HUB75_PARALLEL_CHANS;
    const uint32_t hub75_word_width = 8 * HUB75_PARALLEL_CHANS;
    const uint32_t hub75_word_data_width = 6 * HUB75_PARALLEL_CHANS;
    const uint32_t parallel_pixels = HEIGHT / SCAN;
    const uint32_t end_bit_pos = 31;
    uint32_t addr_hub75 = HUB75STATUSSIZE + ((half) ? HUB75SIZE / 2 : 0);
    //assert column_step * parallel_pixels == 8
    for (uint32_t row = 0; row < SCAN/2; row++) {
      uint32_t row_ = (half) ? row + SCAN / 2 : row;
        for (uint32_t pixel_depth = 0; pixel_depth < PDPTH; pixel_depth++) {
            const uint32_t r1g1b1_shift = 3 * pixel_depth;
            const uint32_t r1g1b1_mask = (((1 << r1g1b1_width) - 1) << r1g1b1_shift);
            const uint32_t control_word = ((TIME_BASE << (pixel_depth + HUB75_SCAN_CONTROL_BITS))) | row_;
            for (uint32_t column = 0; column < WIDTH; column += column_step) {
                uint32_t framebuffer_word = 0;
                const int32_t dif = column + 32 - WIDTH;              
                for (uint32_t idx = 0; idx < parallel_pixels; idx++)         
                    framebuffer_word |= (((drgbi[WIDTH * row_ + idx * SCAN * WIDTH + column] & r1g1b1_mask) >> r1g1b1_shift) << r1g1b1_width * idx) + 
                        (((drgbi[WIDTH * row_ + idx * SCAN * WIDTH + column + 1] & r1g1b1_mask) >> r1g1b1_shift) << (r1g1b1_width * idx + hub75_word_width)); 
                if(dif>=0)
                    framebuffer_word |= (((control_word >> dif) & 1) << hub75_word_data_width) + (((control_word >> dif) & 2) << (hub75_word_data_width + hub75_word_width - 1));      
                fb[addr_hub75++] = framebuffer_word;
            }
            fb[addr_hub75 - 1] |= 1 << end_bit_pos;
        }  
    }
}