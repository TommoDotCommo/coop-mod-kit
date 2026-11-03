# The co-op mod standard

How a co-op mod's installer, launcher, in-game menus, keys, packaging and Steam Deck support should behave, so that every mod built this way feels the same to players. Each game keeps its own look; what is shared is where things are, what they are called and how they behave.

Rule IDs (A1, B2, ...) are stable: mods and task specs cite them. Research and sources behind the rules: [RESEARCH.md](RESEARCH.md). Interactive prototype: [the kit's page](https://tommodotcommo.github.io/coop-mod-kit/).


## A. LAUNCHER STRUCTURE

- **A1** Four tabs, this order and names: Play, Saves, Settings, About. Nothing else is top level.
- **A2** Play: your name, three tiles (Host a game / Join a friend / Play solo), one primary button that says what it does (Open lobby / Join <name> / Play solo). Last choice remembered.
- **A3** Hosting or joining turns Play into the lobby (tab label "Lobby"); other tabs stay usable; leaving returns to the tiles.
- **A4** "Rejoin <name>" shortcut on Play when the last host is online.
- **A5** Launcher skippable (MDK2: Show launcher at start, hold Shift to bring it back).
- **A6** Keyboard everywhere: Enter confirm, Esc back, Tab move, F1 quick guide (hold).

## B. FINDING EACH OTHER (joiner looks everywhere at once; host does nothing special; query/answer probe generalised from MDK2's CPQ1/CPS1, no traffic when nobody looks)

- **B1** LAN and L2 VPNs: directed broadcast on EVERY IPv4 adapter's subnet, to the default port and the next three. Never only 255.255.255.255 (Windows sends that on one adapter only).
- **B2** Tailscale: if installed, run `tailscale status --json` (no window; PATH or %ProgramFiles%\Tailscale\tailscale.exe), probe each Online peer's IPv4 TailscaleIP on the same ports. Shared-in nodes included. Nothing sent anywhere else. Setting: on by default.
- **B3** Steam friends whose rich presence shows this mod's lobby are listed when the game runs from Steam.
- **B4** Recent hosts remembered automatically after each successful join (name, route that worked, when); probed for live status; replace MDK2's "Saved addresses" (rename/pin/forget stay).
- **B5** Row: host name, state (in lobby / in game / full), players of limit, route (LAN, Tailscale, Steam friend, Internet, Recent), ping. Different version shown greyed with its version, not hidden.
- **B6** List fills as soon as Join is chosen, refreshes every 5 s while shown; each source named with a tick when done.
- **B7** One paste box accepts invite code, invite link, Steam lobby number, address, address:port, host name, Tailscale name; says what it recognised before Join.
- **B8** On launcher focus, an invite for this game on the clipboard is offered ("Fox's invite is on your clipboard. Join?"). Clipboard read only for that check, never stored or sent.
- **B9** Host chooses lobby visibility: "Open to my networks" (listed, joinable without code on LAN/Tailscale) or "Invite only" (not listed; code required). Steam lobbies stay friends-only.

## C. HOSTING AND THE INVITE

- **C1** Lobby panel "Friends can join by": Steam, Same network, Tailscale (with this PC's Tailscale name), Internet, Windows Firewall, each with a check result and its fix.
- **C2** While a lobby is open: UPnP-IGD or NAT-PMP mapping of the port (miniupnpc, BSD), removed when the lobby closes; on by default, Network setting to turn off. Router WAN address private or in 100.64.0.0/10 => "Your internet provider shares your address ... use Steam or Tailscale".
- **C3** Copy invite copies: "<name> is hosting <product>. Join: https://<join page>/#<CODE> (for example https://tommodotcommo.github.io/coop-mod-kit/join/#<CODE>)". Code carries every working route, port, lobby secret; internet address only if the internet check passed.
- **C4** Joiner tries all routes at once, keeps the first answer, prefers LAN > Tailscale > internet when several answer within 300 ms.
- **C5** The join page reads the code from the fragment (never reaches a server), offers "Open in launcher" (<mod>-coop:// handler registered per user by Setup), the code to copy, and a download link.

**Invite code v1:** "<GAME>-" + Crockford base32 (case-insensitive, I/L->1, O->0 accepted), groups of 5 separated by '-'. Payload bytes: [0] version(4 bits)=1 | flags(4 bits: bit0 has-steam, bit1 invite-only); [1..4] lobby secret u32 LE; [5..6] port u16 LE; [7] n routes (0..4); then n x (kind u8: 1 LAN, 2 Tailscale/VPN, 3 internet; IPv4 4 bytes); if has-steam: Steam lobby id u64 LE; last byte CRC-8 (poly 0x07, init 0) over all previous bytes. GAME prefixes: MDK2, WIT. Any CRC/length error => "This code is incomplete. Ask for the whole invite again." Wrong prefix => "That invite is for <other game>."


**Probe v2** (UDP, same port as the lobby or game listener; hosts keep answering CPQ1): query "CPQ2" + nonce u32 + game id (4 ASCII, e.g. "MDK2", "WIT ") + flags u8. Reply "CPS2" + nonce + game id + state u8 (0 lobby,1 in game,2 reachable only) + players u8 + limit u8 + flags u8 (bit0 invite-only, bit1 version mismatch possible) + version len u8 + version ASCII (1..47) + name len u8 + host display name UTF-8 (0..32; empty when invite-only) + [if invite-only] nothing else. Replies only to queries with the same game id. Rate-limit replies per source (e.g. 10/s).


## D. LOBBY

- **D1** Three columns: players (host mark, ready, ping, speaking light, per-player mute) | chat | "Friends can join by" above "What to play".
- **D2** What to play = game slot, host chooses, read-only for others.
- **D3** Host button "Start game", enabled when all ready, else "Waiting for <names>"; others "Ready"/"Not ready"; Leave always one click.
- **D4** Lobby voice follows Voice settings; mute me / deafen in the players column.
- **D5** After the game everyone returns to the same lobby with the result in chat.

## E. SETTINGS (left: sections; right: rows). Sections in order: Co-op rules (host) | Players and interface | Voice and sound | Controls | Display | Network | Launcher. Shared rows: Co-op rules: who can see my lobby, player limit. Players and interface: your colour, name tags, text chat, pings. Voice and sound: voice chat (off/open/push to talk), push-to-talk key, microphone, output, voice volume, mic volume, hearing range (host). Controls: key bindings by press-a-key, gamepad on/off. Display: screen mode, resolution, frame limit. Network: port, find games on my network, find friends on Tailscale, open the router port while hosting, put my internet address in invites. Launcher: show at start, check for updates, offer pre-releases. Games add their rows in the right section.

- **E1** Saved on change; no Save button; "Reset section" per section.
- **E2** Host-decided rows tagged "host decides"; guests in a lobby see the host's value greyed.
- **E3** Sliders with units, choosers, press-a-key capture (refuses duplicates and F1). No typed numbers except the port.
- **E4** Same setting, same name in launcher, in-game menu and quick guide.

## F. SAVES, ABOUT, MESSAGES

- **F1** Saves: co-op saves (who played, progress, when), then the New game slot; actions Host this save, Play solo, Back up, Delete (confirm in page).
- **F2** About: version, updates (check, install, roll back, auto-check), quick guide, Report a problem (diagnostics), links, credits.
- **F3** Update available => banner on Play (Update / What's new); never interrupts a lobby or running game.
- **F4** Every failure says what happened, names who/what, offers the next step as a button (Try again, Update, Fix firewall, Send diagnostics).

## G. SETUP

- **G1** Finds every install (Steam libraries, GOG, Epic manifests, MDK2 Overhaul build) as cards with a build check; Browse fallback.
- **G2** One state-driven button (Install/Update/Repair/Up to date/Needs attention); uninstall restores originals, keeps saves.
- **G3** Three ticked options: desktop shortcut; allow hosting through Windows Firewall on Private AND Public (one admin prompt); open invite links in the launcher (HKCU <mod>-coop:// handler, no admin). Uninstall removes all three.
- **G4** Last screen's primary button is Play.

## H. KEYS AND CONTROLLER

- Shared co-op keys (same default in every mod; MDK2's set is the base): hold F1 quick guide; F2 mark a problem (bug marker); T text chat; V or middle mouse ping; B or mouse side button (XBUTTON1) push to talk; hold H emote wheel; hold Tab player list; [ and ] watch previous/next player.
- Moves: Witness P->V ping, N->B push to talk, G->H emote, F5 roster->hold Tab, V watch->from the player list then [ ], next voice channel H->Y, ink colour B->mouse wheel while drawing. MDK2 Modern preset: open inventory loses Tab (F stays). Developer keys stay where they are, work only in developer mode, never on a shared key.
- Examples checked in the two reference games: The Witness itself does not use Tab, V, B, H or Y (its Steam controller.vdf maps only WASD, LShift, Esc, mouse L/R; its UI hints cover move, run, mouse buttons). Its pad hints use A, B, X, Y, LB, RB, L3, R3, D-pad, never View/Back. MDK2's pad: D-pad moves the player; Back/View = ping today.
- **H1** shared keys as above in every mod.
- **H2** game-specific co-op actions may use any other key, never a shared key, never a key the game uses in any preset.
- **H3** hold opens/release acts for wheels, talk, player list, guide; toggles are single presses.
- **H4** every key rebindable by press-a-key; conflicts checked within action sets used together (MDK2 model); F1 fixed.
- **H5** guide, Controls page and launcher read ONE binding table.
- **H6** controller: see K. Migration: only defaults move; a player's own changed key stays; launcher What's new lists moved keys once.

## I. MENUS AND HUD

- **I1** One "Co-op" entry in the game's options menu, sections and names as launcher Settings, plus a Session page: players, invite friends, copy invite, save now (host), leave.
- **I2** Witness (no in-game settings today): the F5 roster becomes the hold-Tab panel with session actions and cheap live toggles (voice volumes, name tags, partner picture); the rest stays in the launcher marked "applies next start".
- **I3** Player list row: colour, name, host mark, ping, speaking light, area or distance / state (loading, down, watching). Mouse: mute, volume, watch, (host) kick.
- **I4** Zones: player strip top-left; notices top-centre (max 3, 4 s); chat bottom-left; banners needing an answer bottom-centre; name tags/pings/typing dots in the world; each game lists its own HUD areas and co-op stays out.
- **I5** One colour per player everywhere (lobby, tag, chat name, ping, ink, outline), assigned in join order from a palette readable on light and dark scenes; changeable.
- **I6** Same words for the same events: "Fox joined", "Fox left", "Lost Fox, waiting up to 30 s", "Reconnecting to Fox", "Fox has a different version (v0.4.0)". One wording file per mod, same strings.
- **I7** Esc never leaves co-op without one confirmation saying what happens ("The others keep playing" / host: "Everyone returns to the lobby"). Game keeps running behind co-op menus and they say so.
- **I8** Text readable at 1280x720 and 1280x800, scales with the window; both mods run a text-size / on-screen test (the Witness has UiCheckTarget / UiCheckReachable).

## J. MOD MANAGERS AND LOADERS (copying the package's files is a complete install; deleting them a complete uninstall; the mod still knows what it is and who installed it)

- **J1** Drop-in package: zip mirrors the game folder; no step renames, edits or patches a file the game ships; anything outside the game folder the mod sets up itself at first start and copes with being removed.
- **J2** Proxy a Windows system DLL the game imports but does not ship; load the real one from the system folder (GetSystemDirectory). Not dxgi/d3d11 (ReShade, Special K, DXVK). Witness candidates: dsound.dll (about 12 exports) or winmm.dll; both imported by witness64_d3d11.exe.
- **J3** Chain-load: an existing non-ours file with the proxy's name is kept as <name>.chain.dll and loaded by our proxy after itself; diagnostics report what was chained.
- **J4** Nothing written into the game folder at run time: settings, logs, dumps, update backups, markers in %LOCALAPPDATA%\<mod>\. Settings found in the game folder are moved once. Lab/dev exception: an explicit per-instance override file placed by test tools may sit beside the exe; the mod never creates it.
- **J5** coop-mod.json at package root, generated by the release script, read by Setup, updater, diagnostics: {"schema":1,"id","name","version","game":{"name","steam_appid","exe":[...],"builds":[{"name","sha256"}]},"proxy":{"file","system_dll","chain_file"},"proton":{"dll_overrides"},"files":[{"path","sha256","size"}],"user_data","homepage","updates":{"github","nexus"}}. Every DLL/exe has a VERSIONINFO resource with the mod version.
- **J6** Know who installed it: Setup receipts => Setup; no receipt but hashes match a release => mod manager or by hand (a file with more than one hard link => almost certainly Vortex); Setup shows "Installed by a mod manager" + Play, not "Needs attention".
- **J7** Updater still checks; when not Setup-installed it shows "Update it in your mod manager" (Vortex wording when hard-linked) with the Nexus/GitHub link instead of replacing files.
- **J8** One version: GitHub Releases canonical; Nexus gets the identical zip <mod>-<version>.zip and version string.
- **J9** Scriptable Setup verbs everywhere (Setup.exe and install-linux.sh): status --json, install <game folder> --yes, uninstall <game folder> --yes; documented exit codes (0 ok, 1 failed, 2 usage, 3 unsupported build, 4 game running).
- **J10** Wrong build or missing pieces never crash the game: co-op turns off and the launcher says why.

## K. STEAM DECK AND CONTROLLERS (whole session in Deck Game Mode with no keyboard/mouse/desktop)

- **K1** Play in Steam opens the co-op launcher, Game Mode included, no non-Steam shortcut. Witness: the game started without a launcher session hands over to the launcher (as the Steam-invite +connect_lobby path already does in the proxy); whole session stays inside the one Steam launch.
- **K2** No launch options to type: the Linux install writes the DLL override for the game exe into the game's Proton prefix (compatdata/<appid>/pfx/user.reg: [Software\\Wine\\AppDefaults\\<exe>\\DllOverrides] "<dll>"="native,builtin"); launch option stays in README as fallback; diagnostics report whether it is in place.
- **K3** Launchers fully usable with a pad: stick/D-pad focus, A confirm, B back, LB/RB tabs, X/Y context actions shown in the footer, Start = primary button.
- **K4** Shared co-op pad layer on View/Back: tap = ping; hold = player list, and while held D-pad up emote wheel (right stick picks), down chat, left/right watch previous/next; D-pad presses are swallowed from the game while View is held.
- **K5** Voice on a pad: open mic with the voice gate by default when the last input was a pad; push to talk bindable to a free stick click.
- **K6** Text without a keyboard: chat/name/paste open Steam's floating keyboard (ISteamUtils ShowFloatingGamepadTextInput) when Steam runs, else our own on-screen keyboard; quick-chat wheel (Ready, Wait for me, Over here, Come look at this, Thanks).
- **K7** Hints show pad buttons after pad input, keys after keyboard input, switching at once; Xbox icons default.
- **K8** Fits 1280x800 16:10; the Witness launcher window becomes resizable and fills the screen in Game Mode.
- **K9** A published Steam community controller layout per game (back buttons: push to talk, ping, player list).
- **K10** Tests: T0 for navigation, pad layer, glyph switching from recorded pad states; T1 with the mods' pad-injection hooks; a real Steam Deck session is the last gate.
