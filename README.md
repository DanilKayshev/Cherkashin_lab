# Здесь собраны лабы
---
# Лаба 1
### вариант 1
Опpеделить следующие отношения: СЫH, ДОЧЬ, ОТЕЦ, МАТЬ, МУЖЧИHА и ЖЕHЩИHА. Описать факты для некотоpых из них. 
Запросы: 
- Опpеделить всех сыновей опpеделенной матеpи.
```Prolog
son(Son, мария).
```
- Опpеделить всех детей опpеделенной паpы pодителей.
```Prolog
children_of_couple(петр, мария, Child).
```
- Опpеделить pодителей опpеделенного человека.
```Prolog
parents_of(иван, Father, Mother).
```
- Является ли опpеделенный человек женщиной?
```
woman(петр).
```
#### Code
```
% Факты о поле
man(петр).
man(иван).
man(алексей).
man(сергей).

woman(мария).
woman(елена).
woman(ольга).
woman(анна).

% Факты о родительских отношениях
parent(мария, иван).    % Мария - родитель Ивана
parent(мария, ольга).   % Мария - родитель Ольги
parent(петр, иван).     % Петр - родитель Ивана
parent(петр, ольга).    % Петр - родитель Ольги
parent(елена, алексей). % Елена - родитель Алексея
parent(сергей, алексей).% Сергей - родитель Алексея
parent(ольга, анна).    % Ольга - родитель Анны
parent(иван, анна).     % Иван - родитель Анны

% Правила
father(Father, Child):-
    man(Father),
    parent(Father, Child).

mother(Mother, Child):-
    woman(Mother),
    parent(Mother, Child).

son(Child, Parent):-
    man(Child),
    parent(Parent, Child).

daughter(Child, Parent):-
    woman(Child),
    parent(Parent, Child).

% Дети определенной пары родителей
children_of_couple(Parent1, Parent2, Child):-
    parent(Parent1, Child),
    parent(Parent2, Child),
    Parent1 \= Parent2.

% Родители определенного человека
parents_of(Person, Father, Mother):-
    father(Father, Person),
    mother(Mother, Person).
```
---
# лаба 2
#### задание 7
Опpеделить, являются ли два заданных элемента соседними в списке.
##### Code
```prolog
is_neighbor(X, Y, [X, Y | _]).

check_neighbor(X, Y, L):-
    is_neighbor(X, Y, L);
    is_neighbor(Y, X, L).

move_next(X, Y, [H|T]):-
    check_neighbor(X, Y, [H|T]);
    move_next(X, Y, T).
```
##### use
```prolog
move_next(4, 5, [1, 2, 3, 4, 5]).
```
#### задание 26
Определить двуаргументный предикат translate(С1, С2) для перевода списка цифр в список соответствующих слов. Например, истинным будет следующее высказывание translate(\[3, 4, 8], \[’три’, ’четыре’, ’восемь’]).
##### Code
```prolog
dict(0,"zero").
dict(1,"one").
dict(2,"two").
dict(3,"three").
dict(4,"four").
dict(5,"five").
dict(6,"six").
dict(7,"seven").
dict(8,"eight").
dict(9,"nine").


translate([], []).
translate([H|T], [Word|Res]):-
    dict(H, Word),
    translate(T, Res).
```
##### use
```prolog
translate([3, 3, 7], T).
```
---
# лаба 3
## из ais2 - учебник
### задача 2 
Решение диофантова уравнения 4𝑥 + 5𝑦 = 0 для значений переменных 𝑥 и 𝑦 из некоторого диапазона, значения переменных — целые числа.
#### Code
``` Prolog
solve_diophant(Min, Max, X, Y) :-
    between(Min, Max, X),
    between(Min, Max, Y),
    4*X + 5*Y =:= 0.
```
#### Use
```
solve_diophant(-20, 20, X, Y)
```
## Из списка
### «Шесть пешек»
#### Что делает этот код
- Реализует *Hexapawn* (3×3, по 3 пешки).
- Человек управляет белыми, вводя ходы в формате `r1 c1 r2 c2`. // r1,c1 - начальная кордината выбранного хода, r2,c2 - конечная кордината
- Компьютер (чёрные) ищет оптимальный ответ с помощью **минимакса на глубину 10** (полное дерево игры — менее 4000 позиций, мгновенный расчёт).
- Проверяет все условия победы: проход на последнюю горизонталь, лишение ходов или полное уничтожение пешек противника.
#### Code 
```python
import copy

# ----------------------------- Игровая логика -----------------------------
ROWS, COLS = 3, 3
WHITE = 'W'
BLACK = 'B'
EMPTY = '.'

def initial_board():
    """Начальная расстановка: белые снизу, чёрные сверху."""
    board = [[EMPTY]*COLS for _ in range(ROWS)]
    for c in range(COLS):
        board[2][c] = WHITE   # белые на строке 2
        board[0][c] = BLACK   # чёрные на строке 0
    return board

def print_board(board):
    """Простой вывод доски в консоль."""
    print("  0 1 2")
    for r in range(ROWS):
        print(r, ' '.join(board[r]))
    print()

def is_valid(r, c):
    return 0 <= r < ROWS and 0 <= c < COLS

def legal_moves(board, player):
    """Возвращает список ходов (r1,c1, r2,c2) для игрока player."""
    moves = []
    direction = -1 if player == WHITE else 1   # белые вверх, чёрные вниз
    for r in range(ROWS):
        for c in range(COLS):
            if board[r][c] != player:
                continue
            # Движение вперёд
            nr = r + direction
            if is_valid(nr, c) and board[nr][c] == EMPTY:
                moves.append((r, c, nr, c))
            # Взятие по диагонали
            for dc in (-1, 1):
                nc = c + dc
                if is_valid(nr, nc) and board[nr][nc] not in (EMPTY, player):
                    moves.append((r, c, nr, nc))
    return moves

def apply_move(board, move):
    """Исполняет ход (r1,c1, r2,c2) и возвращает новую доску (копию)."""
    r1, c1, r2, c2 = move
    new_board = copy.deepcopy(board)
    new_board[r2][c2] = new_board[r1][c1]
    new_board[r1][c1] = EMPTY
    return new_board

def check_winner(board):
    """Возвращает победителя: WHITE, BLACK или None."""
    # Пешка дошла до противоположного края
    for c in range(COLS):
        if board[0][c] == WHITE:   # белая пешка на верхней горизонтали
            return WHITE
        if board[2][c] == BLACK:   # чёрная пешка на нижней горизонтали
            return BLACK

    # Нет ходов у одного из игроков (или все пешки соперника съедены)
    white_moves = legal_moves(board, WHITE)
    black_moves = legal_moves(board, BLACK)
    # Проверяем, остались ли пешки вообще
    white_exists = any(board[r][c] == WHITE for r in range(ROWS) for c in range(COLS))
    black_exists = any(board[r][c] == BLACK for r in range(ROWS) for c in range(COLS))

    if not white_exists or not white_moves:
        return BLACK   # белые не могут ходить – победа чёрных
    if not black_exists or not black_moves:
        return WHITE   # чёрные не могут ходить – победа белых

    return None   # игра продолжается

def game_over(board):
    return check_winner(board) is not None

# ----------------------------- Минимаксный ИИ -----------------------------
def minimax(board, depth, maximizing_player):
    """
    Возвращает оценку позиции.
    ИИ играет за чёрных (BLACK) – минимизирующий игрок.
    """
    winner = check_winner(board)
    if winner == WHITE:
        return 10 - depth   # предпочитаем быструю победу (для белых – положительно)
    if winner == BLACK:
        return -10 + depth  # для чёрных – отрицательно, быстрая победа лучше

    if depth == 0 or game_over(board):
        return 0   # ничья (в этой игре ничьих нет, но на всякий случай)

    if maximizing_player:   # ход белых (максимизируем оценку)
        max_eval = -float('inf')
        for move in legal_moves(board, WHITE):
            new_board = apply_move(board, move)
            eval = minimax(new_board, depth-1, False)
            max_eval = max(max_eval, eval)
        return max_eval
    else:                   # ход чёрных (минимизируем оценку)
        min_eval = float('inf')
        for move in legal_moves(board, BLACK):
            new_board = apply_move(board, move)
            eval = minimax(new_board, depth-1, True)
            min_eval = min(min_eval, eval)
        return min_eval

def best_move(board, player=BLACK):
    """Выбирает лучший ход для чёрных с помощью минимакса (глубина 10 – полный обход)."""
    best_score = float('inf')
    best_moves = []
    for move in legal_moves(board, player):
        new_board = apply_move(board, move)
        score = minimax(new_board, depth=10, maximizing_player=True)
        if score < best_score:
            best_score = score
            best_moves = [move]
        elif score == best_score:
            best_moves.append(move)
    # Если несколько равноценных ходов – выбираем первый
    return best_moves[0] if best_moves else None

# ----------------------------- Основной цикл -----------------------------
def main():
    board = initial_board()
    print("Добро пожаловать в «Шесть пешек»!")
    print("Вы играете белыми (W), ходите первыми. Ваши пешки идут вверх.")
    print("Для хода введите координаты: r1 c1 r2 c2 (через пробел).")
    print_board(board)

    while not game_over(board):
        # Ход человека (белые)
        moves = legal_moves(board, WHITE)
        if not moves:
            print("У вас нет ходов. Вы проиграли.")
            break
        print("Ваши возможные ходы:", moves)
        while True:
            try:
                r1, c1, r2, c2 = map(int, input("Ваш ход: ").split())
                if (r1, c1, r2, c2) in moves:
                    break
                else:
                    print("Недопустимый ход, попробуйте снова.")
            except (ValueError, IndexError):
                print("Введите 4 числа: строка и столбец начальной клетки, затем конечной.")
        board = apply_move(board, (r1, c1, r2, c2))
        print_board(board)
        if game_over(board):
            break

        # Ход компьютера (чёрные)
        print("Компьютер думает...")
        move = best_move(board, BLACK)
        if move is None:
            print("У компьютера нет ходов. Вы выиграли!")
            break
        print(f"Ход компьютера: {move}")
        board = apply_move(board, move)
        print_board(board)

    winner = check_winner(board)
    if winner == WHITE:
        print("Поздравляю! Вы победили!")
    elif winner == BLACK:
        print("Компьютер выиграл. Попробуйте ещё раз.")
    else:
        print("Игра завершена.")

if __name__ == "__main__":
    main()
```
