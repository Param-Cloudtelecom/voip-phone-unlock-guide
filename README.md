# voip-phone-unlock-guide

A guide for removing **vendor/carrier provisioning locks** from desk
phones you legitimately own — refurbished, decommissioned, or bought
secondhand hardware that originally shipped locked to a specific hosted
PBX provider's auto-provisioning system. This is standard ITAD
(IT Asset Disposition) / VoIP reseller practice: a phone a previous
provider locked to their platform becomes useless e-waste unless it can be
reset to vendor-default and reprovisioned against a different PBX.

**Scope, explicitly:** this covers official, vendor-documented procedures
on hardware you own or are authorized to manage. It does not cover
bypassing security/DRM on a device you don't have the right to modify —
that's a different (and not legitimate) problem.

## The general pattern, across every vendor

Most "locked" phones aren't actually cryptographically locked — they're
just pointed at a specific provider's provisioning server via a
**redirection/zero-touch service** (the same services covered in
[`voip-phone-provisioning-guides`](https://github.com/Param-Cloudtelecom/voip-phone-provisioning-guides):
Yealink RPS, Cisco EDOS, Grandstream GAPS, Snom's redirection service).
Once a phone has phoned home to one of these once, it keeps re-checking
that same redirect on every factory reset *unless* you also clear the
locally cached redirect URL, not just the phone's SIP account config.

## Per-vendor procedure

### Yealink
1. Factory reset: `Menu > Advanced (default password admin) > Reset Config > Reset to Factory`
2. **Critical step most guides skip**: the RPS redirect URL is cached
   separately from the SIP config. Confirm `Settings > Auto Provision >
   Server URL` is genuinely blank after reset — if RPS already redirected
   it once, it can re-pull the old provider's config on next boot from the
   same network. Override `auto_provision.server.url` to your own server's
   URL manually if so.

### Cisco MPP
1. Factory reset: hold `*` and `#` and `4`... (model-dependent — check the
   specific model's hold-key combo) or via Admin Web UI > Factory Reset.
2. Confirm firmware is genuinely MPP (not a carrier-customized variant) -
   some carrier-sold units run modified firmware that can't be
   reprovisioned without first reflashing to stock Cisco MPP firmware via
   TFTP recovery mode.

### Grandstream
1. Factory reset via `Menu > System > Factory Reset`, or `***` DTMF reset
   code from the handset on some models.
2. GAPS redirect is tied to the MAC address on Grandstream's side, not
   cached locally — a factory reset is normally sufficient unless the unit
   was also locked at the firmware level by the original provider.

### Snom
1. Factory reset via web UI `Maintenance > Update > Reset values`.
2. If the phone still redirects to the old provider after reset, the
   redirection service entry persists server-side on Snom's end tied to
   the MAC — contact Snom support to release the MAC from the previous
   provider's account, this isn't something resettable from the phone
   itself.

## Verifying a phone is actually clean before redeploying

```bash
# Watch what the phone requests on boot, before pointing it at your real
# provisioning server - confirms it isn't still phoning home to the old
# provider's redirect service first
sudo tcpdump -i eth0 -n host <phone-ip> and port 80 or port 443
```

If you see a DNS lookup or HTTP request to a domain you don't recognize
before the phone hits your provisioning URL, the redirect cache wasn't
actually cleared — repeat the factory reset, or escalate to vendor support
for a server-side MAC release.

## Why this matters for a Cloud PBX deployment

Refurbished/secondhand desk phones are a real, common source of "new"
inventory for SMB Cloud PBX rollouts — being able to confidently
de-provision and re-provision hardware that already passed through another
provider is a basic operational skill, not a one-off trick.
