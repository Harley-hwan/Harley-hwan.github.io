---
layout: post
title: (C#) 41. 미로 알고리즘 (PlayerMoving)
subtitle: 플레이어 이동
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c#, unity, maze, algorithm, player]
comments: true
---

# 미로에서 플레이어의 이동 구현

- 최초 작성일: 2021년 3월 30일(수)

## 목차

[TOC]

## 내용

### Player 이동 구현

앞서 미로를 생성했으면, 이제 그 미로를 탈출할 플레이어를 생성해준다.

그리고 x, y 좌표를 설정할 PosX, PosY를 생성해주고, 

상/하/좌/우 좌표 이동에 대한 로직을 구현해준다.

이때 상하좌우 이동은 random Value에 의해 결정되며, 즉 랜덤으로 미로를 찾아가는 과정이 진행된다. 

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Maze
{
    class Player
    {
        public int PosX { get; private set; }   // Player 좌표는 Player만 고칠 수 있다.
        public int PosY { get; private set; }

        Random _random = new Random();
        Board _board;

        public void Initialize(int posY, int posX, int destY, int destX, Board board)
        {
            PosX = posX;
            PosY = posY;
            _board = board;
        }

        const int MOVE_TICK = 10;
        int _sumTick = 0;

        public void Update(int deltaTick)
        {
            _sumTick += deltaTick;
            if (_sumTick >= MOVE_TICK)
            {
                _sumTick = 0;

                //여기에다가 0.1초마다 실횡될 로직을 넣어준다.
                int randValue = _random.Next(0, 4);
                switch (randValue)
                {
                    case 0:     // 상
                        if (PosY - 1 >= 0 && _board.Tile[PosY - 1, PosX] == Board.TileType.Empty)
                            PosY--;
                        break;
                              
                    case 1:     // 하
                        if (PosY + 1 < _board.Size && _board.Tile[PosY + 1, PosX] == Board.TileType.Empty)
                            PosY++;
                        break;

                    case 2:     // 좌
                        if (PosX - 1 >= 0 && _board.Tile[PosY, PosX - 1] == Board.TileType.Empty)
                            PosX--;
                        break; 

                    case 3:     // 우
                        if (PosX + 1 < _board.Size && _board.Tile[PosY, PosX + 1] == Board.TileType.Empty)
                            PosX++;
                        break;
                }
            }
        }
    }
}

```

### Result Video

<iframe id="video" width="750" height="500" src="/assets/video/PlayerMoving.mp4" frameborder="0"> </iframe>
