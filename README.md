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


