# psychic-adventure# psychic-adventure

import pygame
import sys
import math

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
#박주형 끝

#배서윤 시작
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

#배서윤 끝

#유은별 시작

import pygame  
import random  
#주석'''되어있는 부분 #지우면 랜덤 생성만 남음
# ───────────── 기본 설정 ─────────────
pygame.init() 

# 변수
width, height = 1200, 600
screen = pygame.display.set_mode((width, height))
pygame.display.set_caption("랜덤 아이템 게임")

# 색 (대문자 유지)
WHITE = (255, 255, 255)
YELLOW = (255, 255, 0)
RED = (255, 0, 0)
BLACK = (0, 0, 0)
BLUE = (0, 0, 255)

# 객체 크기
player_size = 40
box_size = 30
bomb_size = 20

# 플레이어 속도
player_speed = 5

# 박스 생성 주기
spawn_tick = 120

# 아이템 종류 및 확률
items = ["열쇠", "스피드 물약", "폭탄"]
item_weights = [20, 40, 40]

# ───────────── 클래스 정의 ─────────────

#''' [플레이어 클래스 시작]
class Player:
    def __init__(self):
        # 플레이어의 초기 위치(화면 중앙)와 크기를 가진 사각형(Rect) 생성
        self.rect = pygame.Rect(width//2, height//2, player_size, player_size)
        self.base_speed = player_speed  # 기본 속도 저장
        self.speed = player_speed       # 현재 속도 (버프에 따라 변함)
        self.boost_end_time = 0         # 속도 버프가 끝나는 시간

    def move(self, keys):
        """키보드 입력에 따라 플레이어 이동"""
        if keys[pygame.K_LEFT] and self.rect.left > 0:
            self.rect.x -= self.speed
        if keys[pygame.K_RIGHT] and self.rect.right < width:
            self.rect.x += self.speed
        if keys[pygame.K_UP] and self.rect.top > 0:
            self.rect.y -= self.speed
        if keys[pygame.K_DOWN] and self.rect.bottom < height:
            self.rect.y += self.speed

    def apply_speed_boost(self):
        """스피드 물약 획득 시 호출"""
        self.speed = self.base_speed * 2
        self.boost_end_time = pygame.time.get_ticks() + 5000 

    def update_status(self):
        """버프 시간 만료 확인"""
        current_time = pygame.time.get_ticks()
        if self.boost_end_time > 0 and current_time > self.boost_end_time:
            self.speed = self.base_speed
            self.boost_end_time = 0
            print("속도 버프 종료!")

    def draw(self, screen):
        """플레이어를 화면에 그림"""
        color = BLUE if self.boost_end_time > 0 else RED
        pygame.draw.rect(screen, color, self.rect)

class PlacedBomb:
    def __init__(self, x, y):
        """플레이어가 설치한 폭탄 클래스"""
        self.rect = pygame.Rect(x, y, bomb_size, bomb_size)
    
    def draw(self, screen):
        """설치된 폭탄 그리기"""
        center = self.rect.center
        pygame.draw.circle(screen, BLACK, center, bomb_size // 2)
#''' [플레이어 클래스 끝]


class Box:
    def __init__(self):
        # 화면 내 랜덤한 X, Y 좌표 생성
        x = random.randint(0, width - box_size)
        y = random.randint(0, height - box_size)
        self.rect = pygame.Rect(x, y, box_size, box_size)

    def draw(self, screen):
        """박스를 화면에 그림"""
        pygame.draw.rect(screen, YELLOW, self.rect)


# ───────────── 메인 게임 루프 ─────────────
def main():
    clock = pygame.time.Clock()
    
    boxes = []           # 생성된 아이템 박스 리스트
    tick_count = 0       # 시간 흐름 변수
    running = True
    
    font = pygame.font.SysFont("malgungothic", 20)

    # UI 표시를 위한 변수는 플레이어 유무와 상관없이 존재해야 하므로 밖으로 뺌
    last_item_msg = "없음"
    bomb_inventory = 0   # 폭탄 개수

    #''' [플레이어 객체 생성]
    player = Player()    # 플레이어 생성
    placed_bombs = []    # 설치된 폭탄 리스트
    #'''

    while running:
        
        screen.fill(WHITE)

        # 이벤트 처리
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            
            #''' [스페이스바 입력]
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE and bomb_inventory > 0:
                    new_bomb = PlacedBomb(player.rect.centerx - bomb_size//2, 
                                          player.rect.centery - bomb_size//2)
                    placed_bombs.append(new_bomb)
                    bomb_inventory -= 1
                    print("폭탄 설치 완료!")
            #'''

        #''' [플레이어 이동]
        keys = pygame.key.get_pressed()
        player.move(keys)
        player.update_status()
        #'''

        # 박스 생성 로직 (플레이어와 무관하게 작동)
        tick_count += 1
        if tick_count >= spawn_tick:
            boxes.append(Box())
            tick_count = 0

        #''' [충돌 및 아이템 획득]
        # 플레이어가 없으면 충돌 체크도 필요 없음
        for box in boxes[:]:
            if player.rect.colliderect(box.rect):
                got_item = random.choices(items, weights=item_weights, k=1)[0]
                last_item_msg = got_item
                print(f"획득한 아이템: {got_item}")

                if got_item == "스피드 물약":
                    player.apply_speed_boost()
                elif got_item == "폭탄":
                    bomb_inventory += 1
                
                boxes.remove(box) # 먹은 박스 제거
        #'''

        # 화면 그리기
        
        #''' [폭탄 그리기]
        for bomb in placed_bombs:
            bomb.draw(screen)
        #'''

        # 박스는 항상 그림
        for box in boxes:
            box.draw(screen)
            
        #''' [플레이어 그리기]
        player.draw(screen)
        #'''

        # 정보 텍스트 (아이템, 폭탄 텍스트는 플레이어가 없어도 표시됨)
        info_text1 = font.render(f"마지막 아이템: {last_item_msg}", True, BLACK)
        info_text2 = font.render(f"보유 폭탄: {bomb_inventory}개 (Space로 설치)", True, BLACK)
        screen.blit(info_text1, (10, 10))
        screen.blit(info_text2, (10, 40))
        
        #''' [속도 텍스트 - 플레이어 정보 필요]
        speed_msg = "속도: 기본"
        if player.boost_end_time > 0:
             remain_time = (player.boost_end_time - pygame.time.get_ticks()) / 1000
             speed_msg = f"속도: 2배! ({remain_time:.1f}초 남음)"
        info_text3 = font.render(speed_msg, True, BLUE if player.boost_end_time > 0 else BLACK)
        screen.blit(info_text3, (10, 70))
        #'''
        
        # 화면 업데이트
        pygame.display.flip()
        clock.tick(60)

    pygame.quit()

if __name__ == "__main__":
    main()

#유은별 끝

#김도현 시작

pygame.quit()
#save as move_square.py
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

