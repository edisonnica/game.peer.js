# game.peer.js

## What is game.peer.js
game.peer.js is a Peer to Peer (#P2P), Serverless, Zero Trust, Distributed, Cooperative, Javascript Game Framework.

## Motivation
When playing a game online, is the game server to be trusted? How do we know the server does not cheat? While dealing bad card might not be a problem in a game like Uno, bad cards distributed by a malicious server can have material implications for Poker or Blackjack players.

## Solution
We suggest a Serverless, Zero Trust, Distrubuted solution, that uses SHA256 and/or Cryptography to manage games. There are 2 building block ideas to ensure a fair, competitive game:
- First, for players private moves, the players will first present their signed state/moves, and once all players shared their signed state/moves, they provide the actual state/moves.
- Second, for shared private state, e.g. like a deck of card, all players will contribute at encrypting and building the state.

This solution does not require a server, each player will cooperate with the rest of players. Poker and Blackjack are perfect candidates for an implementation, since it will remove from the system the need of a server, which, especially for Blackjack, could malitiosly distribute bad cards, with no way for players to enforce no-server-chating. 

## Classes of games targeted
We are targeting a few classes of games:
 - Games with initial setup that is hidden to some/most players (e.g. Poker, Battleships)
 - Games with adversary, concurrent moves (e.g. Rock-Paper-Scissors) 
 - Games that distribute cards that are hidden to all players in the beginning (e.g. Blackjack)
 
## Design and Principles

### Assumption
Games are played in a serveless environment, without a central server. We can use for that an implementation on peerjs for example (normally it still needs a central server to serve the initial App State, but there are solutions in the works that will not require that).

### Concurrent Adversarial Moves (Rock-Paper-Scissors)
In a game like Rock-Paper-Scissors, one of the players is to trust the other player. Without a third party to ensure both players made their moves, there is no way of knowing if a player waited until it read the other player's move, not much different than kids that wait a fraction of a second to show their hand after the other kid.

#### Proposed solution
Two phase disclosure.
- Phase 1: both players release encrypted selections (signatures of selection). This can be as simple as sha256(selection + random string), or encryption with private key.
- Phase 2: once the players received the opponent's encrypted solution, they will release the actual move, along secret used to decrypt the encrypted selection from Phase one. If sha256 is used, there is no need for any secret to be sent, as the opponent can use sha256 for the selection + random string, which should result in same hash.

### User selected initial board
In a game like Battleship, users will create their own boards, that they will be used to play against the opponent. Without a third party that holds the initial boards, there is no way to know if an unscrupulous player will move their ships during game.

#### Proposed solution
Before the game starts, both players will release the encrypted or securely hashed board state. At the end of the game (at least) the winner will have to release the full board state, which the other player can check against each move and the initial hashed/encrypted state.

### Distributing hidden cards
In a game like Poker or Blackjack, most of the cards are hidden to all players, and some players have cards, that are known only to them

#### Shuffling a deck of cards
- Phase 1: each player creates a random sequence of numbers, large enough to be used during game, or an initial state (seed) of a cryptographically secure random number generator.
- Phase 2: each player publishes signatures of their random numbers
- Phase 3: Player 1, shuffles the sorted deck of cards according to their random generators, using shuffle algorithm, known to everyone, then encloses each card in a signed envelope. The envelope will be marked with the player and a unique note on the card, known only by this player, it can be as simple as a card index. Then, passes the deck of envelopes to Player 2.
- Phase 3.x: Player 2 to N, will do the same thing (shuffles the deck of envelopes, and encloses each envelope in a new envelope, again, marked similarly) then passes them to the next player, until all envelopes are back to Player 1.
- Phase 4: envelopes are distributed to players according to game rules, or are placed on the board for everyone to see.

Unwrapping envelopes: each player will (publicly) ask the player that wrapped the card, for the secret to unwrap the envelope. The secret is sent only to player holding the envelope. It will continue to do so, until it unwraps all envelopes, discovering the card.

For the cards that are placed on the board, visible to everyone, everyone will distribute the secrets to everyone, according to the marking on the envelope.

At the end of the game, all players must disclose the initial random seed/numbers, and each player can check the initial signature/hash, and replay the shuffling along with the game operations, ensuring that no cheating took place.

## Posible implementation
Using peerjs, we can create standard classes, like InitialState, Move, Deck, ... that will internally manage the communication protocol between players, along with ensuring no cheating.

## Playing for Money
An Escrow Service, which could be the server delivering the Web App, but that is not necessary, will manage the money on the table. At the end, the winner(s) will share the full log of the game, and they will receive the Money, minus the Escrow fee. At each move, the players will provide proof of escrow, e.g. a signed token or somethign similar, to prove they put money on the table.
