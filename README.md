# coop-mod-kit

A shared standard and reference code for co-op mods: how the installer, the launcher, the in-game menus and keys,
mod-manager packaging and Steam Deck support should behave, so every co-op mod built this way works the same for players.
Each game keeps its own look.

Used by [MDK2 Co-op](https://github.com/TommoDotCommo/mdk2-coop) and [The Witness Co-op](https://github.com/TommoDotCommo/witness-coop).
Built alongside the [Co-op Anything](https://github.com/cranebeaker/coop-anything) playbook.

| | |
|---|---|
| [STANDARD.md](STANDARD.md) | The rules, A to K: launcher structure, finding friends without typing addresses, hosting and invites, lobby, settings, Setup, keys and controller, menus and HUD, mod managers and loaders, Steam Deck. |
| [RESEARCH.md](RESEARCH.md) | Why: the findings behind the rules, with sources. |
| [Prototype](https://tommodotcommo.github.io/coop-mod-kit/) | A clickable model of the shared launcher in three skins, and the keyboard map. |
| [reference/](reference/) | Portable C99 code: invite codes (`coop_invite.h`), the discovery probe and friend finding (`coop_discover.h`), the Windows adapter (`coop_discover_win.h`), with tests. See [reference/DISCOVERY.md](reference/DISCOVERY.md). |
| [Join page](https://tommodotcommo.github.io/coop-mod-kit/join/) | Where invite links point. It reads the code after `#` in the browser; nothing is sent to a server. |

## Using it in a mod

Copy the `reference/` headers into your mod and note the kit's tag you copied from. Run the tests with `make -C reference test`.
Cite rule IDs (for example "B1, C3") in your mod's docs and issues so changes can be traced back here.

## License

MIT, see [LICENSE](LICENSE).
