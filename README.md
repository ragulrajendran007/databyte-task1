# databyte-task1
website link:  http://127.0.0.1:5500/databyte.html

html:
<!DOCTYPE html>
<html lang="en">
<head>
   
    <title>Memory Game with JavaScript</title>
    <link rel="stylesheet" href="databyte.css">
    <script src="databyte.js" defer></script>
</head>
<body>
    <div class="game">
        <div class="controls">
            <button>Start</button>
            <div class="stats">
                <div class="moves">0 moves</div>
                <div class="timer">Time: 0 sec</div>
            </div>
        </div>
        <div class="board-container">
            <div class="board" data-dimension="4"></div>
            <div class="win">You won!</div>
        </div>
    </div>
</body>
</html>

css:
html{
    height:100%;
    width:100%;
    background:linear-gradient(325deg, black 0% , red 30%,yellow  70%, white 100%);;
    font-family:Arial, Helvetica, sans-serif;
    overflow:  hidden ;
}
.game {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}
.controls {
    display: flex;
    gap: 20px;
    margin-bottom: 20px;
}
button {
    background: #282A3A;
    color:white;
    border-radius: 5px;
    padding: 10px 20px;
    border: 0;
    cursor: pointer;
    font-family: Arial, Helvetica, sans-serif;
    font-size: 18pt;
    font-weight: bold;
}
.disabled {
    color: grey;
}
.stats {
    color:black;
    font-size: 14pt;
    font-weight: bold;
}
.board-container {
    position: relative;
}
.board,
.win {
    border-radius: 5px;
    box-shadow: 0 25px 50px rgb(33 33 33 / 25%);
    background: linear-gradient(135deg,  #03001e 0%,#c00303 0%,#f3f30b 50%, #fdeff9 100%);
    transition: transform .6s cubic-bezier(0.4, 0.0, 0.2, 1);
    backface-visibility: hidden;
}
.board {
    padding: 20px;
    display: grid;
    grid-template-columns: repeat(4, auto);
    grid-gap: 20px;
}
.board-container.flipped .board {
    transform: rotateY(180deg) rotateZ(50deg);
}
.board-container.flipped .win {
    transform: rotateY(0) rotateZ(0);
}
.card {
    position: relative;
    width: 100px;
    height: 100px;
    cursor: pointer;
}
.card-front,
.card-back {
    position: absolute;
    border-radius: 5px;
    width: 100%;
    height: 100%;
    background: #282A3A;
    transition: transform .6s cubic-bezier(0.4, 0.0, 0.2, 1);
    backface-visibility: hidden;
}
.card-back {
    transform: rotateY(180deg) rotateZ(50deg);
    font-size: 28pt;
    user-select: none;
    text-align: center;
    line-height: 100px;
    background: #FDF8E6;
}
.card.flipped .card-front {
    transform: rotateY(180deg) rotateZ(50deg);
}
.card.flipped .card-back {
    transform: rotateY(0) rotateZ(0);
}
.win {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    text-align: center;
    background: #FDF8E6;
    transform: rotateY(180deg) rotateZ(50deg);
}
.win-text {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    font-size: 21pt;
    color: #282A3A;
}
.highlight {
    color: #0b0510;
}


script:

const selectors = {
    boardContainer: document.querySelector('.board-container'),
    board: document.querySelector('.board'),
    moves: document.querySelector('.moves'),
    timer: document.querySelector('.timer'),
    start: document.querySelector('button'),
    win: document.querySelector('.win')
}

const state = {
    gameStarted: false,
    flippedCards: 0,
    totalFlips: 0,
    totalTime: 0,
    loop: null
}

const shuffle = array => {
    const clonedArray = [...array]

    for (let i = clonedArray.length - 1; i > 0; i--) {
        const randomIndex = Math.floor(Math.random() * (i + 1))
        const original = clonedArray[i]

        clonedArray[i] = clonedArray[randomIndex]
        clonedArray[randomIndex] = original
    }

    return clonedArray
}

const pickRandom = (array, items) => {
    const clonedArray = [...array]
    const randomPicks = []

    for (let i = 0; i < items; i++) {
        const randomIndex = Math.floor(Math.random() * clonedArray.length)
        
        randomPicks.push(clonedArray[randomIndex])
        clonedArray.splice(randomIndex, 1)
    }

    return randomPicks
}

const generateGame = () => {
    const dimensions = selectors.board.getAttribute('data-dimension')  

    if (dimensions % 2 !== 0) {
        throw new Error("The dimension of the board must be an even number.")
    }

    const emojis = ['🥔', '🍒', '🥑', '🌽', '🥕', '🍇', '🍉', '🍌', '🥭', '🍍']
    const picks = pickRandom(emojis, (dimensions * dimensions) / 2) 
    const items = shuffle([...picks, ...picks])
    const cards = `
        <div class="board" style="grid-template-columns: repeat(${dimensions}, auto)">
            ${items.map(item => `
                <div class="card">
                    <div class="card-front"></div>
                    <div class="card-back">${item}</div>
                </div>
            `).join('')}
       </div>
    `
    
    const parser = new DOMParser().parseFromString(cards, 'text/html')

    selectors.board.replaceWith(parser.querySelector('.board'))
}

const startGame = () => {
    state.gameStarted = true
    selectors.start.classList.add('disabled')

    state.loop = setInterval(() => {
        state.totalTime++

        selectors.moves.innerText = `${state.totalFlips} moves`
        selectors.timer.innerText = `Time: ${state.totalTime} sec`
    }, 1000)
}

const flipBackCards = () => {
    document.querySelectorAll('.card:not(.matched)').forEach(card => {
        card.classList.remove('flipped')
    })

    state.flippedCards = 0
}

const flipCard = card => {
    state.flippedCards++
    state.totalFlips++

    if (!state.gameStarted) {
        startGame()
    }

    if (state.flippedCards <= 2) {
        card.classList.add('flipped')
    }

    if (state.flippedCards === 2) {
        const flippedCards = document.querySelectorAll('.flipped:not(.matched)')

        if (flippedCards[0].innerText === flippedCards[1].innerText) {
            flippedCards[0].classList.add('matched')
            flippedCards[1].classList.add('matched')
        }

        setTimeout(() => {
            flipBackCards()
        }, 1000)
    }
    if (!document.querySelectorAll('.card:not(.flipped)').length) {
        setTimeout(() => {
            selectors.boardContainer.classList.add('flipped')
            selectors.win.innerHTML = `
                <span class="win-text">
                    You won!<br />
                    with <span class="highlight">${state.totalFlips}</span> moves<br />
                    under <span class="highlight">${state.totalTime}</span> seconds
                </span>
            `

            clearInterval(state.loop)
        }, 1000)
    }
}

const attachEventListeners = () => {
    document.addEventListener('click', event => {
        const eventTarget = event.target
        const eventParent = eventTarget.parentElement

        if (eventTarget.className.includes('card') && !eventParent.className.includes('flipped')) {
            flipCard(eventParent)
        } else if (eventTarget.nodeName === 'BUTTON' && !eventTarget.className.includes('disabled')) {
            startGame()
        }
    })
}

generateGame()
attachEventListeners()

