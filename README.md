#include <stdio.h>
#include <stdlib.h>
#include <conio.h>
#include <time.h>
#include <windows.h>

#define SIZE 4  // 棋盘大小4x4

// 全局变量：4x4棋盘
int board[SIZE][SIZE];

// 函数声明
void InitBoard();          // 初始化棋盘
void PrintBoard();         // 显示棋盘
int GenerateRandomNum();   // 生成随机数（2或4）
void GenerateRandomPosition();  // 在随机空位生成数字
int IsBoardFull();         // 判断棋盘是否已满
int MoveLeft();            // 向左移动合并
int MoveRight();           // 向右移动合并
int MoveUp();              // 向上移动合并
int MoveDown();            // 向下移动合并
int CheckWin();            // 检查游戏胜利（是否有2048）
int CheckLose();           // 检查游戏失败（无空位且无相邻相同数字）
void GameOver(int isWin);  // 游戏结束提示

int main() {
    char key;
    int moveSuccess;  // 标记移动是否成功

    // 初始化随机数种子
    srand((unsigned int)time(NULL));
    // 初始化棋盘
    InitBoard();

    while (1) {
        system("cls");  // 清屏
        PrintBoard();   // 显示棋盘

        // 检查游戏胜负
        if (CheckWin()) {
            GameOver(1);
            break;
        }
        if (CheckLose()) {
            GameOver(0);
            break;
        }

        // 获取方向键输入（上：72 下：80 左：75 右：77）
        key = getch();
        moveSuccess = 0;
        switch (key) {
            case 72:  // 上键
                moveSuccess = MoveUp();
                break;
            case 80:  // 下键
                moveSuccess = MoveDown();
                break;
            case 75:  // 左键
                moveSuccess = MoveLeft();
                break;
            case 77:  // 右键
                moveSuccess = MoveRight();
                break;
            case 'q':  // 按q退出游戏
                printf("游戏已退出！\n");
                return 0;
            default:
                break;
        }

        // 移动成功且棋盘未满时，生成随机数
        if (moveSuccess && !IsBoardFull()) {
            GenerateRandomPosition();
        }
    }

    return 0;
}

// 初始化棋盘：清零并生成两个随机数
void InitBoard() {
    // 棋盘清零
    for (int i = 0; i < SIZE; i++) {
        for (int j = 0; j < SIZE; j++) {
            board[i][j] = 0;
        }
    }
    // 生成两个随机数
    GenerateRandomPosition();
    GenerateRandomPosition();
}

// 显示棋盘
void PrintBoard() {
    printf("=========================\n");
    for (int i = 0; i < SIZE; i++) {
        printf("|");
        for (int j = 0; j < SIZE; j++) {
            if (board[i][j] == 0) {
                printf("    |");  // 空位显示四个空格
            } else {
                printf("%4d|", board[i][j]);  // 数字右对齐，占4个字符
            }
        }
        printf("\n");
        printf("=========================\n");
    }
    printf("操作说明：方向键控制移动，按q退出游戏\n");
}

// 生成随机数：2（90%概率）或4（10%概率）
int GenerateRandomNum() {
    return (rand() % 10 == 0) ? 4 : 2;
}

// 在随机空位生成数字
void GenerateRandomPosition() {
    int x, y;
    // 随机找空位
    do {
        x = rand() % SIZE;
        y = rand() % SIZE;
    } while (board[x][y] != 0);

    // 生成数字并赋值
    board[x][y] = GenerateRandomNum();
}

// 判断棋盘是否已满
int IsBoardFull() {
    for (int i = 0; i < SIZE; i++) {
        for (int j = 0; j < SIZE; j++) {
            if (board[i][j] == 0) {
                return 0;  // 有空位，返回0
            }
        }
    }
    return 1;  // 无空位，返回1
}

// 向左移动合并：返回1表示移动成功，0表示未移动
int MoveLeft() {
    int moved = 0;
    int temp[SIZE][SIZE];  // 临时数组存储移动后的结果

    // 初始化临时数组
    for (int i = 0; i < SIZE; i++) {
        for (int j = 0; j < SIZE; j++) {
            temp[i][j] = 0;
        }
    }

    // 处理每一行：向左靠拢
    for (int i = 0; i < SIZE; i++) {
        int idx = 0;  // 临时数组当前存储位置
        for (int j = 0; j < SIZE; j++) {
            if (board[i][j] != 0) {
                temp[i][idx++] = board[i][j];
            }
        }

        // 合并相同数字
        for (int j = 0; j < idx - 1; j++) {
            if (temp[i][j] == temp[i][j + 1]) {
                temp[i][j] *= 2;
                temp[i][j + 1] = 0;
                moved = 1;
            }
        }

        // 再次向左靠拢（处理合并后的空位）
        idx = 0;
        for (int j = 0; j < SIZE; j++) {
            if (temp[i][j] != 0) {
                board[i][idx++] = temp[i][j];
            } else if (idx > 0) {
                board[i][idx++] = 0;
            }
        }
    }

    // 判断是否有移动（对比原棋盘和移动后棋盘）
    if (!moved) {
        for (int i = 0; i < SIZE; i++) {
            for (int j = 0; j < SIZE; j++) {
                if (temp[i][j] != board[i][j]) {
                    moved = 1;
                    break;
                }
            }
            if (moved) break;
        }
    }

    return moved;
}

// 向右移动合并：返回1表示移动成功，0表示未移动
int MoveRight() {
    int moved = 0;
    int temp[SIZE][SIZE];

    // 初始化临时数组
    for (int i = 0; i < SIZE; i++) {
        for (int j = 0; j < SIZE; j++) {
            temp[i][j] = 0;
        }
    }

    // 处理每一行：向右靠拢
    for (int i = 0; i < SIZE; i++) {
        int idx = SIZE - 1;
        for (int j = SIZE - 1; j >= 0; j--) {
            if (board[i][j] != 0) {
                temp[i][idx--] = board[i][j];
            }
        }

        // 合并相同数字
        for (int j = SIZE - 1; j > idx + 1; j--) {
            if (temp[i][j] == temp[i][j - 1]) {
                temp[i][j] *= 2;
                temp[i][j - 1] = 0;
                moved = 1;
            }
        }

        // 再次向右靠拢
        idx = SIZE - 1;
        for (int j = SIZE - 1; j >= 0; j--) {
            if (temp[i][j] != 0) {
                board[i][idx--] = temp[i][j];
            } else if (idx < SIZE - 1) {
                board[i][idx--] = 0;
            }
        }
    }

    // 判断是否有移动
    if (!moved) {
        for (int i = 0; i < SIZE; i++) {
            for (int j = 0; j < SIZE; j++) {
                if (temp[i][j] != board[i][j]) {
                    moved = 1;
                    break;
                }
            }
            if (moved) break;
        }
    }

    return moved;
}

// 向上移动合并：返回1表示移动成功，0表示未移动
int MoveUp() {
    int moved = 0;
    int temp[SIZE][SIZE];

    // 初始化临时数组
    for (int i = 0; i < SIZE; i++) {
        for (int j = 0; j < SIZE; j++) {
            temp[i][j] = 0;
        }
    }

    // 处理每一列：向上靠拢（列转行处理）
    for (int j = 0; j < SIZE; j++) {
        int idx = 0;
        for (int i = 0; i < SIZE; i++) {
            if (board[i][j] != 0) {
                temp[idx++][j] = board[i][j];
            }
        }

        // 合并相同数字
        for (int i = 0; i < idx - 1; i++) {
            if (temp[i][j] == temp[i + 1][j]) {
                temp[i][j] *= 2;
                temp[i + 1][j] = 0;
                moved = 1;
            }
        }

        // 再次向上靠拢
        idx = 0;
        for (int i = 0; i < SIZE; i++) {
            if (temp[i][j] != 0) {
                board[idx++][j] = temp[i][j];
            } else if (idx > 0) {
                board[idx++][j] = 0;
            }
        }
    }

    // 判断是否有移动
    if (!moved) {
        for (int i = 0; i < SIZE; i++) {
            for (int j = 0; j < SIZE; j++) {
                if (temp[i][j] != board[i][j]) {
                    moved = 1;
                    break;
                }
            }
            if (moved) break;
        }
    }

    return moved;
}

// 向下移动合并：返回1表示移动成功，0表示未移动
int MoveDown() {
    int moved = 0;
    int temp[SIZE][SIZE];

    // 初始化临时数组
    for (int i = 0; i < SIZE; i++) {
        for (int j = 0; j < SIZE; j++) {
            temp[i][j] = 0;
        }
    }

    // 处理每一列：向下靠拢（列转行处理）
    for (int j = 0; j < SIZE; j++) {
        int idx = SIZE - 1;
        for (int i = SIZE - 1; i >= 0; i--) {
            if (board[i][j] != 0) {
                temp[idx--][j] = board[i][j];
            }
        }

        // 合并相同数字
        for (int i = SIZE - 1; i > idx + 1; i--) {
            if (temp[i][j] == temp[i - 1][j]) {
                temp[i][j] *= 2;
                temp[i - 1][j] = 0;
                moved = 1;
            }
        }

        // 再次向下靠拢
        idx = SIZE - 1;
        for (int i = SIZE - 1; i >= 0; i--) {
            if (temp[i][j] != 0) {
                board[idx--][j] = temp[i][j];
            } else if (idx < SIZE - 1) {
                board[idx--][j] = 0;
            }
        }
    }

    // 判断是否有移动
    if (!moved) {
        for (int i = 0; i < SIZE; i++) {
            for (int j = 0; j < SIZE; j++) {
                if (temp[i][j] != board[i][j]) {
                    moved = 1;
                    break;
                }
            }
            if (moved) break;
        }
    }

    return moved;
}

// 检查游戏胜利：存在2048返回1，否则返回0
int CheckWin() {
    for (int i = 0; i < SIZE; i++) {
        for (int j = 0; j < SIZE; j++) {
            if (board[i][j] == 2048) {
                return 1;
            }
        }
    }
    return 0;
}

// 检查游戏失败：无空位且无相邻相同数字返回1，否则返回0
int CheckLose() {
    // 先判断是否已满
    if (!IsBoardFull()) {
        return 0;
    }

    // 检查水平方向是否有相邻相同数字
    for (int i = 0; i < SIZE; i++) {
        for (int j = 0; j < SIZE - 1; j++) {
            if (board[i][j] == board[i][j + 1]) {
                return 0;
            }
        }
    }

    // 检查垂直方向是否有相邻相同数字
    for (int j = 0; j < SIZE; j++) {
        for (int i = 0; i < SIZE - 1; i++) {
            if (board[i][j] == board[i + 1][j]) {
                return 0;
            }
        }
    }

    // 无空位且无相邻相同数字，游戏失败
    return 1;
}

// 游戏结束提示
void GameOver(int isWin) {
    system("cls");
    if (isWin) {
        printf("=========================\n");
        printf("|        恭喜你！        |\n");
        printf("|      游戏胜利啦！      |\n");
        printf("|      你合成了2048！    |\n");
        printf("=========================\n");
    } else {
        printf("=========================\n");
        printf("|        游戏结束        |\n");
        printf("|      很遗憾，你输了！  |\n");
        printf("=========================\n");
    }
    printf("按任意键退出...\n");
    getch();
}
