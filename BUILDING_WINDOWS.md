# Compiling AIDA-X on Windows (Native)

This guide provides instructions on how to compile the AIDA-X project natively on a Windows machine.

## 1. Prerequisites Installation

You will need the following tools installed on your Windows system:

*   **Chocolatey (Optional but Recommended):** A package manager for Windows that simplifies installing the tools below.
    *   Installation: Go to [https://chocolatey.org/install](https://chocolatey.org/install) and follow the instructions.

*   **Git:** For cloning the repository and its submodules.
    *   Using Chocolatey: `choco install git -y`
    *   Manual: Download from [https://git-scm.com/download/win](https://git-scm.com/download/win) and install. Ensure Git is added to your system PATH.

*   **CMake:** The build system generator.
    *   Using Chocolatey: `choco install cmake --installargs 'ADD_CMAKE_TO_PATH=System' -y`
    *   Manual: Download from [https://cmake.org/download/](https://cmake.org/download/) and install. Ensure CMake is added to your system PATH during installation.

*   **Visual Studio Build Tools (or full Visual Studio IDE):** Provides the C++ compiler (MSVC) and linkers.
    *   Download: Go to [https://visualstudio.microsoft.com/downloads/](https://visualstudio.microsoft.com/downloads/).
    *   Option 1: Download "Build Tools for Visual Studio". During installation, select the "C++ build tools" workload.
    *   Option 2: Install a full version of Visual Studio (e.g., Community Edition). Ensure the "Desktop development with C++" workload is selected during installation.

*   **Python:** Required for a resource compilation script (`res2c.py`) used during the build.
    *   Using Chocolatey: `choco install python -y`
    *   Manual: Download from [https://www.python.org/downloads/windows/](https://www.python.org/downloads/windows/) and install. Ensure Python is added to your system PATH during installation.

*   **VST2 SDK (Required if building VST2 plugin target):**
    *   The VST2 SDK is no longer publicly distributed by Steinberg. You will need to obtain it separately.
    *   Once obtained, place the VST2 SDK folder (e.g., a folder named `VST2_SDK` containing `pluginterfaces/vst2.x/aeffect.h` and `aeffectx.h`) in a known location.
    *   DPF's build system (used by AIDA-X) typically tries to find the VST2 SDK. If it fails, you may need to set an environment variable `VST2_SDK_PATH` pointing to the root of your VST2 SDK, or pass the path to CMake using `-DVST2_SDK_PATH=/path/to/your/vst2_sdk`.

## 2. Clone the Repository

1.  Open a Command Prompt (cmd.exe) or PowerShell.
2.  Navigate to the directory where you want to store the project:
    ```bash
    cd path\to\your\development\folder
    ```
3.  Clone the AIDA-X repository:
    ```bash
    git clone https://github.com/AidaDSP/AIDA-X.git
    ```
4.  Navigate into the cloned project directory:
    ```bash
    cd AIDA-X
    ```
5.  Initialize and update the Git submodules (DPF, etc.):
    ```bash
    git submodule update --init --recursive
    ```

## 3. Configure with CMake

CMake is used to generate the native build files (e.g., Visual Studio solution).

1.  Create a build directory inside the `AIDA-X` folder and navigate into it:
    ```bash
    mkdir build
    cd build
    ```
2.  Run CMake to configure the project and generate build files. The example below is for Visual Studio 2019 (VS16) 64-bit. Adjust the generator (`-G`) and architecture (`-A`) if you are using a different version of Visual Studio or a different compiler like MinGW.
    ```bash
    cmake .. -G "Visual Studio 16 2019" -A x64
    ```
    *   **Visual Studio Generators:**
        *   Visual Studio 17 2022: `-G "Visual Studio 17 2022"`
        *   Visual Studio 16 2019: `-G "Visual Studio 16 2019"`
        *   Visual Studio 15 2017: `-G "Visual Studio 15 2017"`
    *   **Architecture:**
        *   For 64-bit: `-A x64`
        *   For 32-bit: `-A Win32` (though 64-bit is generally recommended)
    *   **MinGW:** If you prefer to use MinGW, you would use a generator like `-G "MinGW Makefiles"` and ensure your MinGW compiler is in the PATH.
    *   **VST2 SDK Path:** If CMake cannot find your VST2 SDK and you need VST2 plugins, add the path to your CMake command:
        ```bash
        cmake .. -G "Visual Studio 16 2019" -A x64 -DVST2_SDK_PATH="C:/path/to/your/VST2_SDK"
        ```

## 4. Compile the Project

After CMake has successfully generated the build files:

*   **Using CMake build command (recommended for command-line):**
    This command works with most generators, including Visual Studio and MinGW Makefiles.
    ```bash
    cmake --build . --config Release
    ```
    *   Replace `Release` with `Debug` if you need a debug build.

*   **Using Visual Studio IDE (alternative):**
    1.  Navigate to the `AIDA-X/build` directory.
    2.  Open the `AIDA-X.sln` file with Visual Studio.
    3.  Select the desired build configuration (e.g., "Release", "x64") from the dropdowns in the toolbar.
    4.  Build the solution (e.g., from the "Build" menu, select "Build Solution", or press `Ctrl+Shift+B`).

## 5. (Optional) Build the Installer

The project includes an Inno Setup script to create a Windows installer.

1.  **Install Inno Setup:**
    *   Download from [https://jrsoftware.org/isinfo.php](https://jrsoftware.org/isinfo.php) and install it. Ensure the Inno Setup compiler (`iscc.exe`) is added to your system PATH or you know its location.

2.  **Prepare files and run Inno Setup Compiler:**
    The `CMakeLists.txt` and the CI setup use a shell script (`utils/windows-installer.sh`) to automate this. To do this manually on Windows after compiling the project:
    *   The script first copies the built plugin files (`.dll`, `.vst3`, `.clap`, `.exe`, `.lv2` folder) from the CMake build output (e.g., `build/bin/Release/`) into a temporary staging directory structure.
    *   Then, it calls the Inno Setup compiler.
    *   A simplified manual approach:
        a.  After compiling the project (Step 4), identify the project version (e.g., from `CMakeLists.txt` or Git tags). Let's say it's `1.1.0`.
        b.  You may need to manually copy the built binaries from `AIDA-X/build/bin/Release/` (or your build output directory) to the locations expected by the Inno Setup script (`utils/inno/win64.iss`), or modify the `.iss` script. The original script stages them into a `tmp-installer` directory.
        c.  Open a Command Prompt or PowerShell, navigate to the `AIDA-X/utils/inno` directory.
        d.  Run the Inno Setup compiler. You'll need to define the `VERSION` variable that the script uses.
            ```bash
            iscc.exe win64.iss /dVERSION="1.1.0"
            ```
            (Replace `"1.1.0"` with the actual project version).
            If `iscc.exe` is not in your PATH, provide the full path to it.
        e.  The installer will typically be created in an `Output` subdirectory within `AIDA-X/utils/inno/`.

## 6. Locate Binaries

After a successful compilation (Step 4), the compiled plugins and the standalone executable can typically be found in:

*   `AIDA-X/build/bin/Release/`
    (If you built a `Debug` configuration, they will be in `AIDA-X/build/bin/Debug/`)

The directory will contain files like:
*   `AIDA-X.exe` (Standalone application)
*   `AIDA-X.vst3` (VST3 plugin)
*   `AIDA-X.dll` (This might be the VST2 plugin if VST2 was built)
*   `AIDA-X.clap` (CLAP plugin)
*   `AIDA-X.lv2/` (LV2 plugin bundle)
