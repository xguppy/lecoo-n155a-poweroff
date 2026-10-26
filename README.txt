Lecoo N155A: spontaneous power-on after S5 poweroff. Data bundle, 2026-10-26.
Identifiers (MACs, host name, root UUID, USB serial) are redacted.

dmesg-normal-boot.txt        kernel log of a normal boot (7.0.0-38, Secure Boot on)
lspci-nnk.txt                PCI devices and drivers
acpi-tables/*.dat            raw ACPI tables (acpidump -b), BIOS 2.5
registers/registers.txt      PM1/GPE0, FCH PMx74, GPIO wake bits, EC flags
registers/gpio-debugfs.txt   /sys/kernel/debug/gpio (pinctrl-amd)
registers/ec-ram-0x00-0xff.bin  EC RAM dump via ec_sys (read-only)
shutdown-traces/             kernel log up to "reboot: Power down" from EFI pstore,
                             deduplicated and sorted:
                               shutdown-1-FAILED.log  (powered back on)
                               shutdown-2-FAILED.log  (powered back on)
                               shutdown-3-OK.log      (stayed off)
                             summary.md: comparison
poweroff-history.txt         every poweroff in the journal since 2026-08-05:
                             uptime, gap until next boot, AC state at boot.
                             FAIL = next boot <= 11.5 s later (automatic power-on);
                             "manual?" = 12-40 s, ambiguous (owner pressing power
                             or a slower automatic power-on, e.g. with the pstore
                             dump active the automatic ones took 12-14 s)
