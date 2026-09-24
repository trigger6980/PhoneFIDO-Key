# PhoneFIDO-Key

**Turn your Android phone into a FIDO2 / WebAuthn roaming security key** (USB + BLE), similar to a hardware key like YubiKey, SoloKey, or Titan.

> Status: Early design + skeleton. USB path requires a rooted Android device with ConfigFS support. BLE path targets Android 9+ without root.

## Why this project?

Hardware FIDO keys are excellent but easy to lose. Your phone is almost always with you. This project aims to make the phone act as a **roaming authenticator** that computers see as a standard CTAP2 security key over:

- **USB** (via Android USB Gadget / ConfigFS + CTAPHID) — requires root
- **Bluetooth HID** (Android 9+) — no root required (primary practical path today)

It implements the **Client to Authenticator Protocol (CTAP 2.x)** so browsers and OS login prompts treat the phone exactly like a physical key.

## High-level architecture

```
┌─────────────────┐     USB OTG / BLE HID     ┌──────────────────────┐
│  Computer       │ ◄───────────────────────► │  Android Phone       │
│  (Browser/OS)   │   CTAPHID reports         │                      │
│                 │                           │  ┌────────────────┐  │
│  WebAuthn /     │                           │  │ USB Gadget     │  │
│  Windows Hello  │                           │  │ (ConfigFS)     │  │
│  / libfido2     │                           │  │ or BLE HID     │  │
└─────────────────┘                           │  └────────┬───────┘  │
                                              │           │          │
                                              │  ┌────────▼───────┐  │
                                              │  │ CTAP2 Engine   │  │
                                              │  │ (this app)     │  │
                                              │  └────────┬───────┘  │
                                              │           │          │
                                              │  ┌────────▼───────┐  │
                                              │  │ Android        │  │
                                              │  │ Keystore +     │  │
                                              │  │ Biometrics     │  │
                                              │  └────────────────┘  │
                                              └──────────────────────┘
```

### Components

1. **Transport layer**
   - USB: Activate FIDO CTAP HID function via ConfigFS (see [tejado/android-usb-gadget](https://github.com/tejado/android-usb-gadget)). App talks to `/dev/hidg0`.
   - BLE: Android Bluetooth HID Device Profile (API 28+). Phone appears as a HID FIDO device.

2. **CTAP2 protocol engine**
   - Implements `authenticatorMakeCredential`, `authenticatorGetAssertion`, `authenticatorGetInfo`, `authenticatorClientPIN`, credential management, etc.
   - CBOR encoding, CTAPHID framing (64-byte packets), CID management.
   - Crypto: P-256 ECDSA, ECDH for PIN protocols, HMAC-secret / PRF where possible.
   - Private keys stored in Android Keystore (hardware-backed when available).

3. **User verification**
   - BiometricPrompt (fingerprint / face) or device PIN/pattern.
   - Optional clientPIN for compatibility with sites that require it.

4. **Credential storage**
   - Resident (discoverable) credentials for true passkeys.
   - Non-resident for classic 2FA / U2F-style use.

## Realistic limitations (important)

| Path          | Root needed? | Works today? | Notes |
|---------------|--------------|--------------|-------|
| USB Gadget    | Yes          | Limited      | Needs kernel ConfigFS + HID gadget support. Many OEM kernels strip it. Use USB Gadget Tool to enable. |
| BLE HID       | No           | Best chance  | Android 9+. Host OS/browser must accept BLE FIDO keys (Windows is good; Linux/macOS variable). |
| Stock Android | N/A          | No           | Google does not allow third-party apps to become full USB CTAP devices without root/kernel support. |
| iOS           | N/A          | Very limited | No USB gadget for third parties. BLE HID is restricted. Platform passkeys + hybrid (QR) are the practical path. |

**This is not a drop-in replacement for a certified hardware key on every site.** Some services only accept USB or specific AAGUIDs. Certification (FIDO Alliance) is out of scope for a community project.

## Existing related open-source work (we build on these ideas)

- [tejado/android-usb-gadget](https://github.com/tejado/android-usb-gadget) — ConfigFS USB gadget profiles including FIDO CTAP HID.
- [tejado/Authorizer](https://github.com/tejado/Authorizer) — Password manager + experimental FIDO over BLE/USB (root for USB).
- [WIOsense/rauth-android](https://github.com/WIOsense/rauth-android) + [wiokey-android](https://github.com/WIOsense/wiokey-android) — FIDO2 roaming authenticator over BLE HID.
- [pokusew/lionkey](https://github.com/pokusew/lionkey), [google/OpenSK](https://github.com/google/OpenSK), [soft-fido2](https://github.com/pando85/soft-fido2) — Clean CTAP2 cores in C/Rust.
- Android platform passkeys + Credential Manager (for comparison; those are platform authenticators, not pure roaming USB keys).

## Project structure (planned)

```
PhoneFIDO-Key/
├── README.md
├── docs/
│   ├── architecture.md
│   ├── usb-gadget-setup.md
│   ├── ble-setup.md
│   └── threat-model.md
├── android/
│   ├── app/                    # Kotlin Android app
│   │   ├── src/main/java/.../ctap/
│   │   ├── src/main/java/.../transport/usb/
│   │   ├── src/main/java/.../transport/ble/
│   │   └── ...
│   └── ...
├── core/                       # Shared CTAP logic (if extracted)
└── LICENSE
```

## Quick start (current state)

1. Clone the repo.
2. For USB experiments: install [USB Gadget Tool](https://f-droid.org/packages/net.tjado.usbgadget/) on a **rooted** device, enable the FIDO CTAP profile, then run a CTAP implementation that talks to `/dev/hidg0`.
3. For BLE: the skeleton will target Bluetooth HID Device Profile + a CTAP2 state machine.
4. Test with https://webauthn.io or Windows Hello / Chrome.

Full build instructions and APK will appear as the code matures.

## Security notes

- Private keys should live in Android Keystore (StrongBox when available).
- User presence / verification is mandatory for registration and assertion.
- This software is experimental. Do **not** use it as the sole second factor for high-value accounts until audited.
- Prefer hardware-backed keys + biometrics over pure software keys.

## Roadmap

- [x] Repository + architecture document
- [ ] Minimal CTAP2 `getInfo` + `makeCredential` / `getAssertion` over BLE
- [ ] USB Gadget integration (root)
- [ ] Resident credentials + credential management
- [ ] PIN protocol v1/v2
- [ ] PRF / hmac-secret extension
- [ ] Basic UI for pairing, credential list, reset
- [ ] Threat model & hardening guide
- [ ] Optional: bridge mode for existing hardware keys

## Contributing

Issues and PRs welcome. Focus areas:
- Solid CTAP2 CBOR + CTAPHID framing
- Reliable BLE HID on major OEMs
- USB ConfigFS edge cases across kernels
- Keystore integration and key attenuation

## License

Apache-2.0 (or MIT — to be confirmed in LICENSE).

## Disclaimer

This project is not affiliated with the FIDO Alliance, Google, Yubico, or any hardware key vendor. It is an experimental community effort. Use at your own risk.
