# psychic-adventure# psychic-adventure

import pygame

#초기화
pygame.init()                           #초기화

#화면 크기 설정
WINDOW_WIDTH = 600       
WINDOW_HEIGHT = 300  

#rgb 색깔 설정
BLACK = (0,0,0)
WHITE = (255,255,255)
RED = (255,0,0)
GREEN = (0,255,0)
BLUE = (0,0,255)

#창 설정
display_surface=pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
pygame.display.set_caption('maze runner')

#이미지 불러오기 및 설정
person_image = pygame.image.load('c:/pygame/image/person.png')   #사람
person_rent = person_image.get_rect()
person_rent.topleft = (100, 100)

gun_image= pygame.image.load('c:/pygame/image/gun.png')         #총든사람
gun_rent = person_image.get_rect()
gun_rent.topleft = (150,150)

#화면 배경
display_surface.fill(WHITE)

#게임이 동작하는 동안 계속 실행
running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    display_surface.blit(person_image, person_rent) #사람 그리기
    display_surface.blit(gun_image, gun_rent)
    #디스레이 업데이트
    pygame.display.update()

import numpy as np
import matplotlib.pyplot as plt
import random
import math

# ==========================
# 설정
# ==========================
RINGS = 20               # 반지 개수
SECTORS = 64             # 각 반지 분할 수
CENTER_RINGS = 2         # 중앙 빈 공간
BG = "#0c2e19"           # 어두운 녹색
WALL = "black"
OUTFILE = "circular_maze.png"

# 얇은 라인 설정
LINE_THIN = 0.8         # 미로 벽 두께
BORDER_THIN = 1.2       # 사각형 외곽 두께
EXIT_ERASE = 6          # 출구 부분 덮는 라인 굵기 (BG 색으로 덮기)


# ==========================
# 데이터 구조 세팅
# ==========================
cells = {}
for r in range(RINGS):
    for s in range(SECTORS):
        cells[(r, s)] = {
            "visited": False,
            "in": True,
            "out": True,
            "cw": True,
            "ccw": True
        }


def neighbors(r, s):
    res = []
    res.append((r, (s + 1) % SECTORS))
    res.append((r, (s - 1) % SECTORS))

    if r > 0:
        res.append((r - 1, s))
    if r < RINGS - 1:
        res.append((r + 1, s))

    return res


# ==========================
# 미로 생성
# ==========================
def carve():
    stack = [(CENTER_RINGS, 0)]
    cells[(CENTER_RINGS, 0)]["visited"] = True

    while stack:
        r, s = stack[-1]

        unvisited = [(nr, ns) for (nr, ns) in neighbors(r, s)
                     if not cells[(nr, ns)]["visited"]]

        if not unvisited:
            stack.pop()
            continue

        nr, ns = random.choice(unvisited)

        # 벽 제거
        if nr == r:
            if ns == (s + 1) % SECTORS:
                cells[(r, s)]["cw"] = False
                cells[(nr, ns)]["ccw"] = False
            else:
                cells[(r, s)]["ccw"] = False
                cells[(nr, ns)]["cw"] = False

        elif nr == r - 1:
            cells[(r, s)]["in"] = False
            cells[(nr, ns)]["out"] = False

        elif nr == r + 1:
            cells[(r, s)]["out"] = False
            cells[(nr, ns)]["in"] = False

        cells[(nr, ns)]["visited"] = True
        stack.append((nr, ns))


# ==========================
# 중앙 공간 비우기
# ==========================
def clear_center():
    for r in range(CENTER_RINGS):
        for s in range(SECTORS):
            cells[(r, s)]["in"] = False
            cells[(r, s)]["out"] = False
            cells[(r, s)]["cw"] = False
            cells[(r, s)]["ccw"] = False


# ==========================
# 출구 만들기 (4방향)
# ==========================
def carve_exits():
    angles = [0, math.pi/2, math.pi, 3*math.pi/2]  # 동, 북, 서, 남

    for ang in angles:
        s = int(SECTORS * ang / (2 * math.pi)) % SECTORS
        r = RINGS - 1

        while r >= CENTER_RINGS:
            cells[(r, s)]["in"] = False
            if r > 0:
                cells[(r-1, s)]["out"] = False
            r -= 1


# ==========================
# 미로 그리기
# ==========================
def draw_maze():
    fig, ax = plt.subplots(figsize=(10, 10))
    ax.set_facecolor(BG)
    ax.set_aspect("equal")
    plt.axis("off")

    dr = 1 / RINGS

    for r in range(RINGS):
        inner = r * dr
        outer = (r + 1) * dr
        dtheta = 2 * math.pi / SECTORS

        for s in range(SECTORS):
            th0 = s * dtheta
            th1 = (s + 1) * dtheta

            # ccw
            if cells[(r, s)]["ccw"]:
                ax.plot(
                    [inner * math.cos(th0), outer * math.cos(th0)],
                    [inner * math.sin(th0), outer * math.sin(th0)],
                    color=WALL, linewidth=LINE_THIN
                )

            # cw
            if cells[(r, s)]["cw"]:
                ax.plot(
                    [inner * math.cos(th1), outer * math.cos(th1)],
                    [inner * math.sin(th1), outer * math.sin(th1)],
                    color=WALL, linewidth=LINE_THIN
                )

            # inner arc
            if cells[(r, s)]["in"]:
                th = np.linspace(th0, th1, 6)
                ax.plot(inner * np.cos(th), inner * np.sin(th),
                        color=WALL, linewidth=LINE_THIN)

            # outer arc
            if cells[(r, s)]["out"]:
                th = np.linspace(th0, th1, 6)
                ax.plot(outer * np.cos(th), outer * np.sin(th),
                        color=WALL, linewidth=LINE_THIN)

    # 사각형 외곽 (얇게)
    ax.plot([-1, 1], [-1, -1], color=WALL, linewidth=BORDER_THIN)
    ax.plot([-1, 1], [1, 1], color=WALL, linewidth=BORDER_THIN)
    ax.plot([-1, -1], [-1, 1], color=WALL, linewidth=BORDER_THIN)
    ax.plot([1, 1], [-1, 1], color=WALL, linewidth=BORDER_THIN)

    # 출구 4개
    gap = 0.15
    ax.plot([-gap, gap], [1, 1], color=BG, linewidth=EXIT_ERASE)
    ax.plot([-gap, gap], [-1, -1], color=BG, linewidth=EXIT_ERASE)
    ax.plot([1, 1], [-gap, gap], color=BG, linewidth=EXIT_ERASE)
    ax.plot([-1, -1], [-gap, gap], color=BG, linewidth=EXIT_ERASE)

    plt.savefig(OUTFILE, dpi=300, bbox_inches="tight", facecolor=BG)
    print("완료! →", OUTFILE)
    plt.show()


# ==========================
# 실행
# ==========================
random.seed(0)
clear_center()
carve()
carve_exits()
draw_maze()

pygame.quit()
#save as move_square.py
import pyfame
import sys
import math

pygame.init()

#화면 
WIDH, HEIGHT = 800,600
screen = pygame.display.set_mode((WIDTH,HEIGHT))
pygame.display.set_caption('캐릭터 wsad')

clock = pygame.time.Clok()
FPS = 60

# 캐릭터 속성
square_size = 40
x = WIDTH // 2 - square_size // 2
y = HEIGHT // 2 - square_size // 2
speed = 250  # 픽셀/초

# 색상
BG_COLOR = (30, 30, 30)
SQUARE_COLOR = (135, 206, 235)

def clamp(val, a, b):
    return max(a, min(b, val))

running = True
while running:
    dt = clock.tick(FPS) / 1000.0  

   for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

 # 키 상태 읽기 (연속 이동)
 keys = pygame.key.get_pressed()
    dx = 0
    dy = 0
 if keys[pygame.K_w]:  # 위
        dy -= 1
 if keys[pygame.K_s]:  # 아래
        dy += 1
 if keys[pygame.K_a]:  # 왼쪽
        dx -= 1
 if keys[pygame.K_d]:  # 오른쪽
        dx += 1

 # 대각선에서 속도 보정 (같은 speed 유지)
 if dx != 0 or dy != 0:
        length = math.hypot(dx, dy)
        dx /= length
        dy /= length

 x += dx * speed * dt
 y += dy * speed * dt

 # 화면 경계 안으로 고정
 x = clamp(x, 0, WIDTH - square_size)
 y = clamp(y, 0, HEIGHT - square_size)

 # 그리기
 screen.fill(BG_COLOR)
 pygame.draw.rect(screen, SQUARE_COLOR, (int(x), int(y), square_size, square_size))
 pygame.display.flip()

pygame.quit()
sys.exit()

