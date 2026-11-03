# v70 built-in HFS kernel

This branch is a narrow build input for the OnePlus 7 Pro HakiOS kernel.

- Base kernel: `Gsjsjzhznsz/hackios-xnu-kernel@4c0ed41d78e6bcbff57d3d32d678c965f49cd1e4`
- GPU driver: the verified guacamole/Raphael path, including the non-fast
  `SCM_PAS_INIT_IMAGE` zap-signature call.
- Filesystem addition: Apple `hfs-627.100.6` built into XNU as `options HFS`,
  registered through the static VFS table.
- The files under `xnu-root` are copied over the patched XNU tree after
  `xnu/apply.sh`; no ios-sweet patch stack is used.

The `hfs-builtin-v70` workflow builds this input with the same pinned Theos
toolchain used by the HakiOS K20P workflow and tests for the linked HFS symbols
before uploading the kernel artifact.
