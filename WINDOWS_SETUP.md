Using MinGW-w64 (GCC for Windows)

### Step 1: Install MSYS2

1. Download MSYS2 from: https://www.msys2.org/
   - Click the download button for `msys2-x86_64-xxxxxxxx.exe`

2. Run the installer:
   - Install to `C:\msys64` (default)
   - Keep "Run MSYS2 now" checked
   - Click Finish

3. A terminal window opens. Run these commands:

```bash
# Update package database
pacman -Syu
```

4. The window will close. Open **"MSYS2 MINGW64"** from Start Menu (NOT the regular MSYS2!)

5. Install the compiler:

```bash
pacman -S mingw-w64-x86_64-gcc mingw-w64-x86_64-make
```

6. Type `Y` and press Enter when asked

### Step 2: Add to PATH

1. Press `Win + R`, type `sysdm.cpl`, press Enter

2. Click **"Advanced"** tab → **"Environment Variables"**

3. Under "System variables", find **"Path"**, click **"Edit"**

4. Click **"New"** and add:
   ```
   C:\msys64\mingw64\bin
   ```

5. Click OK on all windows

6. **Restart your computer** (important!)

### Step 3: Build the Project

1. Open **Command Prompt** (cmd) or **PowerShell**

2. Navigate to the project:
   ```cmd
   cd C:\path\to\packet_analyzer
   ```

3. Build:
   ```cmd
   g++ -std=c++17 -O2 -I include -o dpi_engine.exe ^
       src/dpi_mt.cpp ^
       src/pcap_reader.cpp ^
       src/packet_parser.cpp ^
       src/sni_extractor.cpp ^
       src/types.cpp
   ```

4. If successful, you'll see `dpi_engine.exe`

### Step 4: Run

```cmd
dpi_engine.exe test_dpi.pcap output.pcap
```

---