
# Customizing the RustDesk UI - Sciter Version

> This repository and guide help you build a **custom version of RustDesk** with your own **name, icon, and logo**, and connect it to **your own server**.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Cloning and Preparing Source](#cloning-and-preparing-source)
- [Customizing Branding (Logo, Icon, Name)](#customizing-branding-logo-icon-name)
- [Setting Server (hbbs/hbbr)](#setting-server-hbbshbbr)
- [Customizing the RustDesk UI](#customizing-the-rustdesk-ui)
- [Build for Windows](#build-for-windows)
- [Troubleshooting & Tips](#troubleshooting--tips)
- [License & Legal Notes](#license--legal-notes)
- [Credits](#credits)

---

## Prerequisites

To build the latest RustDesk (Sciter GUI version) you need:

- **Visual Studio Installer - MSVC** (select **.NET desktop development** and **Desktop development with C++**)
- **Git** (during installation, select all Environment Variables (PATH) options)
- **Rust** (install latest stable using `rustup-init.exe`) + Cargo
- **vcpkg** (a package manager for C++ to handle libraries and dependencies)
- **Clang/LLVM** (a compiler for C/C++ — [LLVM 15.0.2 (x64)](https://github.com/llvm/llvm-project/releases/download/llvmorg-15.0.2/LLVM-15.0.2-win64.exe))
- **Sciter SDK** (download `sciter.dll` from [sciter-sdk/bin.win/x64](https://raw.githubusercontent.com/c-smile/sciter-sdk/master/bin.win/x64/sciter.dll))
- **Python** (required to compile UI resources into the executable)
- **Visual Studio Code** (recommended for editing source files)

> **Note:**  
> Flutter is not required for the Sciter build.  
> Knowledge of **HTML, JavaScript, CSS, and JSX** is helpful for customizing the Sciter-based UI.

---

## Cloning and Preparing Source

> **Note:**  
> These commands must be run in **Git Bash**, not Command Prompt.

```bash
# 1) Clone vcpkg and install
git clone https://github.com/microsoft/vcpkg
vcpkg/bootstrap-vcpkg.bat
export VCPKG_ROOT=$PWD/vcpkg
vcpkg/vcpkg install libvpx:x64-windows-static libyuv:x64-windows-static opus:x64-windows-static aom:x64-windows-static

# Add System environment variable
VCPKG_ROOT=C:\vcpkg
```

### Install LLVM
- Download LLVM 15.0.2 (x64): [LLVM-15.0.2-win64.exe](https://github.com/llvm/llvm-project/releases/download/llvmorg-15.0.2/LLVM-15.0.2-win64.exe)  
- Install to `C:\LLVM` and set environment variable:
  ```
  LIBCLANG_PATH=C:\LLVM\bin
  ```

### Clone RustDesk
```bash
git clone --recurse-submodules https://github.com/rustdesk/rustdesk
cd rustdesk
mkdir -p target/debug
wget https://raw.githubusercontent.com/c-smile/sciter-sdk/master/bin.win/x64/sciter.dll
mv sciter.dll target/debug
```

---

## Customizing Branding (Logo, Icon, Name)

### 1. Change Application Icons
Replace `.ico`, `.png`, and `.svg` files in the `res/` directory with your own branded icons.

### 2. Change Application Name
Edit `rustdesk/Cargo.toml`:
```toml
[package]
name = "MyCompany"  # your exe name
version = "1.2.0"
authors = ["mycompany <info@mycompany.com>"]
edition = "2021"
build = "build.rs"
description = "A remote control software."
default-run = "mycompany" # must match 'name'

[package.metadata.bundle]
name = "My Company" # Display name
identifier = "com.mycompany.remote"
icon = ["res/32x32.png", "res/128x128.png", "res/128x128@2x.png"]
```

### 3. Change APP_NAME in `config.rs`
In `/rustdesk/libs/hbb_common/src/config.rs` (around line 50):
```rust
pub static ref APP_NAME: Arc<RwLock<String>> = Arc::new(RwLock::new("My Company".to_owned()));
```

### 4. Changing the application icons in `ui.rs`
In `/rustdesk/src/ur.rs` (last line):
```rust
pub fn get_icon() -> String {
    // 128x128
    #[cfg(target_os = "macos")]
    // 128x128 on 160x160 canvas, then shrink to 128, mac looks better with padding
    {
        "data:image/png;base64,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".into()
    }
    #[cfg(not(target_os = "macos"))] // 128x128 no padding Windows
    {
        "data:image/png;base64,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".into()
    }
}
```
[Convert your image to Base64](https://www.base64-image.de/) and then replace the contents of ICON with your base64 encoded image (it should start with data:image/png;base64, followed by your image data). Your image should be 128x128

### 5. How to Embed Sciter.dll into the Executable
- Copy sciter.dll to your project root directory (where Cargo.toml resides)
- In the file below, find the fn main() function and replace it with the following code:
In `/rustdesk/src/main.rs`
```rust
#[cfg(not(any(
    target_os = "android",
    target_os = "ios",
    feature = "cli",
    feature = "flutter"
)))]
fn main() {

     //BEGIN CHANGES
    //Embed the Sciter.dll file into the exe, and then write it to disk when application starts
    println!("================ LOADING SCITER DLLL ==================");
    let bytes = std::include_bytes!("..\\sciter.dll"); //since main.rs is in rustdesk/src, we need to go up one level (to rustdesk)
    std::fs::write("sciter.dll", bytes.as_slice());
    //END CHANGES
    
    if !common::global_init() {
        return;
    }
    #[cfg(all(windows, not(feature = "inline")))]
    unsafe {
        winapi::um::shellscalingapi::SetProcessDpiAwareness(2);
    }
    if let Some(args) = crate::core_main::core_main().as_mut() {
        ui::start(args);
    }
    common::global_clean();
}
```

```rust
> Note: Make sure to select the correct platform. This line indicates that the target operating system is Windows.
#[cfg(not(any(
    target_os = "android",
    target_os = "ios",
    feature = "cli",
    feature = "flutter"
)))]

```

### 6. Enable Inline Build (Embed UI)
Edit `Cargo.toml`:
 
  `default = ["use_dasp"]`  change to `default = ["use_dasp", "inline"]`

Run:
```cmd
python res\inline-sciter.py
```
> Note: This command must be run in CMD.

---

## Setting Server (hbbs/hbbr)

To lock RustDesk to your own server:

### 1. Run OSS or Pro Server
- Start `hbbs` and `hbbr` services (via Docker or systemd).
- Open required ports:
  - TCP: `21115-21117`
  - UDP: `21116`
- Follow official docs: [RustDesk Self-Hosting](https://rustdesk.com/docs/en/self-host/rustdesk-server-oss/install/)
- Set up the server according to the official RustDesk instructions, and obtain your `IP` and `encryption key`.

You will also need to specify the public key you generated on your server

### 2. Hardcode Server Address
Edit `/rustdesk/libs/hbb_common/src/config.rs`:
```rust
pub const RENDEZVOUS_SERVERS: &'static [&'static str] = &["myserver.com"]; // or IP address
```

Also update `/rustdesk/src/common.rs` (around line 517):
```rust
let rendezvous_server = socket_client::get_target_addr(&format!("myserver.com:{}", config::RENDEZVOUS_PORT))?;
```

### 3. Hardcode Public Key
Assign your server's `id_ed25519.pub` key to `RS_PUB_KEY`:
```rust
pub const RS_PUB_KEY: &'static str = "your_public_key_here";
```

---

## Customizing the RustDesk UI
- Main UI files:
  ```
  rustdesk/src/ui/index.tis
  rustdesk/src/ui/index.css
  ```
- Modify `index.tis` (JSX-like structure) and `index.css` for styles.
- At line 566 `index.tis`, you'll find the `function render()` related to the UI.
  You can make changes here according to your custom design, but keep in mind that programming knowledge (as mentioned at the beginning of this document) is required.

  - The file `index.css` is used for styling and design. You can apply your personal customizations along with the main project files.
- Run `python res/inline-sciter.py` after each change.

---

## Build for Windows
Embedding UI / Enable Inline Builds
```cmd
python res\inline-sciter.py
```
> Note: This command must be run in CMD.

Build and run application
```bash
cargo build --release
```

- Output: `rustdesk\target\release`
- Copy `sciter.dll` into same folder as `.exe` before running on other systems.

---

## Troubleshooting & Tips

- Clear cache after replacing icons:  
  ```
  cargo clean
  ```
- Restart system to refresh icon cache.
- Ensure Sciter runtime matches build version.

---

## License & Legal Notes

- RustDesk is under **AGPL-3.0**.  
- If distributing binaries, publish your changes and credit RustDesk authors.  
- For commercial use, consult RustDesk Pro license.

---

## Credits

Thanks to the RustDesk team for their outstanding open-source project. If this guide helped you, give the original repository a star!

---

## 🚀 About Me

[![portfolio](https://img.shields.io/badge/my_portfolio-000?style=for-the-badge&logo=ko-fi&logoColor=white)](https://github.com/mojtabco) [![linkedin](https://img.shields.io/badge/linkedin-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/mojtaba-askari-b0522b378/)
