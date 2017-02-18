---
layout: post
title: "Redux Without A Framework"
date: 2017-02-15 07:03:35
image: '/assets/img/'
description: 'How to use Redux without using a framework'
tags:
- javascript
- redux
categories:
- Web Development
---

Up until now, any Javascript development that I've worked on involving Redux
has also involved using React. In fact, most of the Redux examples that
I've seen on the web also use some sort of front end framework or library
(mostly React and Angular).

Over the past few days I've built a simple version of the game Tic Tac Toe using
Redux and plain ES6. In this post, I'm going to go through some of the steps I
took to create my game. If tutorials aren't really your thing and you'd rather
just see the what the final code looks like, you can check out my project for
this app on <a href="https://github.com/scottmcallister/es6-redux-tic-tac-toe" target="_blank"> Github</a>.


## Setting up ##

First, let's create a new directory to put our app in and install some NPM
packages.

<pre>
<code>$ mkdir tic-tac-toe; cd tic-tac-toe
$ npm init
$ npm install --save-dev webpack webpack-dev-server babel-core babel-loader babel-preset-es2015 path</code>
</pre>

At this point you should see a package.json file and a node_modules directory
inside your project folder. Next we'll add a webpack config file so we can
transpile our ES6 code to ES5 using webpack and babel.

<div class="code-description">
  <p>./webpack.config.js</p>
</div>
<pre>
<code class="language-javascript">var path = require('path');
var webpack = require('webpack');

module.exports = {
    entry: './js/index.js',
    output: {
        filename: './dist/bundle.js'
    },
    module: {
        loaders: [
            {
                loader: 'babel-loader',  /* eslint-disable */
                test: path.join(__dirname, 'js'), /* eslint-enable */
                query: {
                    presets: 'es2015'
                }
            }
        ]
    },
    plugins: [
        // Avoid publishing files when compilation fails
        new webpack.NoEmitOnErrorsPlugin()
    ],
    stats: {
        // Nice colored output
        colors: true
    },
    // Create Sourcemaps for the bundle
    devtool: 'source-map'
};</code>
</pre>

Basically this file is telling webpack to transpile any Javascript files in the
'js' folder and put the transpiled code into './dist/bundle.js'. There's two
comments I've included here that will disable linting warnings for the
'__dirname' variable. If you aren't using <a href="http://eslint.org/" target="_blank">ESlint</a> currently, I'd highly
recommend trying it out. I won't include instructions here though since it's not really required for this app to work.

Now let's add an HTML document and some JS code to our project.

<div class="code-description">
  <p>./index.html</p>
</div>
<pre>
<code class="language-html">&lt;head>
  &lt;title>Tic Tac Toe&lt;/title>
&lt;/head>

&lt;body>
&lt;script src="dist/bundle.js">&lt;/script>
&lt;/body></code>
</pre>

<div class="code-description">
  <p>./js/index.js</p>
</div>
<pre>
<code class="language-javascript">const name = 'world';
const message = `Hello ${name}!`;
console.log(message);</code>
</pre>

At this point we should be able to run our app using webpack-dev-server. There
isn't any content yet, but we should still see "Hello world!" printed to the
console.

<pre>
<code>$ webpack-dev-server --inline --hot</code>
</pre>

I added an NPM script to the package.json file in my app to run this command
with less typing, but doing this is totally optional.

<div class="code-description">
  <p>./package.json</p>
</div>
<pre>
<code class="language-javascript">{
    // name, version, etc...
    "scripts": {
        "dev": "webpack-dev-server --inline --hot"
    },
    // ...
}</code>
</pre>

## Adding Redux ##

Now that we have webpack set up and we're able to run ES6 code in the browser,
let's start actually adding Redux to the project. First, we'll need to install
redux as a dependency:

<pre>
<code>$ npm install --save redux</code>
</pre>

Next, we'll initialize a redux store and write a simple reducer with an initial
state for our game. We'll also print the state to the console after the DOM
loads and include some middleware to enable <a href="https://github.com/zalmoxisus/redux-devtools-extension" target="_blank">Redux DevTools</a>.

<div class="code-description">
  <p>./js/index.js</p>
</div>
<pre>
<code class="language-javascript">import { createStore } from 'redux';
import reducer from './reducer';

const store = createStore(
    reducer,
    window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()
);

console.log(store.getState())</code>
</pre>

<div class="code-description">
  <p>./js/reducer.js</p>
</div>
<pre>
<code class="language-javascript">const initialState = {
    xTurn: false,
    grid: [
        ['','',''],
        ['','',''],
        ['','','']
    ],
    gameOver: false,
    winner: '',
    title: 'O Turn'
};

// reducer
const reducer = (state=initialState, action) => {
    switch(action.type) {
    default:
        return state;
    }
};

export default reducer;</code>
</pre>

Now that we have a Redux store to play with, we can write actions for the
game logic and their corresponding cases to be handled in the reducer.

<div class="code-description">
  <p>./js/reducer.js</p>
</div>
<pre>
<code class="language-javascript">export const RESET_GAME = 'RESET_GAME';
export const SET_TITLE = 'SET_TITLE';
export const SET_GRID = 'SET_GRID';
export const MAKE_MOVE = 'MAKE_MOVE';
export const END_GAME = 'END_GAME';

/**
 * resetGame - load the initial game state
 */
export const resetGame = () => {
    return {
        type: RESET_GAME
    };
};

/**
 * setTitle - set new game info above game grid
 * @param {String} newTitle
 */
export const setTitle = (newTitle) => {
    return {
        type: SET_TITLE,
        title: newTitle
    };
};

/**
 * setGrid - update the contents of the game grid
 * @param {Array} newGrid - a 2 dimensional 3x3 array
 */
export const setGrid = (newGrid) => {
    return {
        type: SET_GRID,
        title: newGrid
    };
};

/**
 * move - performs a move on behalf of the current player
 * @param  {Number} xIndex - grid index on x axis
 * @param  {Number} yIndex - grid index on y axis
 * @param  {String} player - player letter ('X' or 'O')
 */
export const move = (row, col, player) => {
    return {
        type: MAKE_MOVE,
        row,
        col,
        player
    };
};

/**
 * endGame
 * @param  {String} winner - String for the winning player (or a tie)
 */
export const endGame = (winner) => {
    return {
        type: END_GAME,
        winner
    };
};</code>
</pre>

<div class="code-description">
  <p>./js/reducer.js</p>
</div>
<pre>
<code class="language-javascript">import {
    RESET_GAME,
    SET_TITLE,
    SET_GRID,
    MAKE_MOVE,
    END_GAME
} from './actions';

const initialState = {
    xTurn: false,
    grid: [
        ['','',''],
        ['','',''],
        ['','','']
    ],
    gameOver: false,
    winner: '',
    title: 'O Turn'
};

// reducer
const reducer = (state=initialState, action) => {
    switch(action.type) {
    case RESET_GAME:
        return Object.assign({}, state, {
            grid: [
                ['','',''],
                ['','',''],
                ['','','']
            ],
            gameOver: false,
            winner: '',
            title: `${state.xTurn ? 'X' : 'O'} Turn`
        });
    case SET_TITLE:
        return Object.assign({}, state, {
            title: action.title
        });
    case SET_GRID:
        return Object.assign({}, state, {
            grid: action.grid
        });
    case MAKE_MOVE: {
        const box = state.grid[action.row][action.col];
        const nextTurnIsX = (state.xTurn && box !== '')
            || (!state.xTurn && box === '');
        let newGrid = state.grid;
        newGrid[action.row][action.col] = box === '' ?
            action.player : box;
        return Object.assign({}, state, {
            xTurn: nextTurnIsX,
            grid: newGrid,
            title: `${nextTurnIsX ? 'X' : 'O'} Turn`
        });
    }
    case END_GAME: {
        const newTitle = action.winner === 'tie' ?
            'Tie Game!' : `Player ${action.winner} Wins!`;
        return Object.assign({}, state, {
            winner: action.winner,
            gameOver: true,
            title: newTitle
        });
    }
    default:
        return state;
    }
};

export default reducer;</code>
</pre>

We can't see anything yet in the UI, but we can still test what we have so far
by opening Redux dev tools in our browser and dispatching the action's that have
been written.

## Adding the UI ##

After putting the pieces of our game together in Redux, we'll need to display
the game grid and allow users to dispatch the actions we've written. First let's
update our HTML document to add in the elements we need for the game grid.

<div class="code-description">
  <p>./index.html</p>
</div>
<pre>
<code class="language-html">&lt;!DOCTYPE html>
&lt;html>
&lt;head>
    &lt;meta charset="UTF-8">
    &lt;title>Tic Tac Toe&lt;/title>
    &lt;link rel="stylesheet" type="text/css" href="/css/main.css">
&lt;/head>
&lt;body>
    &lt;div id="game">
        &lt;h1 class="title">&lt;/h1>
        &lt;div class="wrapper">
            &lt;table class="grid">
                &lt;tbody>
                    &lt;tr>
                        &lt;td>&lt;button>&lt;/button>&lt;/td>
                        &lt;td>&lt;button>&lt;/button>&lt;/td>
                        &lt;td>&lt;button>&lt;/button>&lt;/td>
                    &lt;/tr>
                    &lt;tr>
                        &lt;td>&lt;button>&lt;/button>&lt;/td>
                        &lt;td>&lt;button>&lt;/button>&lt;/td>
                        &lt;td>&lt;button>&lt;/button>&lt;/td>
                    &lt;/tr>
                    &lt;tr>
                        &lt;td>&lt;button>&lt;/button>&lt;/td>
                        &lt;td>&lt;button>&lt;/button>&lt;/td>
                        &lt;td>&lt;button>&lt;/button>&lt;/td>
                    &lt;/tr>
                &lt;/tbody>
            &lt;/table>
        &lt;/div>
        &lt;button id="reset">Start Over&lt;/button>
    &lt;/div>
    &lt;script src="dist/bundle.js">&lt;/script>
&lt;/body>
&lt;/html></code>
</pre>

You might have noticed we've added a stylesheet to our document in the head tag.
Here are the styles I wrote in my version, but feel free to modify these as much
as you like.

<div class="code-description">
  <p>./css/main.css</p>
</div>
<pre>
<code class="language-css">body {
    background-color: #f6d68f;
    text-align: center;
    font-family: verdana;
    color: #ffffff;
}

.title {
    width: 100%;
}

.wrapper {
    margin: auto;
    width: 300px;
}

.grid {
    margin-top: 70px;
    width: 100%;
    background-color: #f1a90d;
    border-radius: 5px;
    border: 5px solid #f1a90d;
}

tbody, tr, td {
    background-color: #f9c658;
    width: 30%;
    height: 100px;
}

tbody {
}

tr, td {
    border: 1px solid #f1a90d;
}

#reset {
    color: #ffffff;
    background-color: #00c0ff;
    padding: 10px;
    font-size: 25px;
    border: 2px solid #ffffff;
    border-radius: 3px;
    margin-top: 30px;
}

#reset:hover {
    background-color: #06a0d2;
}

td button {
    color: #ffffff;
    width: 100%;
    height: 100%;
    background: none;
    border: none;
    font-size: 60px;
}</code>
</pre>

In order to dispatch Redux actions through the UI and update the game state,
we'll need to bind click events to our buttons. We'll be writing an ES6 class
to handle the bindings and game logic. Add a file in the 'js' directory
called 'game.js'.

<div class="code-description">
  <p>./js/game.js</p>
</div>
<pre>
<code class="language-javascript">import * as actions from './actions';

// all possible winning coordinates for each box in the grid
const winConditions = [
    [
        [[0,0],[0,1],[0,2]],
        [[0,0],[1,1],[2,2]],
        [[0,0],[1,0],[2,0]]
    ],
    [
        [[0,0],[0,1],[0,2]],
        [[0,1],[1,1],[2,1]]
    ],
    [
        [[0,0],[0,1],[0,2]],
        [[2,0],[1,1],[0,2]],
        [[2,2],[1,2],[0,2]]
    ],
    [
        [[0,0],[1,0],[2,0]],
        [[1,0],[1,1],[1,2]]
    ],
    [
        [[0,0],[1,1],[2,2]],
        [[1,0],[1,1],[1,2]],
        [[0,1],[1,1],[2,1]],
        [[2,0],[1,1],[0,2]]
    ],
    [
        [[0,2],[1,2],[2,2]],
        [[1,0],[1,1],[1,2]]
    ],
    [
        [[2,2],[2,1],[2,0]],
        [[0,2],[1,1],[2,0]],
        [[0,0],[1,0],[2,0]]
    ],
    [
        [[0,1],[1,1],[2,1]],
        [[2,0],[2,1],[2,2]]
    ],
    [
        [[2,0],[2,1],[2,2]],
        [[0,0],[1,1],[2,2]],
        [[0,2],[1,2],[2,2]]
    ]
];

class Game {
    constructor(options) {
        this.ui = options.ui;
        this.store = options.store;
        this.store.subscribe(this.update.bind(this));
        this.ui
            .querySelector('#reset')
            .addEventListener('click',
                this.resetGame.bind(this)
            );
        const boxes = this.ui.querySelectorAll('td button');
        boxes.forEach((box, index) => {
            const row = Math.floor(index/3);
            const col = index % 3;
            box.addEventListener('click',
                this.makeMove.bind(this, row, col)
            );
        });
    }

    renderTitle(state) {
        this.ui
            .querySelector('.title')
            .innerHTML = state.title;
    }

    renderGrid(state) {
        const { grid } = state;
        const boxes = this.ui.querySelectorAll('td button');
        boxes.forEach((box, index) => {
            const value = grid[Math.floor(index/3)][index % 3];
            box.innerHTML = value;
        });
    }

    isWinner(row, col, player) {
        const state = this.store.getState();
        let gameOver = false;
        const boxIndex = row * 3 + col;
        winConditions[boxIndex].map((wc) => {

            // check if win condition coordinates match player
            if (state.grid[wc[0][0]][wc[0][1]] === player &&
                state.grid[wc[1][0]][wc[1][1]] === player &&
                state.grid[wc[2][0]][wc[2][1]] === player) {
                gameOver = true;
            }

        });
        return gameOver;
    }

    isTie() {
        const state = this.store.getState();
        let result = true;
        state.grid.forEach((row) => {
            row.forEach((column) => {
                if(column === '') {
                    result = false;
                }
            });
        });
        return result;
    }

    makeMove(row, col) {
        const player = this.store.getState().xTurn ? 'X' : 'O';
        if(this.store.getState().gameOver) { return; }
        this.store.dispatch(actions.move(row, col, player));
        if(this.isWinner(row, col, player)) {
            this.store.dispatch(actions.endGame(player));
        } else if(this.isTie()) {
            this.store.dispatch(actions.endGame('tie'));
        }
    }

    resetGame() {
        this.store.dispatch(actions.resetGame());
    }

    update() {
        /* eslint-disable */
        const state = this.store.getState();
        this.renderTitle(state);
        this.renderGrid(state);
    }

    render() {
        this.update();
    }
}

export default Game;</code>
</pre>

It's worth noting that the huge ugly "winConditions" variable at the top of the
file is included for performance reasons. Since each box in the game's grid can
only be a part of up to 4 sets of three in a row, we'll only need at most 4
checks to see if the last move resulted in a win. Checking every three in a row
combination after each move is a waste.

Now that we have the game logic completely written, we'll need to update our
'index.js' file to use our new Game class.

<div class="code-description">
  <p>./js/index.js</p>
</div>
<pre>
<code class="language-javascript">import { createStore } from 'redux';
import reducer from './reducer';
import Game from './game';

const store = createStore(
    reducer,
    window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()
);

document.addEventListener('DOMContentLoaded', () => {
    const game = new Game({
        ui: document.querySelector('#game'),
        store: store
    });
    game.render();
});</code>
</pre>

## Testing ##

One of the major benefits of using Redux is how easy it becomes to unit test
your code. Because reducers are just pure functions, each of your tests will
essentially verify that the function produces an expected state for any given
action and current state.

Here's an example of a couple of unit tests I wrote for this app using Mocha:

<div class="code-description">
  <p>./test/test.js</p>
</div>
<pre>
<code class="language-javascript">/* eslint-disable no-undef */
import assert from 'assert';
import reducer from '../js/reducer';

describe('Test reducer', () => {
    it('should switch turns after making a move', () => {
        const initialState = {
            xTurn: false,
            grid: [
                ['','',''],
                ['','',''],
                ['','','']
            ],
            gameOver: false,
            winner: '',
            title: 'O Turn'
        };
        const output = reducer(initialState, {
            type: 'MAKE_MOVE',
            row: 1,
            col: 1,
            player: 'O'
        });
        assert(output.xTurn === true);
    });
    it('should add player symbol to empty square after move', () => {
        const initialState = {
            xTurn: false,
            grid: [
                ['','',''],
                ['','',''],
                ['','','']
            ],
            gameOver: false,
            winner: '',
            title: 'O Turn'
        };
        const output = reducer(initialState, {
            type: 'MAKE_MOVE',
            row: 1,
            col: 1,
            player: 'O'
        });
        assert(output.grid[1][1] === 'O');
    });
    // etc...
});</code>
</pre>

The slightly trickier code to test is the ES6 "Game" class. Because the code is
so tightly coupled to the game's HTML markup, I ended up using "jsdom" to
generate a fake document for testing.

<div class="code-description">
  <p>./test/test.js</p>
</div>
<pre>
<code class="language-javascript">/* eslint-disable no-undef */
import { createStore } from 'redux';
import jsdom from 'mocha-jsdom';
import assert from 'assert';
import reducer from '../js/reducer';
import Game from '../js/game';

const gameHTML = `&lt;div id="game">
    &lt;h1 class="title">&lt;/h1>
    &lt;div class="wrapper">
        &lt;table class="grid">
            &lt;tbody>
                &lt;tr>
                    &lt;td>&lt;button>&lt;/button>&lt;/td>
                    &lt;td>&lt;button>&lt;/button>&lt;/td>
                    &lt;td>&lt;button>&lt;/button>&lt;/td>
                &lt;/tr>
                &lt;tr>
                    &lt;td>&lt;button>&lt;/button>&lt;/td>
                    &lt;td>&lt;button>&lt;/button>&lt;/td>
                    &lt;td>&lt;button>&lt;/button>&lt;/td>
                &lt;/tr>
                &lt;tr>
                    &lt;td>&lt;button>&lt;/button>&lt;/td>
                    &lt;td>&lt;button>&lt;/button>&lt;/td>
                    &lt;td>&lt;button>&lt;/button>&lt;/td>
                &lt;/tr>
            &lt;/tbody>
        &lt;/table>
    &lt;/div>
    &lt;button id="reset">Start Over&lt;/button>
&lt;/div>`;

describe('test game', () => {
    jsdom();

    const store = createStore(
        reducer
    );
    let game;
    let xTurn;

    before(() => {
        document.body.innerHTML = gameHTML;
        const options = {
            ui: document.querySelector('#game'),
            store
        };
        game = new Game(options);
        game.render();
    });

    beforeEach(() => {
        game.resetGame();

        // resetting game does not affect player order!
        xTurn = store.getState().xTurn;
    });

    it('has title', () => {
        const title = document.querySelector('.title').innerHTML;
        assert(title === `${xTurn ? 'X':'O'} Turn`);
    });

    it('should update grid after moves', () => {
        game.makeMove(0, 0);
        const topLeftBox = document.querySelector('td button').innerHTML;
        assert(topLeftBox === xTurn ? 'X':'O');
    });

    // etc...

});
</code>
</pre>

## Wrapping up ##

I realize this wasn't as much of a "step by step list of instructions" as some
of my other tutorials, but hopefully by taking a look at the code you can see
that you don't necessarily need to rely on a JS framework to use Redux. You'll
still get the benefit of being able to go forward and backward between different
application states, simplify your unit testing, and easily debug unexpected
behaviour.
