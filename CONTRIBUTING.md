# Contributing

Thanks for wanting to improve Niripedia. Arch changes fast, and packages get renamed or dropped, so a fix from someone who just followed the guide is worth a lot. Everyone taking part agrees to the [Code of Conduct](CODE_OF_CONDUCT.md).

## What helps most

- **A step that fails.** A command errors, a package was renamed or moved to the AUR, or a config option changed.
- **An outdated section.** The Noctalia section covers the old shell, v4, and a newer Noctalia has been released since. Steps for the new one are the most useful addition right now.
- **A missing explanation.** Somewhere you got stuck because the guide assumed too much.
- **A typo or a broken link.** These are quick, and they're welcome.

If you're not sure whether something is worth changing, open an issue and ask. A short question costs nothing.

## Reporting a problem

Open an issue with the **Wrong or outdated step** form. It asks for the section, what happened and your package versions, which is usually enough to reproduce a problem. For a conduct problem, follow [Reporting a problem](CODE_OF_CONDUCT.md#reporting-a-problem) instead. For a command that could harm someone's system, see [SECURITY.md](SECURITY.md).

## Sending a change

1. Fork the repository and create a branch for one change.
2. Edit the README, or the file you're fixing.
3. Check your change against the list below.
4. Open a pull request and fill in the template. Say what you changed and what you tested it on.

Small pull requests get reviewed faster than large ones. One fix per pull request is ideal.

## Before you submit

Every command in the guide should have been run, not just read. The checks below catch most mistakes:

- **Run it on a current Arch install.** A virtual machine works well. Test the whole section from a clean state, not only the line you changed.
- **Check package names.** `pacman -Si <package>` for the official repositories and `yay -Si <package>` for the AUR. Use `pacman` for official packages and `yay` for anything from the AUR or for mixed lists.
- **Check Niri config.** Run `niri validate` on any config you add, and make sure keybindings sit inside the `binds { }` block.
- **Check links.** Anchors in the contents list must match their headings, and external links must open.
- **Say when a change takes effect.** Tell the reader whether it applies at once, after logging out and back in, or after a reboot.
- **Say how to undo it** when the step changes something that's hard to find again.

## Writing style

The guide is written for a reader who is typing along. Keep it consistent:

- Plain, direct sentences in the second person. No hype, and no jokes at the reader's expense.
- Be honest about limits and risks, and say which steps can erase data.
- Exact commands in code blocks, with the language set (`bash`, `kdl`, `text`).
- Backticks for package names, commands and file paths. Bold for the names of things on screen, such as menu items.
- Tables for lists where each row has the same kind of facts. Short prose for explanations.
- No emojis.
- Say what a command does before or right after it, and explain any option the reader might not know.

## Licensing

By sending a change you agree that it's released under the same [MIT license](LICENSE) as the rest of the project. Don't paste in text or commands from another project unless its license allows it, and credit the source in the pull request.
