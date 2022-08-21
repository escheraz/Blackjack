# A Simple Online Blackjack Game

Built with Svelte in Typescript. Card images taken from Wikipedia.

Current Default Rules:
- Dealer stands on S17
- Blackjack pays 3:2
- Blackjack tie pushes
- 6 Decks, Continuous Shuffling (+0.02% Edge)
- Late Surrender (+0.07% Edge)
- No splitting (-0.57% Edge)

Edges are obtained from [Wizard of Odds](https://wizardofodds.com/games/blackjack/rule-variations/), relative to a 8-deck, 3:2 blackjack ruleset which has 0.43% House Edge. Thus, this current implementation has 0.91% House Edge when played optimally.
