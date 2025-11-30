import curses
import random
import numpy as np


# ===============================
#            GAME LOGIC
# ===============================
class Game2048:
    def __init__(self):
        self.grid = np.zeros((4, 4), dtype=int)
        self.score = 0
        self.add_new_tile()
        self.add_new_tile()

    def add_new_tile(self):
        empty_cells = [(i, j) for i in range(4) for j in range(4) if self.grid[i][j] == 0]
        if empty_cells:
            i, j = random.choice(empty_cells)
            self.grid[i][j] = 2 if random.random() < 0.9 else 4

    def move(self, direction):
        # 0: up, 1: right, 2: down, 3: left
        moved = False

        # Rotate grid to simplify logic
        if direction == 0:  # Up
            grid_view = self.grid
        elif direction == 1:  # Right
            grid_view = np.rot90(self.grid, -1)
        elif direction == 2:  # Down
            grid_view = np.rot90(self.grid, 2)
        else:  # Left
            grid_view = np.rot90(self.grid, 1)

        # Process each row
        for i in range(4):
            row = [x for x in grid_view[i] if x != 0]

            j = 0
            while j < len(row) - 1:
                if row[j] == row[j + 1]:
                    row[j] *= 2
                    self.score += row[j]
                    del row[j + 1]
                j += 1

            row.extend([0] * (4 - len(row)))

            if row != list(grid_view[i]):
                moved = True

            grid_view[i] = row

        # Rotate back
        if direction == 1:
            self.grid = np.rot90(grid_view, 1)
        elif direction == 2:
            self.grid = np.rot90(grid_view, -2)
        elif direction == 3:
            self.grid = np.rot90(grid_view, -1)
        else:
            self.grid = grid_view

        if moved:
            self.add_new_tile()

        return moved

    def is_game_over(self):
        if 0 in self.grid:
            return False

        for i in range(4):
            for j in range(4):
                if j < 3 and self.grid[i][j] == self.grid[i][j + 1]:
                    return False
                if i < 3 and self.grid[i][j] == self.grid[i + 1][j]:
                    return False

        return True

    def has_won(self):
        return 2048 in self.grid


# ===============================
#          UI + CURSES
# ===============================
def init_colors():
    curses.start_color()
    curses.use_default_colors()

    curses.init_pair(1, curses.COLOR_WHITE, -1)
    curses.init_pair(2, curses.COLOR_YELLOW, -1)
    curses.init_pair(3, curses.COLOR_RED, -1)
    curses.init_pair(4, curses.COLOR_MAGENTA, -1)
    curses.init_pair(5, curses.COLOR_CYAN, -1)
    curses.init_pair(6, curses.COLOR_GREEN, -1)
    curses.init_pair(7, curses.COLOR_BLUE, -1)
    curses.init_pair(8, curses.COLOR_WHITE, -1)  # Default


def get_tile_color(value):
    mapping = {
        2: 1,
        4: 2,
        8: 3,
        16: 4,
        32: 5,
        64: 6,
        128: 7,
        256: 1,
        512: 2,
        1024: 3,
        2048: 6
    }
    return mapping.get(value, 8)


def draw_game(stdscr, game):
    stdscr.clear()
    height, width = stdscr.getmaxyx()

    # Terminal too small?
    if height < 24 or width < 60:
        stdscr.addstr(1, 1, "🛑 Terminal terlalu kecil!", curses.A_BOLD)
        stdscr.addstr(2, 1, "Perbesar jendela terminal lalu jalankan ulang.")
        stdscr.refresh()
        return

    # Title
    stdscr.addstr(1, width // 2 - 2, "2048", curses.A_BOLD)

    # Score
    stdscr.addstr(3, width // 2 - 6, f"Score: {game.score}")

    # Board offset
    start_y = 5
    start_x = width // 2 - 20

    for i in range(4):
        for j in range(4):
            y = start_y + i * 4
            x = start_x + j * 10

            # Tile box
            for k in range(3):
                stdscr.addstr(y + k, x, " " * 9)

            value = game.grid[i][j]
            if value != 0:
                s = str(value)
                color = curses.color_pair(get_tile_color(value))
                stdscr.addstr(y + 1, x + (9 - len(s)) // 2, s, color | curses.A_BOLD)

    # Instructions
    stdscr.addstr(start_y + 17, width // 2 - 25,
                  "Gunakan Arrow Keys • 'r' restart • 'q' quit")

    # Game Over / Win Messages
    if game.is_game_over():
        stdscr.addstr(start_y + 19, width // 2 - 12,
                      "GAME OVER", curses.A_BOLD | curses.A_BLINK)
    elif game.has_won():
        stdscr.addstr(start_y + 19, width // 2 - 10,
                      "YOU WIN!", curses.A_BOLD | curses.A_BLINK)

    stdscr.refresh()


# ===============================
#           MAIN LOOP
# ===============================
def main(stdscr):
    curses.curs_set(0)
    init_colors()

    game = Game2048()

    while True:
        draw_game(stdscr, game)
        key = stdscr.getch()

        if key == ord("q"):
            break
        elif key == ord("r"):
            game = Game2048()
        elif key == curses.KEY_UP:
            game.move(0)
        elif key == curses.KEY_RIGHT:
            game.move(1)
        elif key == curses.KEY_DOWN:
            game.move(2)
        elif key == curses.KEY_LEFT:
            game.move(3)


# ===============================
#        ENTRY POINT FIXED
# ===============================
if __name__ == "__main__":
    curses.wrapper(main)
