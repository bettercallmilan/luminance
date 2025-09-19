# Luminance

A project dedicated to reverse engineering the Zelotes F-35C mouse to enable custom control over its onboard OLED screen and other features.

> The primary goal is to understand the communication protocol between the official software and the mouse, with the aim of creating custom animations or useful displays for the OLED screen.

---

## Target Device

* **Device:** Zelotes F-35C Gaming Mouse
* **Features:** RGB lighting, onboard OLED screen, multiple DPI profiles.
* **Software:** `Zelotes F-35C Mouse.exe`

---

## Current Progress & Findings

My investigation has proceeded in two main phases: passive USB traffic analysis and attempted software decompilation.

### 1. USB Packet Analysis (Wireshark)

Using **Wireshark** with **USBPcap**, I successfully captured the USB HID packets sent from the host software to the mouse when changing settings.

#### Key Findings:

* **Set DPI Command:** I have successfully identified the command to change the mouse's DPI.
    * **Direction:** `OUT` (Host -> Device)
    * **Report ID:** `0x04`
    * **Command Structure:** `04 20 00 [DPI_Low] [DPI_High] ...`
    * **Example (1600 DPI):**
        ```hex
        # HID Data Packet for setting DPI to 1600
        04 20 00 1a 06 00 00 00 ...
        |  |  |  |  |
        |  |  |  +--+-- DPI Value (0x061a = 1562, the internal value for 1600 DPI)
        |  |  +-------- Parameter (0x00 = DPI)
        |  +----------- Command (0x20 = Set Value)
        +-------------- Report ID
        ```

* **Set RGB Lighting Command:** I captured a more complex packet, initially believed to be for the OLED screen. I've since concluded this is for the RGB lighting effects.
    * **Command Code:** `0x61`
    * **Structure:** `04 61 ... [Length] ... [DATA]`
    * **Note:** The long data payload likely defines complex lighting effects like colors, speed, and patterns.

* **Device Confirmation Packet:** The mouse sends a confirmation packet back to the host after receiving a command.
    * **Direction:** `IN` (Device -> Host)
    * **Example Response:** `04 01 00 01 00 ...`

### 2. Software Analysis

I attempted to decompile the official software to read the source code directly.

* **Attempt 1 (.NET Decompilation):**
    * **Tools:** ILSpy
    * **Files Analyzed:** `DuiLib.dll`, `HookDLL.dll`, `zelotes F-35C mouse.exe`
    * **Result:** **Failed**. The tool returned a `PE file does not contain any managed metadata` error.
    * **Conclusion:** The software is a native application (likely C++), not a .NET application.

* **Attempt 2 (Native Decompilation):**
    * **Tool:** Ghidra
    * **Status:** In progress, this is probably the correct tool for the job.

---

## Current Roadblocks

* **Ghidra Environment Setup:** The immediate blocker is a local environment issue with Ghidra, likely related to missing or misconfigured Java JDK environment variables.

---

## Next Steps

1.  **Fix Ghidra Environment:** Troubleshoot the Java installation to get Ghidra running correctly.
2.  **Analyze Executable in Ghidra:**
    * Import `Zelotes F-35C Mouse.exe` and its related DLLs into a Ghidra project.
    * Run the auto-analysis.
    * Search for defined **strings** (e.g., "DPI", "OLED", "HID").
    * Search for **imports** from system libraries related to USB communication (e.g., `hid.dll`).
3.  **(Alternative Path) Active Probing with Python:**
    * Use **Python** with the **`pyusb`** library to write a script that sends commands to the mouse.
    * **Mission 1: Replicate.** Send the known DPI command (`04 20 00 1a 06...`) and confirm that the mouse's DPI changes.
    * **Mission 2: Explore.** Modify the command's "parameter" byte (`04 20 [XX] ...`) by changing `XX` from `0x00` to `0x01`, `0x02`, etc., to probe for hidden functions that might control the OLED screen.

---

## Tools Used

* **Traffic Analysis:** Wireshark with USBPcap
* **Decompilation:** ILSpy (failed), Ghidra (pending)
* **Active Probing:** Python + pyusb (planned)
