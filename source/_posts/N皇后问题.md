---
title: N皇后问题
date: 2022-03-20 17:11:28
tags: 
     - c++
     - 算法
categories: 
      - 算法
      - DFS
      - N皇后问题
comments: true
---

> n 皇后问题研究的是如何将 n 个皇后放置在 n×n 的棋盘上，并且使皇后彼此之间不能相互攻击。给一个整数 n ，返回所有不同的 n 皇后问题 的解决方案。每一种解法包含一个不同的 n 皇后问题的棋子放置方案，方案中 'Q' 和 '.' 分别代表了皇后和空位。

<!-- more -->

#### N皇后问题

```c++
class Solution {
public:
    vector<vector<string>> res;
    vector<vector<string>> solveNQueens(int n) {
        //'.'表示空，'Q'表示皇后，初始化空棋盘
        vector<string> board(n,string(n,'.'));
        backtrack(board,0);
        return res;
    }
    //路径：board中小于row的那些行都已经成功放置了皇后
    //选择列表：第row行的所有列都是放置皇后的选择
    //结束条件：row超过board的最后一行，说明棋盘放满了
    void backtrack(vector<string>& board, int row){
        //触发结束条件
        if(row==board.size()){
            res.push_back(board);
            return;
        }
        int n=board[row].size();
        for(int col=0; col<n;col++){
            //排除不合法选择
            if(!isValid(board, row, col)) continue;
            // 做选择
            board[row][col]='Q';
            //回溯 进行下一行决策
            backtrack(board, row+1);
            //撤销选择
            board[row][col]='.';
        }
    }
    //是否可以在board[row][col]放置皇后
    bool isValid(vector<string>& board, int row, int col){
        int n=board.size();
        //检查列中是否有皇后互相冲突
        for(int i=0;i<row;i++){
            if(board[i][col]=='Q') return false;
        }
        //检查右上方是否有皇后互相冲突
        for(int i=row-1,j=col+1;i>=0&&j<n;i--,j++){
            if(board[i][j]=='Q') return false;
        }
        //检查左上方是否有皇后冲突
        for(int i=row-1,j=col-1;i>=0&&j>=0;i--,j--){
            if(board[i][j]=='Q') return false;
        }
        return true;
    }

};
```