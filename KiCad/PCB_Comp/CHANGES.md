# PCB_Comp: review and fix log (14 Nov 2026)

## Results

| Check | Before | After |
|---|---|---|
| ERC (all severities, kicad-cli 10.0.6) | 175 messages (15 errors, 160 warnings) | **0 errors, 0 warnings** |
| Symbols with a footprint | 76 / 183 | **183 / 183** |
| Symbol pin numbers matching footprint pad names | n/a | **183 / 183 match** |
| Footprints that resolve in the libraries | 10 missing | **all resolve** |
| Unconnected IC pins compared with the LILYGO V1.2 NC list | n/a | identical |
| DRC | 1 (no board outline) | 1 (no board outline): the .kicad_pcb is still empty. Clears once Edge.Cuts is drawn |

The original ERC breakdown was: 135 pin_to_pin, 11 power_pin_not_driven, 11 lib_symbol_issues, 10 footprint_link_issues, 3 unconnected_wire_endpoint, 2 pin_not_connected, 2 multiple_net_names, 1 no_connect_dangling.

## Real circuit bugs fixed

1. **Power_USB:** the buck output symbol #PWR070 was `power:+3V3` / "+3V3". Every load uses `+3.3V`, so the 3.3 V rail was a separate, unconnected net. Changed it to `power:+3.3V`.
2. **Power_USB:** #PWR0109 and #PWR0119 were valued "+3V7_BAT", while the rest of the battery net used "+3.7V_BAT". This split the net in two. Both are now "+3.7V_BAT".
3. **Audio:** U10 (MAX98357A) VDD was not connected to any supply. Added a wire and a +3.3V symbol at C53/C54.
4. **Audio:** C59/C60 ground was labelled AGND_MAX98357 (the amplifier ground), but they decouple the mic. Relabelled to AGND_MP34.
5. **ESP32_Core:** X1 pin 1 was not wired. Added a 1.27 mm wire.
6. **ESP32_Core:** the C3 (NFM18) GND pin was not on the GND wire. Rerouted and moved #PWR066 up 1.27 mm.
7. **Memory_Storage:** the SD card VSS (pin 6) and C65/C66 had no ground. Added a wire, a junction and a GND symbol.
8. **Memory_Storage:** SD card pin 10 had a no-connect flag but must be GND. Removed the flag and wired it to GND. Also removed a stray no-connect flag at (54.61, 127).
9. **Power_USB:** C23 was 1u. The LILYGO reference uses 47n, so it is now 47n.

## Designators matched to LILYGO
- Display: R44 and R45 swapped, so R45 = SCREEN_CS and R44 = TOUCH_INT pull-ups.
- Audio: L9 and L10 swapped (OUTP/OUTN ferrites).
- Memory: Card1 renamed CARD1.
- Power_USB: power-symbol references #PWR091 to #PWR0107 and #PWR070 renumbered to #PWR0203 to #PWR0220, because they clashed with Camera.

## ERC housekeeping (no circuit change)
- **PWR_FLAGs added** where a rail has no power_out driver:
  - ESP32: VDDA and VDD3P3.
  - Audio: AGND_MAX98357, AGND_MP34 and the mic Vdd.
  - Camera: the U7 and U8 input nets.
  - Power_USB: +5V, +3.3V, +3.7V_BAT, GND and the U5 IN net.
- **Removed 2 PWR_FLAGs** in Camera, on +1.5V and +A2.8V. They clashed with the LDO outputs, which already drive those rails.
- **ESP32:** two local labels "VDD_SPI_l" became global labels "VDD_SPI", and two "VDD3P3_RTC_l" became "+3.3V". This fixes the multiple-net-name errors and gives the nets the same names as LILYGO.
- **Library pin types:**
  - Unspecified pins on LCSC symbols were set to passive, which removed the 135 pin_to_pin warnings.
  - Real supply inputs were set to power_in: SY6970 pins 1, 17, 18 and 25; ETA3409 pins 2 and 4; RT9080 pins 1 and 2; AP1511B pins 2 and 5; MP34DT05 pins 1 and 5.
  - The embedded MAX98357A pin 17 (PAD) was set to passive.
- **Symbol libraries:**
  - Added 9 symbols that were placed but missing from copperclash_lcsc.kicad_sym: 12-22_R6G6C-C30_2C, A1257WV-S-2P, ETA3409S2F, FTC252012S2R2MBCA, GT-USB-7010C, MK-12C01-G1.5, RT9080-33GJ5, SDCW2012-2-900TF, SY6970QCC.
  - Added the LCD_240X240_CST816 library to sym-lib-table.
- **Audio / Power_USB:** added the missing `power:PWR_FLAG` and `power:+3.3V` definitions to the embedded symbol libraries.

## Values (unit suffix removed, part numbers filled)
- **Capacitors:**
  - "pF/nF/uF" became "p/n/u" on C4, C7, C10, C20–C39, C53–C62, C65 and C66.
  - Example: 24pF became 24p and 10uF became 10u.
- **L4:** 24nH became 24n.
- **Connectors:**
  - CN1/CN2: HDGC1002WR-S-4P
  - CN4: HDGC1251WV-2P
  - CN5: HC-1.25-2PLT
- **Semiconductors:**
  - D4–D7: ESD05V88D-LC
  - Q1/Q2: S8050
  - Q3: CJ2333
  - Q5: SI2300A
- **ICs and display:**
  - U2: W25Q128JVSIQ
  - U3: APS6404L-3SQR-SN
  - U10: MAX98357AETE+T
  - U11: fp-133h0M1d (was "~")
  - U12: AP1511B

## Footprints
- **Assigned (107 symbols had none):**
  - C_0402: 31 capacitors.
  - C_0603: 20 capacitors (≥4.7u, plus C7 and C12).
  - C_0805: C27 and C29 (22u).
  - L_0402: L1–L4.
  - R_0402: 38 resistors, plus the 0 Ω links L8, L12, L13 and L14.
  - R_0603: R34.
  - R_0805: L6.
  - D_SOD-323: D3.
  - SOT-23: Q3.
  - CN1/CN2: JST SH SM04B-SRSS-TB (1.0 mm, 4P + MP).
  - CN5: Molex PicoBlade 53261-0271 (1.25 mm, 2P + MP).
  - U11: copperclash_lcsc:LCD_FPC-16P_P0.5mm_FH12-16S.
- **Changed:** R43 and R51 from 0402 to 0603, to match the LILYGO 3D model.
- **Built the 10 missing copperclash_lcsc footprints** from KiCad 10 stock footprints, with pads renamed to match the symbol pins:

  | Footprint | Built from |
  |---|---|
  | CONN-SMD_A1257WV-S-2P | PicoBlade 2P |
  | FILTER-SMD_4P-L2.0-W1.3-BL_DLW21 | 2012 common-mode choke |
  | IND-SMD_L2.5-W2.0_MHCHL2520 | 1008 |
  | LED-SMD_R6G6C-C30 | 3010 RGB |
  | QFN-24_L4.0-W4.0-P0.50-TL-EP2.7 | — |
  | SOT-23-5_L2.9-W1.6-P0.95-LS2.8-BL | — |
  | TSOT-23-5_L2.9-W1.6-P0.95-LS2.8-BL | — |
  | SW-SMD_DEALON_MK-12C01-G1.5 | — |
  | USB-C-SMD_GT-USB-7010C | 16P USB-C |
  | LCD_FPC-16P_P0.5mm_FH12-16S | — |

- **Existing easyeda footprints:** the 3D model path now points to `${KIPRJMOD}/lib/copperclash_lcsc.3dshapes/*.step`.

## Root sheet
- All 6 sheet boxes are now 63.5 × 40.64 mm on a 3 × 2 grid.
- Each box has one short line of text inside it.

## Files removed
- `python` (empty file)
- `~PCB_Comp.kicad_pro.lck`
- `~PCB_Comp.kicad_sch.lck`

## Please confirm on your PC
- **LED1:** the RGB pinout (centre pad = common anode) and the outline. If you can, replace it with the exact one: `python -m easyeda2kicad --full --overwrite --lcsc_id=C970147`.
- **SW4:** if you can, replace it the same way with `--lcsc_id=C2904924`.
- **U11:** the display FPC was assumed to be 16 pins at 0.5 mm pitch. Check against the LCD datasheet before layout.
- **USB-C (J1):** the GT-USB-7010C pad geometry is taken from a family-equivalent footprint. Check the pad size against the datasheet.

## Noted, not changed
- U9 CMC: D- is on pins 1–2 and D+ on 3–4, the reverse of LILYGO. This is electrically equivalent.
- SW3 uses the K2 switch rather than LILYGO's 6×6 switch.
- KiCad's default-ignored checks (single global label, four-way junction, footprint filter) are left at the defaults.
