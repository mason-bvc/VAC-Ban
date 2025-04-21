# VAC-Ban

A Pac-Man clone with a Counter-Strike-flavored twist.

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
- Familiarized self with ragdoll system and physics simulation.

## What's the big idea? Why no abstract classes, inheritance, or interfaces?

You may find that the GameMode blueprint is monolithic and does a lot of heavy
lifting. This is definitely intentional!

- No abstract classes because refactoring them is nightmareish, they're not
  nearly flexible enough for behavior tweaking, and they often turn into glorified
  helpers.
- No inheritance because of point #2 in abstract classes, plus behavior becomes
  non-composable very quickly.
- I've found that doing SOLID design for a project whose gameplay elements
  are all well-accounted for is premature and unnecessary abstraction.
  SOLID is also mostly useful as guidelines for object-oriented software
  that is open for extension and has components talking to it crossing
  DLL boundaries AKA plugins EXCEPT for "D" which-- as you know-- is
  dependency inversion; keeping lower-level modules (guns, enemies, items)
  unaware of each other. More importantly however, unable to dictate the
  rules of the game.
- E.g. "what should happen when the player overlaps with a pick-uppable gun?
  Whose responsibility is it to dictate what happens?
  - Putting the burden on either the player or the gun couples them
    (violating D, possibly S in SOLID).
  - Abstracting them to ICollector and/or ICollectible kicks the can down
    the road; we'll have to define the rules as to what happens in the
    concrete classes that implement such interfaces.
    - You could strategy pattern this, but you still have to deal with keeping
    track of like, 5 to 500 interfaces. What happens when you want to refactor,
    especially in a visual scripting solution?
  - "Why don't we keep ICollector/ICollectible but make them stubs in the
    concretes?" at this point we're just leveraging the type system into a
    glorified tagging system (violating I in SOLID).
- Utilizing interfaces in this way is way more useful in enterprise/business
  code whose needs are "we need to be abstract enough to allow for extension of
  our code". Interfaces have their place in game development, but definitely
  not in these cases. It would be much more useful to use interfaces on the
  _module_ level, not the _object_ level. In fact this is pretty much what most
  good enterprise software does already (source: trust me bro [used to work for
  a CRM+SaaS software firm and yes I wanted to die]).
- To reiterate, I highly believe that the game mode should be monolithic and
  very bossy about what happens in its world. Hardly any rule-related gameplay
  should be dictated in anything lower than the game mode.

  - E.g. (in pseudocode)

  ```
  // contains sensible default implementations for interested game modes
  static class DefaultGameModeImplementationUtils {
      static func TickAllObjectsInWorld(Objects[] objects) {
          ...
      }

      static func OnActorOverlap(Object overlapped, Object overlapper) {
          if overlapper is Player {
              if overlapped is Gun {
                  <pick up gun and attach to player>
              } else if overlapped is bullet {
                  <pick up bullet and add to player ammo count>
              }
          }
      }
  }

  class PacManGameMode implements IBeginPlayer, IThinker, IInputHandler, IOverlapHandler {
      func IBeginPlayer.BeginPlay() {
          ...
      }

      func IThinker.Tick() {
          DefaultGameModeImplementationUtils.TickAllObjectsInWorld(objects)
      }

      func IInputHandler(InputEvent e) {
          ...
      }

      func IOverlapHandler.OnActorOverlap(Object overlapped, Object overlapper) {
          <some behavior specific to this game mode>
          DefaultGameModeImplementationUtils.OnActorOverlap(overlapped, overlapper)
          <ditto>
      }
  }

  class GunsNotAllowedGameMode implements IBeginPlayer, IThinker, IInputHandler, IOverlapHandler {
      func IBeginPlayer.BeginPlay() {
          ...
      }

      func IThinker.Tick() {
          DefaultGameModeImplementationUtils.TickAllObjectsInWorld(objects)
      }

      func IInputHandler(InputEvent e) {
          ...
      }

      func IOverlapHandler.OnActorOverlap(Object overlapped, Object overlapper) {
          if overlapper is Player {
              <notice how there is no case for Gun>
              if overlapped is bullet {
                  <pick up bullet and add to player score count>
              }
          }
      }
  }
  ```

  - Such an example yields very little coupling (even if you count `DefaultGameModeImplementationUtils`, but it's a necessary evil in OOP
    languages) AND enough flexibility to allow game
    modes to define what should happen when an item is picked up without any
    of the lower-level objects to know what should even happen to them.

  - Typical abuse of interfaces and shoehorning them into game architecture:

  ```
  interface ICollectible {
      func TryCollect(ICollector collector) returns bool
  }

  interface ICollector {
      func TryCollect(ICollectible collectible) returns bool
  }

  class Player implements ICollector {
      implement func TryCollect(ICollectible collectible) returns bool {
          if collectible is Gun {
              // ^ rather contrived, but this is coupling right here
              <some behavior>
          }

          // OR:

          collectible.Destroy()
          // ^ dictating game rules. also, what if this ICollectible destroys
          // itself already?
      }
  }

  class Gun implements ICollectible {
      implement func TryCollect(ICollector collector) returns bool {
          if collector is Player {
              // ^ coupling again
              <some behavior>
          }

          // OR:

          Destroy()
          // ^ dictating game rules again.
      }
  }
  ```

  - Not only must you deal with all the confusing responsibility stuff, you
    also have to deal with potentially a huge amount of interfaces, inflating
    the codebase and just obfuscating the code. I strongly adhere to KISS.
    When it comes to high level, "dumb" code is almost always better to read,
    especially when the surface area of it becomes so huge with node-based
    visual scripting solutions.

  - "Wait, isn't that utils class in example #1 just begging for boilerplate?"
    yes, but you are free to start abstracting behavior at the module level
    using standard architectural patterns (like strategy pattern).
