# Architecture

## Goals

- Present the phone as a **roaming FIDO2 authenticator** with cross-platform attachment.
- Support CTAP 2.0 / 2.1 mandatory features first.
- Prefer hardware-backed cryptography via Android Keystore.
- Two transports: BLE HID (no root) and USB HID Gadget (root).

## CTAP2 command surface (initial)

| Command                  | Priority | Notes |
|--------------------------|----------|-------|
| authenticatorGetInfo     | P0       | Advertise versions, extensions, options, AAGUID |
| authenticatorMakeCredential | P0    | Create credential, UV required |
| authenticatorGetAssertion | P0     | Sign assertion, UV |
| authenticatorClientPIN   | P1       | PIN protocol v2 preferred |
| authenticatorCredentialManagement | P1 | List/delete resident keys |
| authenticatorReset       | P2       | Factory reset |
| authenticatorSelection   | P2       | |
| largeBlobs / hmac-secret | P2       | Extensions |

## Transport details

### USB (root)

1. Use ConfigFS (via USB Gadget Tool or direct sysfs) to create a composite gadget with a HID function whose report descriptor matches the FIDO CTAP HID descriptor (Usage Page 0xF1D0).
2. Kernel exposes `/dev/hidg0` (or similar).
3. App opens the character device with appropriate SELinux/permissions and implements the CTAPHID state machine (INIT, MSG, PING, CANCEL, KEEPALIVE).
4. Incoming 64-byte reports are parsed into CBOR CTAP commands; responses are framed and written back.

Reference HID descriptor can be taken from the FIDO CTAP spec or from tejado/android-usb-gadget.

### BLE HID

1. Register as a Bluetooth HID Device (Android `BluetoothHidDevice`).
2. Advertise the FIDO service / report map.
3. Handle HID reports the same way as USB once the transport is abstracted.

Many hosts (especially Windows) already treat BLE FIDO devices correctly.

## Crypto & storage

- Credential private keys: Android Keystore (ECDSA P-256). Prefer StrongBox.
- Master secret / PIN key material: Keystore-wrapped or derived via hardware-backed keys.
- Metadata (RP ID, user handle, credential ID, counter): encrypted SQLite or EncryptedSharedPreferences + Keystore.
- Signature counter: monotonic, persisted safely.

## User verification flow

1. Host sends makeCredential / getAssertion.
2. App shows BiometricPrompt (or falls back to device credential).
3. On success, perform the cryptographic operation and return the CBOR response.
4. Optional: require a physical button press or screen interaction for user presence.

## AAGUID & metadata

Use a project-specific AAGUID. Do not impersonate certified hardware AAGUIDs. Sites that whitelist specific AAGUIDs will reject the phone key — this is expected.

## Threat model summary

See `docs/threat-model.md` (to be expanded). High-level:

- Phone compromise → attacker can use resident credentials if they can unlock the device.
- USB/BLE channel is not encrypted by CTAP itself; rely on physical proximity and host OS protections.
- No attestation to a certified model; software attestation only (or none).
