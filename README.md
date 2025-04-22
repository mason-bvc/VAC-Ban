# VAC-Ban

A Pac-Man clone with a Counter-Strike-flavored twist.

## What am I looking at?

This is a parody of Counter-Strike: Source that takes place on a
`cs_office`-like map arranged Pac-Man-wise.

If you aren't well-acquainted with Counter-Strike(: Source) or `cs_office`,
please scrub through the following video:
https://www.youtube.com/watch?v=qpEDhp3kJpo

## "VAC-Ban"? What does that mean?

It's a pun. VAC refers to Valve Anti-Cheat, which is an automated anti-cheating
method implemented by Valve to sniff out and ban hackers from their games. The
system is infamous among Team Fortress 2 and Counter-Strike: Global Offensive
players for its uncompromising swiftness and relatively good accuracy. Because
"VAC-Ban" sounds like "Pac-Man", I decided to go with that.

## Design musings and notable technical exploits

- Decompiled Counter-Strike: Source models, used Blender to export to FBX with
  many manual tweaks.
- Converted Valve-style roughness maps to Unreal PBR, learning its material
  blueprint system in and out (it's fabulous).
- Familiarized self with AI and navigation as we learned in class.
- Familiarized self with Material Instances and the convenient material nodes
  for triplanar mapping (world aligned UV nodes).
- Familiarized self with Animation Montages.
- Familiarized self with Behavior Trees.
  - Enemies run away from you if you have a weapon!
- Familiarized self with ragdoll system and physics simulation.

## How to play:

Roam the maze, collect ammunition (dots), dodge the enemies, find the pistol,
shoot the enemies. Eliminating all four of the enemies means you win!

## Controls

WASD: Move
LMB: Shoot
R: Reload
P: Pause
