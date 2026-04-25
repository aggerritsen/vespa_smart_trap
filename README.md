# GV2 YOLO11 Model Flashing Instructions

Quick reference for flashing YOLO11 models to Grove Vision AI V2 on macOS and Windows 11.

## Repository Setup

- **Clone repo with submodules (all-in-one):** `git clone --recurse-submodules https://github.com/marcory-hub/vespa_smart_trap && cd vespa_smart_trap`
- **Or separately:** `git clone https://github.com/marcory-hub/vespa_smart_trap && cd vespa_smart_trap && git submodule update --init --recursive`
- **If Git reports a safe.directory warning on Windows:** `git config --global --add safe.directory C:/path/to/vespa_smart_trap`

### Python Setup on macOS

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install pyserial
```

### Python Setup on Windows 11 (PowerShell)

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install pyserial
```

If `pip` is not available in the active environment, use `python -m pip install pyserial`.

## Available Models

- `yolo11n_vespa_2026-02v1_30pxNULL_full_integer_quant_vela.tflite`
- `yolo11n_vespa_2026-02v1_40pxNULL_full_integer_quant_vela.tflite`
- `yolo11n_vespa_2026-02v1_60pxNULL_full_integer_quant_vela.tflite`
- `yolo11n_vespa_2026-02v1_allpxNULL_full_integer_quant_vela.tflite`

All models are expected in `gv2_firmware/model_zoo/tflm_yolo11_od/`.

## Prerequisites

- USB-C cable connected to the GV2 board
- Python 3.11+ with the virtual environment activated
- `gv2_firmware` submodule initialized
- Firmware image already built at `gv2_firmware/we2_image_gen_local/output_case1_sec_wlcsp/output.img`

If `output.img` is missing, build the firmware first.

### Build Firmware Image

On macOS or Linux:

```bash
cd gv2_firmware/EPII_CM55M_APP_S
make APP_TYPE=tflm_yolo11_od
```

On Windows 11:

- Recommended: build from WSL or another Unix-like shell, because the firmware build uses `make`
- If your Windows environment already has a working `make` toolchain, run the same command as above

## Flashing on macOS

1. Identify the USB port with `ls /dev/cu.usbmodem*`
2. Change into the firmware folder: `cd gv2_firmware`
3. Run:

```bash
python xmodem/xmodem_send.py \
  --port=PORT \
  --baudrate=921600 \
  --protocol=xmodem \
  --file=we2_image_gen_local/output_case1_sec_wlcsp/output.img \
  --model="model_zoo/tflm_yolo11_od/MODEL_NAME.tflite 0xB7B000 0x00000"
```

Replace `PORT` and `MODEL_NAME.tflite` with your values.

Example:

```bash
python xmodem/xmodem_send.py \
  --port=/dev/cu.usbmodem58FA1047631 \
  --baudrate=921600 \
  --protocol=xmodem \
  --file=we2_image_gen_local/output_case1_sec_wlcsp/output.img \
  --model="model_zoo/tflm_yolo11_od/yolo11n_vespa_2026-02v1_allpxNULL_full_integer_quant_vela.tflite 0xB7B000 0x00000"
```

## Flashing on Windows 11 (PowerShell)

1. Identify the COM port:

```powershell
Get-CimInstance Win32_SerialPort | Select-Object DeviceID, Name
```

2. Change into the firmware folder:

```powershell
cd .\gv2_firmware
```

3. Run the flasher:

```powershell
python .\xmodem\xmodem_send.py `
  --port=COM3 `
  --baudrate=921600 `
  --protocol=xmodem `
  --file=we2_image_gen_local/output_case1_sec_wlcsp/output.img `
  --model="model_zoo/tflm_yolo11_od/MODEL_NAME.tflite 0xB7B000 0x00000"
```

If `python` does not resolve correctly, use `py` instead.

Example:

```powershell
python .\xmodem\xmodem_send.py `
  --port=COM3 `
  --baudrate=921600 `
  --protocol=xmodem `
  --file=we2_image_gen_local/output_case1_sec_wlcsp/output.img `
  --model="model_zoo/tflm_yolo11_od/yolo11n_vespa_2026-02v1_allpxNULL_full_integer_quant_vela.tflite 0xB7B000 0x00000"
```

## During Flashing

1. Start the command
2. When prompted by the GV2 tool, press the board reset button
3. Wait for the X-Modem transfer to reach 100%
4. Let the board reboot

## Post-Flash Validation

This repo already includes `Himax_AI_web_toolkit`, so you can use the local copy instead of downloading it again.

- Open `Himax_AI_web_toolkit/index.html` in a browser
- Select `Grove Vision`
- Click `Connect`

For serial-based validation, the repo also includes diagnostic scripts under `scripts/gv2_swift_yolo_test/`. On Windows, pass the serial port explicitly, for example `--serial-port COM3`.
