# The official Vellum package manager repository

A custom built open-source package manager for PocketBook e-readers.

## What is it?
This repository hosts the metadata (`repo.json`) and application binaries (`.zip`) for the Vellum store. It serves as the central hub for installing custom made apps for PBs.

## How to use
1. ~~**Install Vellum:** Download `Vellum.zip` from the [Releases](https://github.com/aurelian-th/vellum-repo/releases) page.~~ (to be added)
2. **Install on device:** Extract the contents to your PocketBook's `/applications/` folder.
3. **Browse:** Open Vellum on your device to browse and ~~install~~ apps like **SyncBook**.

## Available apps
| App Name | Version | Description |
| :--- | :--- | :--- |
| **SyncBook** | v0.1.0-alpha | Cloud synchronization client (Google Drive) using `rclone`. |
| **Vellum** | v0.1.0-alpha | The store client itself. |

## For developers
Want to build apps for Vellum? Check out the [developer guide](DEVELOPER.md) to learn about:
* The **sidecar architecture** (C + Go/Rust/Python).
* The **dockerized build system** (Windows/Linux/Mac).
* How to publish your app to your repository and add it to the repos in Vellum.

---
*Maintained by [0xAu](https://github.com/aurelian-th). Special thankx to [jjrrw174](https://github.com/jjrrw174/PocketBook-Desktop-and-App-Customizations) for the research on app customisation and [KOReader team](https://github.com/koreader).*
