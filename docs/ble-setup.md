# BLE HID Setup (No Root)

## Requirements

- Android 9.0 (API 28) or higher
- Bluetooth enabled on both phone and host
- Host OS that accepts Bluetooth FIDO authenticators (Windows 10/11 is the most reliable)

## How it works

The app registers as a Bluetooth HID Device and advertises a report map matching the FIDO CTAP HID usage page. Once paired, the host talks CTAPHID over BLE the same way it would over USB.

## Pairing flow (planned)

1. Open PhoneFIDO-Key → “Enable BLE Security Key”.
2. Phone becomes discoverable.
3. On the computer, add a Bluetooth device; select the phone.
4. App confirms pairing and starts the CTAP service.
5. Use any WebAuthn site or OS login that accepts security keys.

## Limitations

- Pairing must be done carefully; some hosts cache the HID descriptors poorly.
- macOS and Linux support for BLE FIDO is less consistent than Windows.
- Range and interference are typical Bluetooth issues.
- Android OEM Bluetooth stacks vary; testing across Pixel, Samsung, etc. is required.
