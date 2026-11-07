# iOS 27 arm64e hardware gate

## Conclusion

The OnePlus 7 Pro (`guacamole`, SM8150) cannot execute an iOS 27 arm64e
system. The limitation is in the CPU hardware, not in the bootloader, XNU
configuration, hypervisor setup or GPU driver.

A RAM-only probe booted on the handset on 2026-11-06 and measured:

```text
CurrentEL        = EL1
MIDR_EL1         = 0x51df805e
ID_AA64ISAR1_EL1 = 0x0000000000100001
ID_AA64ISAR2_EL1 = 0x0000000000000000
APA=0 API=0 GPA=0 GPI=0 APA3=0 GPA3=0
PAC verdict      = NO FEAT_PAuth
```

The probe image SHA-256 is:

```text
42e651cbab8f6dd3cf48757b68c2a0aed74e43648c43f29e91a876c385e56366
```

No partition was flashed or erased by this test.

## arm64 and arm64e

`arm64e` is not a different instruction set. It is Apple's pointer
authentication ABI built on ARMv8.3 Pointer Authentication (PAC).

An arm64e binary signs pointers and validates them before indirect calls,
returns and some data-pointer uses. Typical instructions include
`PACIA/PACIB`, `AUTIA/AUTIB`, `PACIASP/AUTIASP` and `RETAA/RETAB`.

That model requires native hardware support. Removing the instructions from
the kernel is not a valid fix because the system userland, shared cache and
closed-source components still use the arm64e ABI.

Emulating every PAC instruction through exceptions would effectively turn the
phone into a slow instruction-level simulator. It is not a usable iOS port.

## Pass condition for another SoC

Every enabled CPU must satisfy all of the following:

- FEAT_PAuth is implemented in hardware.
- Address and generic authentication are available at EL0 and EL1.
- The PAC key registers can be configured at EL1.
- PAC instructions do not trap.
- EL2 does not block PAC through `HCR_EL2.APK/API`.
- The platform supports native 4 KiB pages.
- Device-specific boot, interrupt, storage and display support exists.

The ID-register test passes when either:

```text
(APA != 0 || API != 0) && (GPA != 0 || GPI != 0)
```

or:

```text
APA3 != 0 && GPA3 != 0
```

The conditional PACIA/AUTIA round trip must also restore the original pointer.
The test must then be repeated on every CPU that XNU enables.

## SoC direction

| SoC class | Status |
| --- | --- |
| SM8550 | Preferred current iOS 27 research target; UniXNU `sheng` provides the most relevant reference |
| SM8450 / SM8475 | Theoretical PAC candidates, but each core and EL2 configuration must be measured |
| SM8650 / SM8750 | Hardware may qualify, but the UniXNU device and driver work is substantially larger |
| SM8150 / Adreno 640 | Excluded; the probe shows no FEAT_PAuth |

SoC qualification alone does not make a device bootable. SM8550 moves the
target to Adreno 740, so A640 GPU registers, GMU/ZAP firmware handling and
scanout code cannot be reused as a drop-in implementation.

## Scope of the arm64 baseline

The successful arm64 XNU compile performed on the Mac is useful proof that the
local toolchain and SM8150 memory profile work. It is not an iOS 27 path and it
is not boot-qualified.

The one-handset route remains:

- iOS 16.7.16 on SM8150, where the existing port is already functional.
- Investigation of a separate iOS 18 arm64 profile only after its IPSW and XNU
  support are verified.

The iOS 27 route requires an arm64e-capable test device. The next useful work
is to select a concrete iPhone identity and IPSW for that hardware, then study
the SM8550 / `sheng` IOMFB, refresh-rate and Metal paths.

## References

- <https://unixnu.com/zh/porting/arm64e>
- <https://github.com/UniXNU/UniXNU-sheng>
