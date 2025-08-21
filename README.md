
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

### 4. Enable Inline Build (Embed UI)
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
