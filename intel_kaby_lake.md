# Intel Kaby Lake-G (i7-8705G) - Ubuntu 26.04

## kaby_lake_g_ai_acceleration
 
This markdown report summarizes the entire technical journey to solve the unique hybrid 
hardware challenges of Intel Kaby Lake-G (i7-8705G) laptop running Ubuntu 26.04. It outlines how to bypass the 
deep sleep states of the AMD Radeon RX Vega M GL chip, handle driver isolation blocks within 
systemd, and ultimately achieve 100% compute offloading inside Ollama.

## Semantic Overview of the Documented Steps
* 
* Core Issue Identified: The Linux kernel aggressively suspend the AMD graphics lane to save 
  power (VK_ERROR_INCOMPATIBLE_DRIVER), while systemd isolates Ollama's background 
  daemon permissions from utilizing direct GPU rendering nodes.
* Kernel Fix: Apply amdgpu.runpm=0 to system GRUB variables, keeping the PCIe lanes awake 
  natively across system reboots.
* Driver Isolation & Masking: Target the exact open-source Mesa Vulkan library 
  (radeon_icd.json) and systematically isolate conflicting configurations.
* Daemon Configuration: Inject explicit DeviceAllow environment parameters into the 
  background system configuration, which allows the background server to handle calculations 
  directly on the discrete hardware instead of silently falling back to the CPU.
* 

Here is the clean chronological blueprint of the final working solution:

# Local AI Acceleration Guide & Conversation Summary
**System Architecture:** Intel Kaby Lake-G (Core i7-8705G) | Intel HD Graphics 630 iGPU | AMD Radeon RX Vega M GL dGPU
**Operating System:** Ubuntu 26.04.1 LTS (Linux Kernel 7.0)
**Core Engine:** Ollama (Llama 3.1 8B via Vulkan back-end)

---

## 📌 Executive Summary
This document outlines the step-by-step troubleshooting path and finalized solution used to achieve 
**100% hardware-accelerated local AI inference** on a highly specialized hybrid processor system. 
By resolving kernel-level power management bugs and bypassing systemd container isolation barriers, I successfully mapped 
**AMD Radeon RX Vega M GL** discrete GPU to process large language models, bringing the hardware load to 
**100% utilization** and offloading the burden from Intel CPU.

---

## 🛠️ Step-by-Step Technical Resolution
### Phase 1: Overcoming Kernel Power State Blocks (Waking the GPU)
*   **The Problem:** The Linux kernel aggressively suspending the AMD graphics lane to save power, resulting in a persistent 
    `VK_ERROR_INCOMPATIBLE_DRIVER` error when apps attempted to access `/dev/dri/renderD129`.
*   **The Solution:** Force the kernel to bypass runtime power management for the `amdgpu` driver layer.
    1. Open the system bootloader profile:
       ```bash
       sudo nano /etc/default/grub
       ```
    2. Append `amdgpu.runpm=0` to the default line:
       ```text
       GRUB_CMDLINE_LINUX_DEFAULT="quiet splash usbcore.autosuspend=-1 amdgpu.runpm=0"
       ```
    3. Update the boot records and force a system restart:
       ```bash
       sudo update-grub
       systemctl reboot -i
       ```
### Phase 2: User Account Permissions & Driver Isolation
*   **The Problem:** Standard user shell accounts lack direct hardware access privileges to open system rendering interfaces. 
    Additionally, Vulkan struggled with conflicting third-party driver JSON layouts (like `asahi`, `virtio`, or `nouveau`).
*   **The Solution:** Grant proper group clearance and map Vulkan strictly to the native open-source Mesa RADV layout.
    1. Bind your user account to the physical video and rendering nodes:
       ```bash
       sudo usermod -aG video,render \$USER
       ```
    2. Force the session profile to look strictly at the AMD driver map:
       ```bash
       echo 'export VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/radeon_icd.json' >> ~/.zshrc
       source ~/.zshrc
       ```
    3. Verify system status via terminal command:
       ```bash
       vulkaninfo --summary
       ```
       *Result:* **GPU0** registers cleanly as `AMD Radeon RX Vega M GL Graphics (RADV VEGAM)` without errors.

### Phase 3: Unlocking the Ollama Background Daemon
*   **The Problem:** Ollama runs as a background service (`systemd`). The isolated system user account (`ollama`) runs 
    inside a tight sandbox that hides graphics hardware layers, forcing models to drop down to slow CPU fallback loops.
*   **The Solution:** Inject explicit environment parameters and read/write hardware gate instructions directly into the service layer.
    1. Open the system service override file:
       ```bash
       sudo systemctl edit ollama.service
       ```
    2. Insert the following environment block:
       ```ini
       [Service]
       Environment="VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/radeon_icd.json"
       Environment="OLLAMA_VULKAN_DEVICE=0"
       DeviceAllow=/dev/dri/renderD128 rw
       DeviceAllow=/dev/dri/renderD129 rw
       ```
    3. Grant direct permission to the background service daemon profile:
       ```bash
       sudo usermod -aG video,render ollama
       ```
    4. Reload parameters and restart the execution loop:
       ```bash
       sudo systemctl daemon-reload
       sudo systemctl restart ollama.service
       ```

---

## 📊 Performance Verification & Status Log
To review performance metrics, launch the model with the active benchmarking tracker:

```bash
ollama run llama3.1:latest --verbose
```

### Real-Time Validation using `nvtop`
Open a separate window panel and execute `nvtop`. Your system now successfully displays:
*   **VRAM Footprint:** `~2.888 GiB / 4.000 GiB` completely allocated on **Device 0 [AMD Radeon]**, containing the model context.
*   **GPU Core Engagement:** The **GPU0 % engine bar pins tightly to 100%** during inference loops, proving perfect hardware offload.

-----------------------------
