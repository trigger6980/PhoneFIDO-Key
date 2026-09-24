# USB Gadget Setup (Root Required)

## Prerequisites

- Rooted Android device
- Kernel with ConfigFS and USB gadget support (`CONFIG_USB_CONFIGFS`, HID function, etc.)
- [USB Gadget Tool](https://github.com/tejado/android-usb-gadget) (F-Droid or GitHub)

## Steps

1. Install and open USB Gadget Tool.
2. Create or select a gadget profile that includes the **FIDO CTAP** function (HID with CTAP report descriptor).
3. Activate the gadget. The phone should now appear to the host computer as a FIDO security key (check `lsusb` / Device Manager).
4. Confirm `/dev/hidg0` (or the node reported by the tool) exists and is accessible.
5. Run PhoneFIDO-Key (once implemented). It will open the HID node and speak CTAP2.

## Troubleshooting

- If the host does not see a FIDO device, the report descriptor or VID/PID may need adjustment.
- SELinux may block access to `/dev/hidg*`. Magisk modules or `supolicy` rules are often required.
- Some OEM kernels disable ConfigFS or HID gadget; custom kernels (LineageOS, etc.) are more likely to work.
- Only one active gadget at a time is typical; disable MTP/ADB if needed.

## Alternative: FunctionFS

Advanced users can implement FunctionFS HID instead of ConfigFS. This is more complex and left for later.
