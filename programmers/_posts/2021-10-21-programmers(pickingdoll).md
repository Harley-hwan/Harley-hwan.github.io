---
layout: post
title: Programmers 크레인 인형뽑기 게임
subtitle: 2019 카카오 개발자 겨울 인턴십
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [programmers, 프로그래머스, algorithm, c++]
comments: true

---

# Programmers 크레인 인형뽑기 게임

- 최초 작성일: 2021년 10월 21일(목)
- 주소: https://programmers.co.kr/learn/courses/30/lessons/64061

## 목차

[TOC]

## 문제 설명

- 네오와 프로도가 숫자놀이를 하고 있습니다. 네오가 프로도에게 숫자를 건넬 때 일부 자릿수를 영단어로 바꾼 카드를 건네주면 프로도는 원래 숫자를 찾는 게임입니다.
- 다음은 숫자의 일부 자릿수를 영단어로 바꾸는 예시입니다.
  - 1478 → "one4seveneight"
  - 234567 → "23four5six7"
  - 10203 → "1zerotwozero3"
- 이렇게 숫자의 일부 자릿수가 영단어로 바뀌어졌거나, 혹은 바뀌지 않고 그대로인 문자열 `s`가 매개변수로 주어집니다. `s`가 의미하는 원래 숫자를 return 하도록 solution 함수를 완성해주세요.

- 참고로 각 숫자에 대응되는 영단어는 다음 표와 같습니다.

| 숫자 | 영단어 |
| ---- | :----- |
| 0    | zero   |
| 1    | one    |
| 2    | two    |
| 3    | three  |
| 4    | four   |
| 5    | five   |
| 6    | six    |
| 7    | seven  |
| 8    | eight  |
| 9    | nine   |

## 제한 사항

- 1 ≤ `s`의 길이 ≤ 50
- `s`가 "zero" 또는 "0"으로 시작하는 경우는 주어지지 않습니다.
- return 값이 1 이상 2,000,000,000 이하의 정수가 되는 올바른 입력만 `s`로 주어집니다.

## 입출력 예

| s                    | result |
| -------------------- | ------ |
| `"one4seveneight"`   | 1478   |
| `"23four5six7"`      | 234567 |
| `"2three45sixseven"` | 234567 |
| `"123"`              | 123    |

## 풀이 방법 1

- moves 벡터는 어떤 인덱스의 가장 위에 위치한 인형을 하나씩 꺼낼지를 나타낸다.
- 그러므로, board의 size만큼 돌며 해당 인덱스의 가장 위 인형을 하나씩 꺼내며 지워주고, 새로운 빈 벡터에 하나씩 넣어준다.
- 그러던 중, 새로운 벡터에 연속으로 같은 숫자의 인형이 쌓이면 pop_back()을 통해 제거해주고, answer를 2 증가한다. 지워진 숫자를 제외한 마지막 숫자를 before로 지정해주어, 다음 번에 같은 숫자가 나왔을 시 지울 수 있도록 해준다.
- 예를 들어, 4 3 1 1 3 2 3 4 순으로 인형을 꺼냈다고 했을 때, 1 1 을 먼저 지워주면 4 3 3 2 3 4 순서가 되므로, 또 3 3을 지워줘야 한다. 
- 그렇게 마지막 answer를 출력해준다.

---

```c++
#include <iostream>
#include <string>
#include <vector>

using namespace std;

int solution(vector<vector<int>> board, vector<int> moves) {
    int answer = 0;
    int before = 0;
    vector<int> v;

    for (int i = 0; i < moves.size(); i++) {
        for (int j = 0; j < board.size(); j++) {
            if (board[j][moves[i]-1] != 0) {
            
                if (before == board[j][moves[i]-1]) {
                    v.pop_back();

                    answer = answer + 2;
                    before = v.back();
                    board[j][moves[i]-1] = 0;
                    break;
                }
                v.push_back(board[j][moves[i]-1]);
                before = board[j][moves[i]-1];

                board[j][moves[i]-1] = 0;
                break;
            }
        }
    }
    return answer;
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(0);

    vector<vector<int>> board = { {0,0,0,0,0},{0,0,1,0,3},{0,2,5,0,1},{4,2,4,4,2},{3,5,1,3,1} };
    vector<int> moves = {1, 5, 3, 5, 1, 2, 1, 4};

    cout << solution(board, moves) << "\n";

    return 0;
}

```

---

