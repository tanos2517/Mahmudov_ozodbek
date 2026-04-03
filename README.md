# snake_tetris_menu.py
import pygame
import random

pygame.init()

# -------------------- Настройки экрана --------------------
WIDTH, HEIGHT = 600, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Игровое меню: Змейка и Тетрис")
FONT = pygame.font.SysFont("comicsansms", 40)
SMALL_FONT = pygame.font.SysFont("comicsansms", 30)
clock = pygame.time.Clock()

# -------------------- Цвета --------------------
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
GREEN = (0, 255, 0)
RED = (255, 0, 0)
BLUE = (0, 0, 255)
GRAY = (100, 100, 100)
CYAN = (0, 255, 255)

# -------------------- Общие функции --------------------
def draw_text_center(text, font, color, y):
    render = font.render(text, True, color)
    rect = render.get_rect(center=(WIDTH//2, y))
    screen.blit(render, rect)

# -------------------- Змейка --------------------
CELL_SIZE = 20
SNAKE_WIDTH, SNAKE_HEIGHT = 600, 400

def snake_game():
    snake = [(SNAKE_WIDTH//2, SNAKE_HEIGHT//2)]
    snake_dir = (0, -CELL_SIZE)
    food = (random.randrange(0, SNAKE_WIDTH, CELL_SIZE), random.randrange(0, SNAKE_HEIGHT, CELL_SIZE))
    score = 0
    running = True
    while running:
        screen.fill(BLACK)
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                return
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_UP and snake_dir != (0, CELL_SIZE):
                    snake_dir = (0, -CELL_SIZE)
                elif event.key == pygame.K_DOWN and snake_dir != (0, -CELL_SIZE):
                    snake_dir = (0, CELL_SIZE)
                elif event.key == pygame.K_LEFT and snake_dir != (CELL_SIZE, 0):
                    snake_dir = (-CELL_SIZE, 0)
                elif event.key == pygame.K_RIGHT and snake_dir != (-CELL_SIZE, 0):
                    snake_dir = (CELL_SIZE, 0)
        # Движение змейки
        head_x, head_y = snake[0]
        dx, dy = snake_dir
        new_head = (head_x + dx, head_y + dy)
        if new_head in snake or not (0 <= new_head[0] < SNAKE_WIDTH and 0 <= new_head[1] < SNAKE_HEIGHT):
            draw_text_center(f"Game Over! Счет: {score}", FONT, RED, HEIGHT//2)
            pygame.display.update()
            pygame.time.delay(2000)
            return
        snake.insert(0, new_head)
        if new_head == food:
            score += 1
            food = (random.randrange(0, SNAKE_WIDTH, CELL_SIZE), random.randrange(0, SNAKE_HEIGHT, CELL_SIZE))
        else:
            snake.pop()
        # Рисуем змейку и еду
        for x, y in snake:
            pygame.draw.rect(screen, GREEN, (x, y, CELL_SIZE, CELL_SIZE))
        pygame.draw.rect(screen, RED, (food[0], food[1], CELL_SIZE, CELL_SIZE))
        draw_text_center(f"Счет: {score}", SMALL_FONT, WHITE, HEIGHT - 20)
        pygame.display.update()
        clock.tick(10)

# -------------------- Тетрис --------------------
TETRIS_WIDTH, TETRIS_HEIGHT = 300, 600
CELL_SIZE_T = 30
COLS, ROWS = TETRIS_WIDTH // CELL_SIZE_T, TETRIS_HEIGHT // CELL_SIZE_T
SHAPES = [
    [[1,1,1,1]],       # I
    [[1,1],[1,1]],     # O
    [[0,1,0],[1,1,1]], # T
    [[1,0,0],[1,1,1]], # L
    [[0,0,1],[1,1,1]], # J
    [[1,1,0],[0,1,1]], # S
    [[0,1,1],[1,1,0]]  # Z
]
COLORS = [(0,255,255),(0,0,255),(255,165,0),(255,255,0),(0,255,0),(128,0,128),(255,0,0)]

class Piece:
    def __init__(self, x, y, shape):
        self.x = x
        self.y = y
        self.shape = shape
        self.color = random.choice(COLORS)
        self.rotation = 0

def create_grid(locked={}):
    grid = [[BLACK for _ in range(COLS)] for _ in range(ROWS)]
    for (x, y), color in locked.items():
        grid[y][x] = color
    return grid

def convert_shape(piece):
    positions = []
    shape = piece.shape
    for i, line in enumerate(shape):
        for j, val in enumerate(line):
            if val:
                positions.append((piece.x + j, piece.y + i))
    return positions

def valid_space(piece, grid):
    accepted = [(j,i) for i in range(ROWS) for j in range(COLS) if grid[i][j]==BLACK]
    for pos in convert_shape(piece):
        if pos not in accepted:
            return False
    return True

def check_lost(locked):
    for (x, y) in locked:
        if y < 1:
            return True
    return False

def draw_grid_tetris(screen, grid):
    for i in range(ROWS):
        for j in range(COLS):
            pygame.draw.rect(screen, grid[i][j], (j*CELL_SIZE_T, i*CELL_SIZE_T, CELL_SIZE_T, CELL_SIZE_T))
    for i in range(ROWS):
        pygame.draw.line(screen, WHITE, (0, i*CELL_SIZE_T), (TETRIS_WIDTH, i*CELL_SIZE_T))
    for j in range(COLS):
        pygame.draw.line(screen, WHITE, (j*CELL_SIZE_T, 0), (j*CELL_SIZE_T, TETRIS_HEIGHT))

def tetris_game():
    locked = {}
    grid = create_grid(locked)
    current_piece = Piece(COLS//2-1, 0, random.choice(SHAPES))
    fall_time = 0
    fall_speed = 0.5
    run = True
    clock_t = pygame.time.Clock()

    while run:
        grid = create_grid(locked)
        fall_time += clock_t.get_rawtime()
        clock_t.tick()

        if fall_time/1000 > fall_speed:
            fall_time = 0
            current_piece.y +=1
            if not valid_space(current_piece, grid):
                current_piece.y -=1
                for pos in convert_shape(current_piece):
                    locked[pos] = current_piece.color
                current_piece = Piece(COLS//2-1, 0, random.choice(SHAPES))
                # Очистка рядов
                for i in range(ROWS-1, -1, -1):
                    row = [grid[i][j] for j in range(COLS)]
                    if BLACK not in row:
                        for j in range(COLS):
                            del locked[(j,i)]
                        for key in sorted(list(locked), key=lambda k: k[1])[::-1]:
                            x, y = key
                            if y < i:
                                locked[(x, y+1)] = locked.pop((x, y))

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                return
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_LEFT:
                    current_piece.x -=1
                    if not valid_space(current_piece, grid):
                        current_piece.x +=1
                if event.key == pygame.K_RIGHT:
                    current_piece.x +=1
                    if not valid_space(current_piece, grid):
                        current_piece.x -=1
                if event.key == pygame.K_DOWN:
                    current_piece.y +=1
                    if not valid_space(current_piece, grid):
                        current_piece.y -=1
                if event.key == pygame.K_UP:
                    current_piece.rotation +=1
                    if not valid_space(current_piece, grid):
                        current_piece.rotation -=1

        for pos in convert_shape(current_piece):
            x, y = pos
            if y >= 0:
                grid[y][x] = current_piece.color

        draw_grid_tetris(screen, grid)
        pygame.display.update()

        if check_lost(locked):
            draw_text_center("Game Over!", FONT, RED, HEIGHT//2)
            pygame.display.update()
            pygame.time.delay(2000)
            return

# -------------------- Главное меню --------------------
def main_menu():
    run = True
    while run:
        screen.fill(BLACK)
        draw_text_center("Выберите игру", FONT, WHITE, 150)
        draw_text_center("1. Змейка", SMALL_FONT, GREEN, 250)
        draw_text_center("2. Тетрис", SMALL_FONT, CYAN, 300)
        draw_text_center("ESC - Выход", SMALL_FONT, GRAY, 550)
        pygame.display.update()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                run = False
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_1:
                    snake_game()
                elif event.key == pygame.K_2:
                    tetris_game()
                elif event.key == pygame.K_ESCAPE:
                    run = False

main_menu()
pygame.quit()
