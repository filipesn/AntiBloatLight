# AntiBloatLight [![Latest Release](https://img.shields.io/github/v/release/ScepticDope/AntiBloatLight?style=flat-square)](https://github.com/ScepticDope/AntiBloatLight/releases) [![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](https://github.com/ScepticDope/AntiBloatLight/blob/main/LICENSE) [![Total Downloads](https://img.shields.io/github/downloads/ScepticDope/AntiBloatLight/total?label=total%20downloads&style=flat-square)](https://github.com/ScepticDope/AntiBloatLight/releases)
This Rust application interacts with a SteelSeries keyboard via the HID protocol, allowing you to send data to the device. Removing the need for `SteelSeries GG` to run if you only want to set colours.

You can also use something like: [SharpKeys](https://github.com/randyrants/sharpkeys), to replace `SteelSeries GG` remap functionality if you would miss that.

## Feature
- Send a raw data packet to a connected `Steelseries Apex 3 TKL (Qwerty US)` setting the color on all LED lights to rgb(157, 0, 255) (`0x9d, 0x0, 0xff`).

## Prerequisites
_You have two options to use this repository:_

**Public Fork and Release**:
   - Make a public fork of this repository.
   - Modify the code as needed, refer to the [Usage](#usage) section for guidance.
   - Create a release, the GitHub Actions workflow [`.github/workflows/rust.yml`](https://github.com/ScepticDope/AntiBloatLight/blob/main/.github/workflows/rust.yml) will attempt to build the project and attach the `.exe` file to the release.
   - Wait for it to finish, then download the `.exe` file from the release and try running it.

**Local Build**:  
Ensure the following prerequisites are installed before building:

1. **Supported Operating Systems**  
   Tested on Windows 10 and 11. <sub><sup>_Credit to [JayRom95_fr](https://www.reddit.com/r/steelseries/comments/1gubzvp/comment/ly7yo92/) for testing Windows 11._</sup></sub>

2. **Rust Programming Language**  
   Ensure you have the Rust toolchain installed. If you don't have it, install Rust using [rustup](https://rustup.rs/).

3. **Visual Studio Build Tools 2022**  
   AntiBloatLight requires the Visual Studio C++ Build Tools to be installed on your system to compile and link the required dependencies.  
   Download and install Visual Studio Build Tools from the following link:  
   [Visual Studio Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/)

   Make sure to select: `Desktop development with C++`.
   
   _Here is a screencap with the only required components:_
   ![Rust Config - Visual Studio Build Tools 2022](https://github.com/ScepticDope/AntiBloatLight/blob/main/Rust%20Config%20-%20Visual%20Studio%20Build%20Tools%202022.PNG?raw=true)

## Installation
Clone the repository and build the project using Cargo:
```
git clone https://github.com/ScepticDope/AntiBloatLight.git
cd AntiBloatLight
cargo build --release
```

## Dependencies
The project uses the following dependencies:

    hidapi - A Rust crate for cross-platform HID device communication.

You can find the latest version of the hidapi crate on [crates.io](https://crates.io/crates/hidapi).
Cargo.toml:
```
[package]
name = "antibloatlight"
version = "1.0.3"
edition = "2021"

[dependencies]
hidapi = "2.6.3" # Check for the latest version on https://crates.io/crates/hidapi
```

## Usage
After building the project, you can run the application by running `antibloatlight.exe`.

The application will search for a connected HID device with the specified Vendor ID, Product ID, and Interface Number. Once found, it will send a raw data packet to the device.

<details>
<summary><strong>File: main.rs</strong></summary>

```rust
// Raw data that needs to be sent to the device, starting with an extra 0x0 to ensure proper alignment.
// Main colour rgb(157, 0, 255) = 0x9d, 0x0, 0xff
// Alternative colour rgb(247, 75, 0) = 0xF7, 0x4B, 0x00
let raw_data_to_send = [
    0x0, 0x21, 0xff, 0x9d, 0x0, 0xff, 0x9d, 0x0, 0xff, 0x9d, 0x0, 0xff, 0x9d, 0x0, 0xff, 0x9d,
    0x0, 0xff, 0x9d, 0x0, 0xff, 0x9d, 0x0, 0xff, 0x9d, 0x0, 0xff, 0x9d, 0x0, 0xff, 0x0, 0x0, 0x0,
    0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0,
    0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0,
];
```
</details>

Just change the code to support your specific Steelseries keyboard and change the color. Personally I used: [WireShark](https://www.wireshark.org/download.html), to sniff out the packet containing the info I needed.
You can also use Windows' **Devices and Printers** to find this information. Simply follow these steps:

1. Open **Devices and Printers** in the Control Panel.  
2. Right-click on your keyboard and select **Properties**.  
3. Go to the **Hardware** tab and select the HID keyboard entries.  
4. Check the **Hardware Ids** under the **Details** tab.  

For example, on my system, the Hardware Ids are:
```
HID\VID_1038&PID_1622&REV_0300&MI_00
HID\VID_1038&PID_1622&MI_00
HID\VID_1038&UP:0001_U:0006
HID_DEVICE_SYSTEM_KEYBOARD
HID_DEVICE_UP:0001_U:0006
HID_DEVICE
```

**VID** stands for **Vendor ID** and **PID** for **Product ID**, while `UP:0001_U:0006` likely represents the range of interfaces. If you're unsure, try all possible combinations until it works. 
Below is the code you need to modify. If you have multiple HID entries (e.g., my keyboard had two), test each option one by one until you find the correct configuration.

<details>
<summary><strong>File: main.rs</strong></summary>

```rust
.find(|device| {
   device.vendor_id() == 0x1038
       && device.product_id() == 0x1622
       && device.interface_number() == 0x01
})
```
</details>

You can place `antibloatlight.exe`, or a shortcut to it, in your startup folder so it runs once automatically after startup.  
To easily access this folder, press `Win + R`, type `shell:startup`, and press Enter.

## License
This project is licensed under the MIT License - see the [LICENSE](https://github.com/ScepticDope/AntiBloatLight/blob/main/LICENSE) file for details.

## Contributing
Feel free to open issues or submit pull requests for improvements or bug fixes.

## Acknowledgments
- The [Rust Programming Language](https://www.rust-lang.org/) for providing a safe, fast, and efficient way to build system applications like AntiBloatLight.
- The [HIDAPI](https://crates.io/crates/hidapi) crate for HID device communication, the only lib I found that plays well with Windows.
