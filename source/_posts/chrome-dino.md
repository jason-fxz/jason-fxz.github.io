---
title: 'chrome小恐龙外挂'
date: 2020-06-01 21:49:08
tags: []
categories:
- 杂
---

引用自：[link](https://blog.csdn.net/yuanwow/article/details/103950597)

浏览器调试console里输入
```js
function TrexRunnerBot() {
  const makeKeyArgs = (keyCode) => {
    const preventDefault = () => void 0;
    return {keyCode, preventDefault};
  };
  const upKeyArgs = makeKeyArgs(38);
  const downKeyArgs = makeKeyArgs(40);
  const startArgs = makeKeyArgs(32);
  if (!Runner().playing) {
    Runner().onKeyDown(startArgs);
    setTimeout(() => {
      Runner().onKeyUp(startArgs);
    }, 500);
  }
  function conquerTheGame() {
    if (!Runner || !Runner().horizon.obstacles[0]) return;
    const obstacle = Runner().horizon.obstacles[0];
    if (obstacle.typeConfig && obstacle.typeConfig.type === 'SNACK') return;
    if (needsToTackle(obstacle) && closeEnoughToTackle(obstacle)) tackle(obstacle);
  }
  function needsToTackle(obstacle) {
    return obstacle.yPos !== 50;
  }
  function closeEnoughToTackle(obstacle) {
    return obstacle.xPos <= Runner().currentSpeed * 18;
  }
  function tackle(obstacle) {
    if (isDuckable(obstacle)) {
      duck();
    } else {
      jumpOver(obstacle);
    }
  }
  function isDuckable(obstacle) {
    return obstacle.yPos === 50;
  }
  function duck() {
    Runner().onKeyDown(downKeyArgs);
    setTimeout(() => {
      Runner().onKeyUp(downKeyArgs);
    }, 500);
  }
  function jumpOver(obstacle) {
    if (isNextObstacleCloseTo(obstacle))
      jumpFast();
    else
      Runner().onKeyDown(upKeyArgs);
  }
  function isNextObstacleCloseTo(currentObstacle) {
    const nextObstacle = Runner().horizon.obstacles[1];
 
    return nextObstacle && nextObstacle.xPos - currentObstacle.xPos <= Runner().currentSpeed * 42;
  }
  function jumpFast() {
    Runner().onKeyDown(upKeyArgs);
    Runner().onKeyUp(upKeyArgs);
  }
  return {conquerTheGame: conquerTheGame};
}
let bot = TrexRunnerBot();
let botInterval = setInterval(bot.conquerTheGame, 2);
```