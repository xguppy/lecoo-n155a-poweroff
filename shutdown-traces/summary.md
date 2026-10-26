# Shutdown trace comparison, 2026-10-26 05:25 (+04)

Kernel 7.0.0-38-generic, BIOS 2.5, parameters: printk.always_kmsg_dump=1 pstore.kmsg_bytes=65536
initcall_debug dyndbg="file ec.c +p". Logs from EFI pstore via systemd-pstore,
deduplicated and sorted by timestamp: shutdown-1-FAILED.log, shutdown-2-FAILED.log, shutdown-3-OK.log.

| # | boot | result (owner) | next boot after |
|---|---|---|---|
| 1 | 881eeaef (up 31 s) | turned itself on | 12 s |
| 2 | f7b25a0d (up 22 s) | turned itself on | 14 s |
| 3 | 30586d7e (up 22 s) | stayed off | manual, 32 s |

## Findings

- Device shutdown: 149 devices, same order in all three runs (only btusb
  interface order differs, async).
- Timings after "Powering off" match within tens of ms: xhci ~0.2 s, ccp ~0.14 s,
  nvme ~0.2 s, then `ACPI: PM: Preparing to enter system sleep state S5` → _PTS
  (0.318 s, the TPM.TPTS Sleep(300)) → syscore → `reboot: Power down`.
- Only notable variation: `tpm tpm0: shutdown_pre` (TPM2_Shutdown to fTPM) took
  88 ms / 577 ms / 58 ms (fail / fail / ok). Run 1 failed with a time close to the
  successful run, so this does not separate outcomes.
- EC: no EC transactions or EC query events at all during shutdown in any run.
  The last EC traffic is 7-11 s before power-off (normal polling).
  `_PTS(5)` does not touch the EC.
- After "Power down" the kernel spent ~5 s writing the pstore dump to EFI
  variables (record timestamps span 5 s) before the final S5 write. Failures
  still happened with this extra delay.

## Conclusion

Linux does exactly the same thing in failed and successful shutdowns, up to the
last message before the S5 write. The outcome is decided after the OS hands
control to firmware (SLP_SMI / S5 handling, EC, PSP), not by anything Linux
does differently between runs.
