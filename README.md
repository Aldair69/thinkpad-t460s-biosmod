# ThinkPad T460 - Unlock Advanced BIOS Menu, Whitelist, and CFG Lock

**Translated from Chinese. Original Author:** [**Comf0rTS**](https://www.zhihu.com/people/li-bai-bi-77) 📖 [Original Zhihu article (archived)](https://web.archive.org/web/20230628105920/https://zhuanlan.zhihu.com/p/343123410)
---

## ⚠️ Disclaimer

1. Flashing BIOS has risks — proceed with caution.
2. I take no responsibility if something goes wrong.
3. I am not a professional. I may not be able to help.
4. Tested on: BIOS version ???, M2xL3 Motherboard Rev???

There seems to be no complete BIOS modification guide for the ThinkPad T460s, so here is one.

> ThinkPad models from the xx30 generation onward cannot be unlocked via software. A hardware flash is required.

**TL;DR:**

> plug CH341A programmer → Connect to BIOS chip → Read and backup → Apply patch → Flash modified BIOS back

---

## 🧰 Required Tools

### Hardware:

- Another computer (not the ThinkPad)
- CH341A programmer + SOIC8 test clip
- Drivers for CH341A
- Optional: USB extension cable

### Software:

- [UEFITool](https://github.com/LongSoft/UEFITool/releases/tag/0.21.5) (Use numeric version for patch support)
- [AsProgrammer](https://github.com/nofeletru/UsbAsp-flash/releases)
- Hex editor (e.g., HxD, i'm going to use hexed.it)

---

## 📋 Preparation

1. Know your BIOS version (from BIOS setup screen)
2. Know your motherboard model and revision (under tape near the CPU)
3. (Optional) Find factory BIOS online
4. Download patch script — Archived: [Index of backup\_thinkpads](https://web.archive.org/web/20210429085237/http://paranoid.anal-slavery.com/files/backup_thinkpads/)

**Patch in case archive.org explodes or something:**

```
# Patch string format
# FileGuid SectionType PatchType:FindPatternOrOffset:ReplacePattern 
# Please ensure that the latest symbol in patch string is space

# Possible section types:
#  PE32 image                    10
#  Position-independent code     11
#  TE Image                      12
#  DXE Dependency                13
#  Version information           14
#  User interface string         15
#  16-bit code                   16
#  Guided freeform               18
#  Raw data                      19
#  PEI Dependency                1B
#  SMM Dependency                1C
# Please do not try another section types, it can make the resulting image broken

# Possible patch types:
#  P - pattern-based, first parameter is a pattern to find, second - a pattern to replace
#  O - offset-based, first parameter is hexadecimal offset, second - a pattern to replace
# Patterns can have . as "any possible value" symbol

# works with paranoidbashthot's bypass method, 's/\x4C\x4E\x56\x42\x42\x53\x45\x43\xFB\xFF/\x4C\x4E\x56\x42\x42\x53\x45\x43\xFF\xFF/g'
# all patches by \x unless stated, no warranties.

# SystemFormBrowserCoreDxe | enable advance menu Lenovo xx60/xx70/xx80 
721C8B66-426C-4E86-8E99-3457C46AB0B9 10 P:04320b483cc2e14abb16a73fadda475f:778b1d826d24964e8e103467d56ab1ba  
32442D09-1D11-4E27-8AAB-90FE6ACB0489 10 P:04320b483cc2e14abb16a73fadda475f:778b1d826d24964e8e103467d56ab1ba  


#
# LenovoWmaPolicyDxe | Lenovo Thinkpad xx60 | remove wwan whitelist
79E0EDD7-9D1D-4F41-AE1A-F896169E5216 10 P:C8390B747A:C8390B7400  
79E0EDD7-9D1D-4F41-AE1A-F896169E5216 10 P:44394B047425:44394B04EB25  
79E0EDD7-9D1D-4F41-AE1A-F896169E5216 10 P:0F84DA000000:E9DB00000000  
```

---

## 🧪 Locating and Connecting to BIOS Chip

1. Locate chip on motherboard (label: U49 — small chip between RAM and mini pci)
2. Identify chip model (e.g., Winbond) <img width="1366" height="768" alt="imagen" src="https://github.com/user-attachments/assets/750769db-bfab-47e3-8afc-274ab3f1c94d" />

3. Assemble CH341A:
   - Bridge pins 1-2 to enter programming mode ![v2-83c227a52311dcc44a89858968e49c8f_1440w](https://github.com/user-attachments/assets/9fef7671-3823-4652-8adf-b9468c58ab3b)

   - Align adapter: pin 1 top-left (consult your CH341A) ![v2-fc144adc2d93529fb5778c422eab82ee_1440w](https://github.com/user-attachments/assets/23b7cf55-11cf-4b3f-b2c5-2ee8e4a03686)

   - Attach SOIC8 clip — red wire = pin 1 ![v2-3947f7b536f421e4b73a03f8077906df_1440w](https://github.com/user-attachments/assets/cf23920c-a4b7-497d-b809-1091e000a619)

4. Plug into USB — ensure device is seen as "Interface", not COM
5. Unplug all batteries (CMOS and internal)
6. Connect clip firmly to BIOS chip (pay attention to pin orientation!) tre little dot says where the red cable should be
7. Reconnect CH341A (extension cable recommended)

---

## 🧩 Flashing Process

1. Install the driver <img width="1366" height="768" alt="imagen" src="https://github.com/user-attachments/assets/a1347283-95db-4963-abab-79fe536e89d8" />
2. Launch AsProgrammer
3. Select chip model via `IC > Search`
4. Click "Read IC"
   - If output is all `FF` or red log text appears, re-check connections
5. Save the dump — backup in a safe place!
6. Do 4 oh more dumps just to be sure
7. Save it again as `1.BIN` in the UEFIPatch folder
8. If u are using 1.bin from this repo make sure your md5 cheksum is 
9. Place your patch file as `patches.txt`
10. It should look like this <img width="1366" height="768" alt="imagen" src="https://github.com/user-attachments/assets/45572bac-f06d-4cea-818d-86787bf50b09" />

11. Open Powershell in the same folder
12. Run:

```sh
./UEFIPatch.exe 1.BIN
```

9. A file `1.BIN.patched` will be generated
10. Open it in a hex editor, search `4C 4E 56 42 42 53 45 43 FB`
    - Change `FB` to `FF`, save
      <img width="1366" height="768" alt="imagen" src="https://github.com/user-attachments/assets/de526907-049f-4e76-8b24-b27b42d5dba9" />

11. Open `1.BIN.patched` in AsProgrammer
12. Perform:

- Erase IC
- Program IC
- Verify IC (Use the buttons in sequence)
  <img width="1366" height="768" alt="imagen" src="https://github.com/user-attachments/assets/dfc95cd5-d738-4a15-abf5-2e2c1040e96f" />


---

## ✅ Done!

Reboot your ThinkPad. BIOS may reboot a few times and beep (e.g., 5+5). That’s normal. You should now have the Advanced Menu available — including CFG Lock options.

> Be gentle when removing the SOIC clip — don’t yank it!

Reassemble the laptop and reconnect all batteries.

---

## 🔁 Recovery (if laptop doesn't boot)

Open AsProgrammer again, load the original backup dump, and flash it back.

---

- This translation was adapted for educational use
