# Pygame Platformer

A deliberately buggy pygame platformer built for the **ADLC Day Of Learning** as a hands-on demo of `fullsend`, our Autonomous Development Lifecycle (ADLC) tool. The game is small, runnable, and infected with three planted bugs of varying severity. It is the perfect playground for practicing issue creation, handoff, and automated resolution with `fullsend`.

## Intent

The goal of this project is *not* the game itself. The goal is to give everyone a realistic, low-stakes codebase to exercise the ADLC workflow end to end:

1. Play the game and observe a bug in action.
2. Write a well-formed issue describing the bug.
3. Let `fullsend` pick up the issue and resolve it.
4. Verify the fix by playing again.

Because the bugs are planted and fully understood, this is a safe environment to
see how `fullsend` behaves when given a real issue against a real codebase — with
no risk to production code.

## Prerequisites

- Clone / fork this repo in the ADLC-Day-Of-Learning organization. Give it a unique name when you fork the repository. For example, prefix the repository with your username: `dmacpher-plaform-game`.
- Download the Fullsend CLI on your laptop and install agents on your cloned repository
- Install `uv` on your laptop, then set up and run the game with it:
  ```bash
  uv python install 3.13   # pygame has prebuilt wheels for 3.13, not 3.14+
  uv sync
  ```
  This creates a virtualenv with Python 3.13 (pinned in `.python-version`) and installs pygame into it.

## How the game works

The game is a single-file pygame platformer (`platformer.py`). Run it with:

```bash
uv run platformer.py
```

- **Move:** arrow keys or `A`/`D`
- **Jump:** `SPACE`, `W`, or `Up` (release early for a shorter hop)
- **Goal:** collect all 6 coins, then touch the red flag to win
- **R** restarts the game at any time. Falling off the bottom of the screen respawns you at the start

The player has gravity, variable jump height, and per-axis collision resolution against the platforms. Reaching the flag after collecting every coin shows the win screen.

## The three bugs

**Spoiler section:** if you're playing the bug hunt, stop reading here and go play
the game first!

### Bug 1: Coin collection is disabled

Walking over a coin does nothing. The coin stays on screen and the counter never leaves 0/6, which makes the game unwinnable by legitimate means. The coin pickup logic in the main game loop includes a guard condition that was intended to prevent double-scoring, but the comparison in it is backwards. Since the player's score starts at zero and never rises (no coins can ever be collected), the guard is permanently false and no coin is ever picked up.

### Bug 2: Jumping up through platforms

Solid platforms are supposed to block the player from every direction, but they only do so from the sides and from above. If the player jumps up into the underside of a platform, they pass straight through it and can land on top. The vertical collision code stops the player's upward motion when they bump their head, but forgets to push them back out of the platform first, so they end up overlapping it and pop out on top. A cunning player can even use this as a shortcut to reach high platforms.

### Bug 3: The flag wins the game on its own

The stated rule is "collect all coins, *then* touch the flag to win." The actual code checks for either condition instead of both. As a result, touching the flag wins immediately no matter how many coins you have, including zero. The game can be "won" with a coinless speedrun straight to the flag.

## Example issue: coin collection bug

Use this text when creating an issue that `fullsend` should pick up and resolve:

> **Title:** Coins can never be collected
>
> **Description:** When the player walks over or stands on a coin, the coin is
> not collected. The coin counter stays at 0/6 no matter what, making the game
> unwinnable.
>
> **Steps to reproduce:**
> 1. Run `uv run platformer.py`
> 2. Walk to any coin and stand on it
> 3. Observe that the coin is not collected and the counter remains 0/6
>
> **Expected behavior:** Standing on a coin should collect it and increment
> the coin counter.
>
> **Actual behavior:** Nothing happens; the coin remains on screen.
>
> **Suspected cause:** The coin pickup logic in the main game loop includes a
> guard that was meant to prevent coins from being scored twice, but its
> comparison appears to be backwards. The player's score starts at zero and
> can never rise while coins can't be collected, so the guard is always false
> and pickup never happens. The fix should correct the guard so overlapping a
> coin collects it, while still preventing the same coin from being scored
> twice.

Once the issue is created, `fullsend` will pick it up and run it through the development lifecycle. After implementing the fix, re-run the game and confirm coins are collected properly.

## Your turn: two more issues

Bug 1 is handled. Now it's your turn! Bugs 2 and 3 are still in the code:

- **Bug 2** lets the player jump up through the underside of platforms.
- **Bug 3** lets the player win instantly by touching the flag with zero coins.

Following the format of the example issue above (title, description, steps to reproduce, expected vs. actual behavior, and a suspected cause), create one issue for each bug. Write them as if you *didn't* have the spoiler section: describe what you observed while playing, then dig into the code to pin down the cause. Hand each issue to fullsend, verify the fixes by playing, and see whether the game finally becomes winnable the honest way: all 6 coins, then the flag.
