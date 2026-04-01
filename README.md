# SNAKE-GAME
**backend code in c laguage **

#include <stdlib.h>

#define WIDTH 60
#define HEIGHT 40
#define MAX_LEN 100

typedef struct {
    int x[MAX_LEN];
    int y[MAX_LEN];
    int length;
    int dir;      // 0=UP,1=DOWN,2=LEFT,3=RIGHT
    int food_x;
    int food_y;
    int score;
    int game_over;
} Snake;

Snake s;

void init_game() {
    s.length = 3;
    s.x[0] = 10; s.y[0] = 10;
    s.x[1] = 9;  s.y[1] = 10;
    s.x[2] = 8;  s.y[2] = 10;
    s.dir = 3;
    s.food_x = rand() % WIDTH;
    s.food_y = rand() % HEIGHT;
    s.score = 0;
    s.game_over = 0;
}

void set_direction(int d) {
    s.dir = d;
}

void update_game() {
    if (s.game_over) return;

    for (int i = s.length; i > 0; i--) {
        s.x[i] = s.x[i-1];
        s.y[i] = s.y[i-1];
    }

    if (s.dir == 0) s.y[0]--;
    if (s.dir == 1) s.y[0]++;
    if (s.dir == 2) s.x[0]--;
    if (s.dir == 3) s.x[0]++;

    if (s.x[0] < 0 || s.y[0] < 0 ||
        s.x[0] >= WIDTH || s.y[0] >= HEIGHT)
        s.game_over = 1;

    for (int i = 1; i < s.length; i++)
        if (s.x[0] == s.x[i] && s.y[0] == s.y[i])
            s.game_over = 1;

    if (s.x[0] == s.food_x && s.y[0] == s.food_y) {
        s.length++;
        s.score += 10;
        s.food_x = rand() % WIDTH;
        s.food_y = rand() % HEIGHT;
    }
}

int get_length() { return s.length; }
int get_x(int i) { return s.x[i]; }
int get_y(int i) { return s.y[i]; }
int get_food_x() { return s.food_x; }
int get_food_y() { return s.food_y; }
int get_score() { return s.score; }
int is_game_over() { return s.game_over; }



front end code in python :
import pygame
import ctypes
import sys
import os
import random

# ================= INIT =================
pygame.mixer.pre_init(44100, -16, 2, 512)
pygame.init()
pygame.mixer.init()

# ================= C BACKEND =================
lib = ctypes.CDLL("./libsnake.dylib")
lib.set_direction.argtypes = [ctypes.c_int]
lib.update_game.restype = None
lib.get_length.restype = ctypes.c_int
lib.get_x.argtypes = [ctypes.c_int]; lib.get_x.restype = ctypes.c_int
lib.get_y.argtypes = [ctypes.c_int]; lib.get_y.restype = ctypes.c_int
lib.get_food_x.restype = ctypes.c_int
lib.get_food_y.restype = ctypes.c_int
lib.get_score.restype = ctypes.c_int
lib.is_game_over.restype = ctypes.c_int

# ================= SETTINGS =================
GRID_W, GRID_H = 60, 40
CELL = 12
WIDTH, HEIGHT = GRID_W * CELL, GRID_H * CELL
FPS = 12
BASE = os.path.dirname(os.path.abspath(__file__))
SCORE_FILE = os.path.join(BASE, "scores.txt")

# ================= DISPLAY =================
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("SLITHERENE")
clock = pygame.time.Clock()
FONT_BIG = pygame.font.SysFont("Arial Black", 56)
FONT = pygame.font.SysFont("Arial", 22)

# ================= SOUNDS =================
def snd(name, vol):
    s = pygame.mixer.Sound(os.path.join(BASE, "sounds", name))
    s.set_volume(vol)
    return s

SND_MOVE = snd("move.wav", 0.15)
SND_EAT  = snd("eat.wav", 0.6)
SND_OVER = snd("gameover.wav", 0.8)

# ================= THEMES =================
THEMES = {
    "Summer": {"sky": (135,206,235), "ground": (60,179,113), "food": (220,20,60)},
    "Autumn": {"sky": (250,214,165), "ground": (205,133,63), "food": (178,34,34)},
    "Christmas": {"sky": (210,235,255), "ground": (245,250,250), "food": (220,20,60)}
}

# ================= PARTICLES =================
snow = [(random.randint(0, WIDTH), random.randint(0, HEIGHT)) for _ in range(80)]
leaves = [(random.randint(0, WIDTH), random.randint(0, HEIGHT)) for _ in range(40)]
lights = [(x, 30) for x in range(0, WIDTH, 40)]

# ================= SCORE =================
def save_score(name, score):
    with open(SCORE_FILE, "a") as f:
        f.write(f"{name}:{score}\n")

def get_leaderboard():
    if not os.path.exists(SCORE_FILE):
        return []
    with open(SCORE_FILE) as f:
        data = [line.strip().split(":") for line in f if ":" in line]
    data.sort(key=lambda x: int(x[1]), reverse=True)
    return data[:5]

# ================= FRONT PAGE =================
def front_page():
    while True:
        clock.tick(30)
        for e in pygame.event.get():
            if e.type == pygame.QUIT:
                sys.exit()
            if e.type == pygame.KEYDOWN:
                return

        screen.fill((135,206,235))
        pygame.draw.rect(screen, (60,179,113),
                         (0, HEIGHT//2, WIDTH, HEIGHT//2))

        title = FONT_BIG.render("SLITHERENE", True, (0,100,0))
        screen.blit(title, title.get_rect(center=(WIDTH//2, HEIGHT//2 - 40)))

        hint = FONT.render("Press any key to start", True, (0,0,0))
        screen.blit(hint, hint.get_rect(center=(WIDTH//2, HEIGHT//2 + 30)))

        pygame.display.flip()

# ================= ENTER NAME =================
def enter_name():
    name = ""
    while True:
        clock.tick(30)
        for e in pygame.event.get():
            if e.type == pygame.QUIT:
                sys.exit()
            if e.type == pygame.KEYDOWN:
                if e.key == pygame.K_RETURN and name:
                    return name
                elif e.key == pygame.K_BACKSPACE:
                    name = name[:-1]
                elif e.unicode.isalpha() and len(name) < 10:
                    name += e.unicode.upper()

        screen.fill((60,160,60))
        title = FONT_BIG.render("ENTER NAME", True, (0,0,0))
        screen.blit(title, title.get_rect(center=(WIDTH//2, HEIGHT//2 - 60)))
        name_txt = FONT.render(name + "_", True, (0,0,0))
        screen.blit(name_txt, name_txt.get_rect(center=(WIDTH//2, HEIGHT//2)))
        pygame.display.flip()

# ================= THEME SELECT =================
def select_theme():
    names = list(THEMES.keys())
    i = 0
    while True:
        clock.tick(30)
        for e in pygame.event.get():
            if e.type == pygame.QUIT:
                sys.exit()
            if e.type == pygame.KEYDOWN:
                if e.key == pygame.K_LEFT:
                    i = (i-1) % len(names)
                elif e.key == pygame.K_RIGHT:
                    i = (i+1) % len(names)
                elif e.key == pygame.K_RETURN:
                    return names[i]

        draw_background(names[i])
        txt = FONT_BIG.render(names[i], True, (0,0,0))
        screen.blit(txt, txt.get_rect(center=(WIDTH//2, 80)))
        hint = FONT.render("← → ENTER", True, (0,0,0))
        screen.blit(hint, hint.get_rect(center=(WIDTH//2, HEIGHT-60)))
        pygame.display.flip()

# ================= BACKGROUND =================
def draw_background(theme_name):
    theme = THEMES[theme_name]
    screen.fill(theme["sky"])
    pygame.draw.rect(screen, theme["ground"], (0, HEIGHT//2, WIDTH, HEIGHT))

    if theme_name == "Summer":
        pygame.draw.circle(screen, (255,215,0), (WIDTH-80, 80), 40)

    if theme_name == "Autumn":
        for i in range(len(leaves)):
            x,y = leaves[i]
            y = (y + 2) % HEIGHT
            leaves[i] = (x,y)
            pygame.draw.circle(screen, (165,42,42), (x,y), 4)

    if theme_name == "Christmas":
        for i in range(len(snow)):
            x,y = snow[i]
            y = (y + 3) % HEIGHT
            snow[i] = (x,y)
            pygame.draw.circle(screen, (255,255,255), (x,y), 2)
        for x,y in lights:
            pygame.draw.circle(
                screen,
                random.choice([(255,0,0),(0,255,0),(255,255,0)]),
                (x,y), 5
            )

# ================= GAME =================
def game_loop(player, theme_name):
    lib.init_game()
    current_dir = 3

    while True:
        clock.tick(FPS)
        for e in pygame.event.get():
            if e.type == pygame.QUIT:
                sys.exit()
            if e.type == pygame.KEYDOWN:
                SND_MOVE.play()
                if e.key == pygame.K_UP and current_dir != 1:
                    current_dir = 0; lib.set_direction(0)
                elif e.key == pygame.K_DOWN and current_dir != 0:
                    current_dir = 1; lib.set_direction(1)
                elif e.key == pygame.K_LEFT and current_dir != 3:
                    current_dir = 2; lib.set_direction(2)
                elif e.key == pygame.K_RIGHT and current_dir != 2:
                    current_dir = 3; lib.set_direction(3)

        prev = lib.get_score()
        lib.update_game()
        if lib.get_score() > prev:
            SND_EAT.play()

        if lib.is_game_over():
            SND_OVER.play()
            save_score(player, lib.get_score())
            return

        draw_background(theme_name)

        for i in range(lib.get_length()):
            pygame.draw.rect(
                screen, (0,120,0),
                (lib.get_x(i)*CELL, lib.get_y(i)*CELL, CELL, CELL), 4
            )

        fx, fy = lib.get_food_x()*CELL, lib.get_food_y()*CELL
        pygame.draw.rect(screen, THEMES[theme_name]["food"], (fx, fy, CELL, CELL), 4)

        score = FONT.render(f"{player} : {lib.get_score()}", True, (0,0,0))
        screen.blit(score, (10,10))
        pygame.display.flip()

# ================= LEADERBOARD =================
def leaderboard_screen():
    data = get_leaderboard()
    while True:
        clock.tick(30)
        for e in pygame.event.get():
            if e.type == pygame.QUIT:
                sys.exit()
            if e.type == pygame.KEYDOWN:
                return

        screen.fill((30,30,30))
        title = FONT_BIG.render("LEADERBOARD", True, (255,255,255))
        screen.blit(title, title.get_rect(center=(WIDTH//2, 60)))

        y = 120
        for n,s in data:
            txt = FONT.render(f"{n} : {s}", True, (200,200,200))
            screen.blit(txt, (WIDTH//2-100, y))
            y += 30

        hint = FONT.render("Press any key to restart", True, (200,200,200))
        screen.blit(hint, hint.get_rect(center=(WIDTH//2, HEIGHT-50)))
        pygame.display.flip()

# ================= MAIN LOOP =================
while True:
    front_page()
    player = enter_name()
    theme = select_theme()
    game_loop(player, theme)
    leaderboard_screen()
