# Research behind the standard (November 2026)

Each finding names its sources. Where a source was a forum post or a third party, the rule built on it is tested in the reference mods before it is relied on.

## Finding each other

- **Tailscale carries no broadcast or multicast**, so LAN-style game discovery finds nothing over it. Every device on a tailnet, including machines shared from friends, is listed by `tailscale status --json` with its 100.x address and an online flag, so a launcher can ask each online peer directly (rule B2). [Tailscale forum](https://forum.tailscale.com/t/broadcast-multicast-support/156), [ipnstate status fields](https://git.quad4.io/Mirrors/tailscale/src/commit/6d02a48d8d6e3d11db4c3d467c85bc36386058a3/ipn/ipnstate/ipnstate.go)
- **Shared Tailscale machines accept connections but cannot start them** (quarantine). That suits host/client games: the host shares their PC, friends join. [Tailscale: sharing](https://tailscale.com/kb/1084/sharing)
- **Windows sends 255.255.255.255 out of one adapter only**, so a PC on Wi-Fi plus a VPN misses half the games. Send a directed broadcast on each adapter's subnet instead (B1); L2 VPNs such as ZeroTier then work too. [WinIPBroadcast](https://github.com/dechamps/WinIPBroadcast), [ZeroTier forum](https://discuss.zerotier.com/t/lan-gaming-discovery-issue/18902)
- **Good co-op games layer the ways in**: Stardew Valley lists Steam friends' farms, takes invite codes, and keeps join-by-IP for LAN; Factorio has a LAN browser and Connect to address; Minecraft announces LAN worlds every 1.5 s. Typing an address is the fallback, not the main path. [Stardew wiki](https://stardewvalleywiki.com/Multiplayer), [Factorio wiki](https://wiki.factorio.com/Server), [LAN Announcer](https://www.curseforge.com/minecraft/mc-mods/lan-announcer)

## Hosting and invites

- **Discord makes only http, https and discord links clickable**, so invites are https links to a static page, plus a code the launcher picks up from the clipboard (C3, C5, B8). [Discord support thread](https://support.discord.com/hc/es/community/posts/14574789338135-Steam-URI-links-no-longer-supported)
- **UPnP and NAT-PMP only configure the player's own router**; behind carrier-grade NAT (WAN address in 100.64.0.0/10) nothing at home opens a port, so the launcher says so and points to Steam or Tailscale (C2). [RFC 6970](https://www.rfc-editor.org/rfc/rfc6970.txt), [CGNAT detection](https://docs.ipcheck.ing/knowledge-base/diagnose/am-i-behind-cgnat)
- **The Windows Firewall first-run prompt allows private networks only**; VPN adapters can be public, so friends cannot reach the host and nothing says why. Setup adds the rule for both profiles, the lobby checks it (C1, G3). [Tailscale docs](https://tailscale.com/docs/reference/messages/client/set-network-category-failed), [Tailscale forum](https://forum.tailscale.com/t/win-10-stopped-connecting-cant-get-ping-response/755)

## Launchers

- **Players resent launchers that add steps**: a second Play button, silent failures, repeated prompts. One studio pulled its launcher within a day. A mod launcher must be faster than going without it (A2, A4, A5, F4). [PC Gamer](https://pcgamer.com/games/rpg/fan-revolt-forces-rpg-studio-to-kill-its-new-launcher-just-19-hours-after-release/), [Aftermath](https://aftermath.site/ubisoft-ea-paradox-launcher-pc-steam)

## Mod managers and loaders

- **Mod managers copy files and later remove exactly those files.** Vortex hard-links an archive's files into the game folder; a program that later changes them triggers Vortex's "external changes" prompt, where players can lose files (J1, J4, J6, J7). [Vortex wiki](https://github.com/Nexus-Mods/Vortex/wiki/MODDINGWIKI-Users-General-Managing-External-Changes/Home)
- **Mod Organizer 2 adds games through a short Python plugin** (`basic_games`); root-folder files such as proxy DLLs need Root Builder. [basic_games](https://github.com/ashemedai/modorganizer-basic_games), [Root Builder](https://www.nexusmods.com/skyrimspecialedition/articles/6614)
- **Proxy DLLs share a small set of names** (dinput8, dsound, winmm, version, ...), so two mods overwrite each other unless one loads the other: chain-load (J2, J3). [Ultimate ASI Loader](https://www.github.com/ThirteenAG/Ultimate-ASI-Loader)

## Steam Deck and controllers

- **Deck checks** ask for full controller support with the right icons, the on-screen keyboard when text is needed, and legible text at 1280 x 800 (K). [Deck Verified criteria](https://destructoid.com/?p=289844)
- **Steam's floating keyboard** can be opened over a text field from the Steam API and reports when it closes (K6). [Steam utils API bindings](https://luasteam.readthedocs.io/en/stable/auto/utils.html)
- **Proton uses its own copy of Windows DLLs** unless told otherwise, so a system-DLL proxy needs a DLL override; the install writes it into the game's Proton prefix instead of asking players to type a launch option (K2).

## Code signing

- **Signing builds reputation; it does not skip the SmartScreen warning.** Since 2024 even EV certificates start without reputation. For open-source projects, SignPath Foundation signs builds made by a public CI, free, with the Foundation as the listed publisher. [Microsoft: SmartScreen reputation](https://learn.microsoft.com/et-ee/windows/apps/package-and-deploy/smartscreen-reputation), [SignPath Foundation terms](https://signpath.org/terms)
