# Ticket #001 — PCs Not Receiving DHCP Addresses

**Related Version:** office-lan  **Severity:** High

## Reported Symptom
PC0 failed to receive an IP address via DHCP; a follow-up check showed all PCs on the LAN were failing DHCP requests.

## Scope / Impact
All DHCP clients on the LAN (PC0–PC3, Printer0) — no devices could obtain an address automatically.

## Investigation
1. Ran `show run | section dhcp` on R1 — no `excluded-address` line appeared at all, only the pool itself.
2. Ran a full `show run` and searched for `excluded-address` — confirmed it was missing entirely, meaning an earlier command hadn't actually been saved to the running config.
3. Re-entered the exclusion command directly and verified it took effect this time.
4. Re-checked DHCP — requests began failing for all PCs.
5. Ran `show run | section dhcp` again — found the exclusion range set to `192.168.1.1 – 192.168.1.254`, covering nearly the entire subnet.
6. Ran `show ip dhcp binding` — table was empty, confirming no addresses were being leased.

## Root Cause
The DHCP exclusion range was set too broad (`192.168.1.1–192.168.1.254`), leaving no usable addresses in the pool for `LAN-POOL` (`192.168.1.0/24`) to assign.

## Fix Applied
```
no ip dhcp excluded-address 192.168.1.1 192.168.1.254
ip dhcp excluded-address 192.168.1.1 192.168.1.10
```
## Verification
- `show run | section dhcp` confirmed the exclusion range was corrected to `192.168.1.1–192.168.1.10`.
- Forced a fresh DHCP request on a PC — received an address in the `192.168.1.11–15` range again, matching the original working state.
