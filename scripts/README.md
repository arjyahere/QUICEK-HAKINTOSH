# Scripts & Post-Install Helpers

Small fixes that have to be applied **after** all three operating systems are installed.

---

## `windows-utc-clock.reg` — Cross-OS clock drift fix

### The problem

Windows writes *local time* into the motherboard RTC. macOS and Linux write *UTC* and convert
in software. On a triple-boot machine this means every switch between operating systems makes
the other OS show a time that is off by your timezone offset.

### The fix

The registry value `RealTimeIsUniversal` tells the Windows kernel the RTC already holds UTC.

### How to apply

1. Boot into Windows 11.
2. Double-click `windows-utc-clock.reg`.
3. Accept the UAC prompt.
4. Reboot.

### Verify

Open PowerShell **as Administrator** and run:

```powershell
Get-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\TimeZoneInformation' `
  -Name RealTimeIsUniversal
```

Expected output: `RealTimeIsUniversal : 1`

Then check that Windows still shows the correct local time, and reboot into macOS/Linux
to confirm the clock no longer jumps.

### Revert

Set the value back to `0`, or delete it:

```powershell
Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\TimeZoneInformation' `
  -Name RealTimeIsUniversal -Value 0 -Type DWord
```

```powershell
Remove-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\TimeZoneInformation' `
  -Name RealTimeIsUniversal
```

Reboot afterwards.

### Notes

- Requires a reboot; the kernel reads this value at boot.
- If a later Windows feature update resets the value, re-apply the `.reg` file.
- Some OEM images ship a scheduled task that rewrites time settings — check
  **Task Scheduler → Task Scheduler Library → Microsoft → Windows → Time Synchronization**
  if the value keeps reverting.

---

## Adding more helpers

Document any new script in this file with the same three sections: **The problem**,
**How to apply**, **Revert**. Keep scripts idempotent — running one twice must not break
anything.
