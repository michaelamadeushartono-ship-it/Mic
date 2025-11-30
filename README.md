import random
import os

def clear_screen():
    os.system('cls' if os.name == 'nt' else 'clear')

def spawn_tile(board):
    empty = [(r, c) for r in range(4) for c in range(4) if board[r][c] == 0]
    if not empty:
        return
    r, c = random.choice(empty)
    board[r][c] = 2 if random.random() < 0.9 else 4

def slide(row):
    new_row = [num for num in row if num != 0]
    new_row += [0] * (4 - len(new_row))
    return new_row

def combine(row):
    for i in range(3):
        if row[i] != 0 and row[i] == row[i+1]:
            row[i] *= 2
            row[i+1] = 0
    return row

def move_left(board):
    moved = False
    new_board = []
    for row in board:
        s = slide(row)
        c = combine(s)
        f = slide(c)
        if f != row:
            moved = True
        new_board.append(f)
    return new_board, moved

def move_right(board):
    inverted = [row[::-1] for row in board]
    moved_board, moved = move_left(inverted)
    return [row[::-1] for row in moved_board], moved

def move_up(board):
    transposed = list(map(list, zip(*board)))
    moved_board, moved = move_left(transposed)
    return list(map(list, zip(*moved_board))), moved

def move_down(board):
    transposed = list(map(list, zip(*board)))
    moved_board, moved = move_right(transposed)
    return list(map(list, zip(*moved_board))), moved

def print_board(board):
    print("\n2048 GAME\n")
    for row in board:
        print("+------" * 4 + "+")
        print("".join(f"|{str(num).center(6)}" if num > 0 else "|      " for num in row) + "|")
    print("+------" * 4 + "+\n")

def has_moves(board):
    for r in range(4):
        for c in range(4):
            if board[r][c] == 0:
                return True
            if c < 3 and board[r][c] == board[r][c+1]:
                return True
            if r < 3 and board[r][c] == board[r+1][c]:
                return True
    return False

def main():
    board = [[0]*4 for _ in range(4)]
    spawn_tile(board)
    spawn_tile(board)

    while True:
        clear_screen()
        print_board(board)

        move = input("Move (W/A/S/D): ").lower()
        if move not in ['w', 'a', 's', 'd']:
            continue

        if move == 'a':
            board, moved = move_left(board)
        elif move == 'd':
            board, moved = move_right(board)
        elif move == 'w':
            board, moved = move_up(board)
        elif move == 's':
            board, moved = move_down(board)

        if moved:
            spawn_tile(board)

        if not has_moves(board):
            clear_screen()
            print_board(board)
            print("GAME OVER!")
            break

if __name__ == "__main__":
    main()
