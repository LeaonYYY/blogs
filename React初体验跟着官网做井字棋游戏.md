# React初体验:跟着官网做井字棋游戏

​		对于初学者来说(至少对于我来说是这样的),当学习一个新的东西的时候如果是循序渐进地慢慢学,那么这次学习之旅大概率是会半途而费的.但是如果能够在刚入门的时候运用基本且常用的语法做出一个简单的demo,就能提升学习这个东西的兴趣并获得小小的成就感,同时对之后的深入学习也能有个大致的方向,不至于如同无头苍蝇一般.

​		这篇文章主要记录了跟着官网学习React的开始: 做一个简单的井字棋游戏

## 从构建九宫格开始

首先画出最简单的一个方格

```jsx
export default function Square() {
  return <button className="square">X</button>;
}
```



目前我们已经得到一个方格了,但九宫格,顾名思义,当然还差八个格子

## 

官网中给出的写法是:

```jsx
export default function Square() {
  return (
    <>
      <div className="board-row">
        <button className="square">1</button>
        <button className="square">2</button>
        <button className="square">3</button>
      </div>
      <div className="board-row">
        <button className="square">4</button>
        <button className="square">5</button>
        <button className="square">6</button>
      </div>
      <div className="board-row">
        <button className="square">7</button>
        <button className="square">8</button>
        <button className="square">9</button>
      </div>
    </>
  );
}
```

但感觉这样感觉代码有点冗长,我们可以利用map来循环这些jsx元素

```jsx
function Square({ value }) {
  return (
  	<button className='square'>{ value }</button>
  )
}
export default function Board() {
  const boardInfo = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8]
  ];
  return (
  	<>
    	{
      boardInfo.map((row, index) => (
        <div key={'row' + index} className='border-row'>
          {
            row.map((item, index) =>
              <Square 
               	key={'col' + index}
                value={ item }
              </Square>  
            )
          }
        </div>
      ))
    }
    </>
  )
}
```

这样, 我们就有了最简单的九宫格板子

## 制作交互式组件

当我们点击九宫格的Square时,将其内容填充为==X==,可以在Board组件中使用state对每个格子的信息进行存储,并定义一个处理点击事件的方法同时传给Square组件

```jsx
import { useState } from 'react'
function Square({ value, onClick }) {
  return (
  	<button className='square' onClick={onClick}>{ value }</button>
  )
}
export default function Board() {
  const [squares, setSquares] = useState(Array(9).fill(null)) // 存储格子信息
  const boardInfo = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8]
  ];
  const handleClick = (index) => {
    if(squares[index]) return // 当squares[index]非空时 返回
  	const temp = squares.slice()
   	temp[index] = 'X'
    setSquares(temp)
  }
  return (
  	<>
    	{
      boardInfo.map((row, index) => (
        <div key={'row' + index} className='border-row'>
          {
            row.map((item, index) =>
              <Square 
               	key={'col' + index}
                value={ squares[item] }
                onClick={()=>{handleClick(item)}}>
              </Square>
            )
          }
        </div>
      ))
    }
    </>
  )
} 
```

这边值得一提的是:在给Square传递带有参数的处理事件时,采用了一个箭头函数内包裹方法调用的形式-->==onClick={( ) => { handleClick(item) }}== 而不是像Vue那样直接写成==onClick="handleClick(item)"== .是因为如果是传递==onClick={handleClick}==时,我们只是将函数本身作为一个prop传递下来,而非要调用它,如果为了添加参数而在后面加上括号时,这个函数就会在渲染的时候进行调用,渲染完成后再点击九宫格是不会有任何反应的.

至此我们已经完成了井字棋游戏的基本构建,接下来则是要考虑游戏规则构建更完整的项目

## 轮流出手

我们可以通过在Board组件中添加一个状态来记录下一次出手的是谁.同时加上胜负规则判定.	

```jsx
  import './App.css'
  import { useState } from 'react'
  function Square({ value, onClick }) {
    return (
      <button className='square' onClick={onClick}>{ value }</button>
    )
  }
  export default function Board() {
    const [isXNext, setIsXNext] = useState(true) // 下一次出手是不是X
    const [squares, setSquares] = useState(Array(9).fill(null)) // 存储格子信息
    const winner = calculateWinner(squares)
    const boardInfo = [
      [0, 1, 2],
      [3, 4, 5],
      [6, 7, 8]
    ];
    const handleClick = (index) => {
      if(squares[index] || winner) return // 当squares[index]非空时或者胜者产生 返回
      const temp = squares.slice()
      temp[index] = isXNext ? 'X' : 'O'
      setSquares(temp)
      setIsXNext(!isXNext)
    }
    return (
      <>
        <div>
        {
          winner ?
          'Winner is :' + winner
          : 'Next Player :' + (isXNext ? 'X' : 'O')
        }
      </div>
        {
        boardInfo.map((row, index) => (
          <div key={'row' + index} className='border-row'>
            {
              row.map((item, index) =>
                <Square 
                  key={'col' + index}
                  value={ squares[item] }
                  onClick={()=>{handleClick(item)}}>
                </Square>
              )
            }
          </div>
        ))
      }
      </>
    )
  }
  /**
 * 检查是否有赢家出现
 * @param {*} squares 九宫格对应的信息
 * @returns 获胜者 
 */
function calculateWinner (squares) {
  const rules = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6]
  ]
  for (let i = 0; i < rules.length; i++) {
    const [a, b, c] = rules[i];
    if(squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a]
    } 
  }
  return null
}


```

## 悔棋(时间旅行)

我们接下来需要做的功能是:让玩家有机会回到之前某一步的棋局状态.

首先我们需要在外层再加入一个Game组件用来进行状态切换管理,并添加一个状态来记录每步的棋局状态.同时我们也不再需要squares来记录我们棋局的状态了,只需要将history的最后一项展示出来.添加一个记录当前为第几步的状态,当我们进行跳步时,只需要改变这个状态.

下面贴出完整的代码

## 完整代码

App.jsx

```jsx
import { useState } from 'react';
import './App.css';
function Square ({ value, handleClick }) {
  return (
    <button className='square' onClick={ handleClick }>
      { value }
    </button>
  )
}
function Board ( { xIsNext, squares, onPlay } ) {
  const winner = calculateWinner(squares) // 获胜者
  const boardInfo = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8]
  ];
  /**
   * 改变对应格的状态
   * @param {*} index 对应格子的编号
   */
  const handleClick = (index) => {
    if(squares[index] || winner) return; // 目标格子不为空或者游戏已经结束 --> 函数返回
    const temp = squares.slice()
    temp[index] = xIsNext ? 'X' : 'O';
    onPlay(temp)
  }
  return (
    <>
    <div>
      {
        winner ?
        'Winner is :' + winner
        : 'Next Player :' + (xIsNext ? 'X' : 'O')
      }
    </div>
    {
      boardInfo.map((row, index) => (
        <div key={'row' + index} className='border-row'>
          {
            row.map((item, index) =>
              <Square key={'col' + index} value={ squares[item] } handleClick={() => {handleClick(item)}}></Square>  
            )
          }
        </div>
      ))
    }
    </>
  )
}

/**
 * 检查是否有赢家出现
 * @param {*} squares 九宫格对应的信息
 * @returns 获胜者 
 */
function calculateWinner (squares) {
  const rules = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6]
  ]
  for (let i = 0; i < rules.length; i++) {
    const [a, b, c] = rules[i];
    if(squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a]
    } 
  }
  return null
}


export default function Game () {
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const [currenMove, setCurrentMove] = useState(0);
  const xIsNext = currenMove % 2 === 0;
  const currentSquare = history[currenMove];
  /**
   * 更新游戏状态
   * @param {*} nextSquares 当前最新的游戏状态
   */
  const handlePlay = (nextSquares) => {
    const nextHistory = [...history.slice(0, currenMove + 1), nextSquares];
    setHistory(nextHistory);
    setCurrentMove(nextHistory.length - 1)
  }

  function jumpTo(nextMove) {
    setCurrentMove(nextMove);
  }

  const moves = history.map((squares, move) => {
    let description = move > 0 ? 'Go to move #' + move : 'Go to game start';
    return (
      <li key={'move' + move}>
        <button onClick={() => jumpTo(move)}>{description}</button>
      </li>
    )
  })

  return (
    <div className='game'>
      <div className='game-board'>
        <Board
          xIsNext = { xIsNext }
          squares = { currentSquare }
          onPlay = { handlePlay }
        ></Board>
      </div>
      <div className='game-history'>
        <ol>{moves}</ol>
      </div>
    </div>
  )
}
```

App.css

```css
.square {
  width: 60px;
  height: 60px;
  border: 0.5px solid black;
  border-radius: 0;
  background-color: #fff;
}

.border-row {
  display: flex;
}
```

