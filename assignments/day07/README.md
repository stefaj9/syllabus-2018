# Lucky21 Game API

## Part 1 - Lucky 21

For the remainder of the course we will be working creating a simple game called
[Lucky 21](https://itunes.apple.com/us/app/lucky-21/id1100111795?mt=8).

## Part 2 - Game Logic

We will now take the itemrepository that we created in week 1 and change it into the
API for our game.\
Change the name of the directory from itemrepository to game-api.\
Now create the following files:\
`game-api/deck.js`:
```javascript
module.exports = () => {
    return [
      '01H', '02H', '03H', '04H', '05H', '06H', '07H', '08H', '09H', '10H', '11H', '12H', '13H', // Hearts
      '01C', '02C', '03C', '04C', '05C', '06C', '07C', '08C', '09C', '10C', '11C', '12C', '13C', // Clubs
      '01D', '02D', '03D', '04D', '05D', '06D', '07D', '08D', '09D', '10D', '11D', '12D', '13D', // Diamonds
      '01S', '02S', '03S', '04S', '05S', '06S', '07S', '08S', '09S', '10S', '11S', '12S', '13S', // Spades
    ];
};
```

`game-api/dealer.js`:
```javascript
module.exports = () => {
    return {
        shuffle: (deck) => {
            for (let i = 0; i < deck.length - 1; i++) {
                const j = Math.floor(Math.random() * (deck.length - i)) + i;
                const card = deck[j];
                const old = deck[i];
                deck[i] = card;
                deck[j] = old;
            }
        },
        draw: (deck) => {
            const card = deck.pop();
            return card;
        },
    };
};
```

`game-api/lucky21.js`:
```javascript
const deckConstructor = require('./deck.js');
const dealerConstructor = require('./dealer.js');

module.exports = () => {
    let deck = deckConstructor();
    let dealer = dealerConstructor();
    dealer.shuffle(deck);
    let card0 = dealer.draw(deck);
    let card1 = dealer.draw(deck);
    let state = {
        deck: deck,
        dealer: dealer,
        cards: [
            card0,
            card1,
        ],
        // The card that the player thinks will exceed 21.
        card: undefined,
    };
    return {
        state: state,
        // Is the game over (true or false).
        isGameOver: (game) => {
            // TODO
        },
        // Has the player won (true or false).
        playerWon: (game) => {
            // TODO
        },
        // The highest score the cards can yield without going over 21 (integer).
        getCardsValue: (game) => {
            // TODO
        },
        // The value of the card that should exceed 21 if it exists (integer or undefined).
        getCardValue: (game) => {
            // TODO
        },
        getTotal: (game) => {
            // TODO
        },
        // The player's cards (array of strings).
        getCards: (game) => {
            // TODO
        },
        // The player's card (string or undefined).
        getCard: (game) => {
            // TODO
        },
        // Player action (void).
        guess21OrUnder: (game) => {
            // TODO
        },
        // Player action (void).
        guessOver21: (game) => {
            // TODO
        },
    };
};
```

Do not implement the `TODO`s for now

## Part 3 - Unit Test Setup

Before we start implementing our game's logic we will create unit tests to validate whether
we are done or not, for this we will use [Jest](https://jestjs.io/).\
Which is a javascript testing framework, that helps us create and run our tests.\
Use `npm` to install the [Jest](https://www.npmjs.com/package/jest) package as a **dev** dependency.

Now lets create a simple test for our game logic inside \
`lucky21.unit-test.js`:
```javascript
const lucky21Constructer = require('./lucky21.js');

test('a new game should have 50 cards left in the deck', () => {
  let game = lucky21Constructer();
  expect(game.deck.length).toEqual(50);
});

test('a new game should have 2 drawn cards', () => {
  let game = lucky21Constructer();
  expect(game.cards.length).toEqual(2);
});
```

Then create another script inside `package.json` that runs Jest on all files ending in `.unit-test.js`.
```bash
npm run test:unit
```

## Part 4 - Dependency Injection

Now we want to create tests where we play an entire game. But we have a problem, the dealer's
shuffle function uses `Math.random` to shuffle the deck, so it is impossible to know what the
player will draw in the test.

To work around this we will use dependency injection, which means the caller will provide
the callee with the dependencies in our case the constructor:\
`game-api/lucky21.js`:
```javascript
module.exports = (deck, dealer) => {
    dealer.shuffle(deck);
    let card0 = dealer.draw(deck);
    let card1 = dealer.draw(deck);
    let state = {
        deck: deck,
        dealer: dealer,
        cards: [
            card0,
            card1,
        ],
        // The card that the player thinks will exceed 21.
        card: undefined,
    };
    return {
        state: state,
        // Is the game over (true or false).
        isGameOver: (game) => {
            // TODO
        },
        // Has the player won (true or false).
        playerWon: (game) => {
            // TODO
        },
        // The highest score the cards can yield without going over 21 (integer).
        getCardsValue: (game) => {
            // TODO
        },
        // The value of the card that should exceed 21 if it exists (integer or undefined).
        getCardValue: (game) => {
            // TODO
        },
        getTotal: (game) => {
            // TODO
        },
        // The player's cards (array of strings).
        getCards: (game) => {
            // TODO
        },
        // The player's card (string or undefined).
        getCard: (game) => {
            // TODO
        },
        // Player action (void).
        guess21OrUnder: (game) => {
            // TODO
        },
        // Player action (void).
        guessOver21: (game) => {
            // TODO
        },
    };
};
```

Notice how the dependencies are sent in as a parameter instead of imported at the top 
of the file. This gives the test (the caller) more control over game's dependencies 
making it easier to test focused parts of the system.

Update the previous tests to use dependency injection.

And lets add another test that tests more of the game logic.

`lucky21.unit-test.js`:
```javascript
const deckConstructor = require('./deck.js');
const dealerConstructor = require('./dealer.js');
const lucky21Constructor = require('./lucky21.js');

test('guess21OrUnder should draw the next card', () => {
  // Arrange
  let deck = deckConstructor();
  deck = [
      '10H', '09S', '01D', '05L',
  ];
  let dealer = dealerConstructor();
  // Override the shuffle to do nothing.
  dealer.shuffle = (deck) => {};
  
  // Inject our dependencies
  let game = lucky21Constructor(deck, dealer);
  
  // Act
  game.guess21OrUnder(game, dealer);
  
  // Assert
  expect(game.cards.length).toEqual(3);
  expect(game.cards[2]).toEqual('01D');
});
```

## Part 5 - Writing Tests

Now write more unit tests for the lucky21 game as we did above, there should be at least
1 test per function and at least 20 unit tests in total.

Run your tests and make sure they are failing:
```bash
npm run test:unit
```

## Part 6 - Implementation

Implement the `TODO`s in the game. Once you are done, all your unit tests should succeed.

## Handin

You should store all the source files in your repository:

```bash
├── game-api
│   ├── app.js
│   ├── deck.js
│   ├── dealer.js
│   ├── lucky21.js
│   ├── lucky21.unit-test.js
│   ├── database.js
│   ├── Dockerfile
│   └── package.json
├── assignments
│   ├── day01
│   │   └── answers.md
│   └── day02
│       └── answers.md
├── scripts
│   ├── initialize_game_api_instance.sh
│   ├── verify_environment.sh
│   ├── docker_compose_up.sh
│   ├── docker_build.sh
│   ├── docker_push.sh
│   └── deploy.sh
├── docker-compose.yml
├── infrastructure.tf
├── Jenkinsfile
└── README.md
```

They must be placed at these location to get full marks.