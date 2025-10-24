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

#화면 배경 설정
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

pygame.quit()
# save as move_square.py
import pygame
import sys
import math

pygame.init()

# 화면 설정
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("캐릭터 wsad")

clock = pygame.time.Clock()
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

