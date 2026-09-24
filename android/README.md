# Android App Skeleton

This directory will contain the Kotlin Android project.

## Planned modules

- `app` — UI, BiometricPrompt, service lifecycle
- `ctap` — CTAP2 command handlers, CBOR, CTAPHID framing
- `transport-usb` — `/dev/hidg*` reader/writer (root)
- `transport-ble` — BluetoothHidDevice implementation
- `crypto` — Keystore-backed key generation and signing

## Build (future)

```bash
./gradlew :app:assembleDebug
```

Minimum SDK: 28 (BLE HID). Target: latest stable.

Root/USB features will be optional and gated behind a build flavor or runtime check.
