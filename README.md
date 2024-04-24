import pygame
import random
import sys

pygame.mixer.pre_init(frequency=44100, size=-16, channels=2, buffer=512)
pygame.init()
screen = pygame.display.set_mode((432,768))
clock = pygame.time.Clock()
game_font = pygame.font.Font('04B_19.ttf', 40)

 # TẠO CÁC BIẾN CHO GAME
gravity = 0.25
bird_movement = 0
game_active = True
score = 0
high_score = 0

 # TẠO HÀM CHO GAME
def drawfloor():
	screen.blit(floor, (floor_x_pos, 650))
	screen.blit(floor, (floor_x_pos + 432, 650))

def create_pipe():
	random_pipe_pos = random.choice(pipe_height)
	if not isinstance(random_pipe_pos, int):
		random_pipe_pos = int(random_pipe_pos)
	bottom_pipe = pipe_surface.get_rect(midtop=(500, random_pipe_pos))
	top_pipe = pipe_surface.get_rect(midtop=(500, random_pipe_pos - 675))
	return (bottom_pipe, top_pipe)

def move_pipe(pipes):
	for pipe in pipes:
		pipe.centerx -= pipe_speed
	return pipes

def draw_pipe(pipes):
	for pipe in pipes:
		if pipe.bottom >= 650:
			screen.blit(pipe_surface, pipe)
		else:
			flip_pipe = pygame.transform.flip(pipe_surface, False, True)
			screen.blit(flip_pipe, pipe)

def check_collision(pipes):
	for pipe in pipes:
		if bird_rect.colliderect(pipe):
			die_sound.play()
			return False
	if bird_rect.top <= -100 or bird_rect.bottom >= 650:
		return False
	return True

def rotate_bird(bird1):
	new_bird = pygame.transform.rotozoom(bird1, -bird_movement * 1.5, 1)
	return new_bird

def bird_animation():
	new_bird = bird_list[bird_index]
	new_bird_rect = new_bird.get_rect(center=(100, bird_rect.centery))
	return new_bird, new_bird_rect

def score_display(game_state):
	global high_score
	if game_state == 'main game':
		score_surface = game_font.render(str(int(score)), True, (255, 255, 255))
		score_rect = score_surface.get_rect(center=(216, 100))
		screen.blit(score_surface, score_rect)
	elif game_state == 'game over':	
		score_surface = game_font.render(f'Score: {int(score)}', True, (255, 255, 255))
		score_rect = score_surface.get_rect(center=(216, 100))
		screen.blit(score_surface, score_rect)

		high_score_surface = game_font.render(f'High Score: {int(high_score)}', True, (255, 255, 255))
		high_score_rect = high_score_surface.get_rect(center=(216, 200))
		screen.blit(high_score_surface, high_score_rect)

		if score > high_score:
			high_score = score

 # chèn background
bg = pygame.image.load('background-night.png').convert()
bg = pygame.transform.scale2x(bg)

 # chèn sàn
floor = pygame.image.load('floor.png').convert_alpha()
floor = pygame.transform.scale2x(floor)
floor_x_pos = 0

 # tạo con chim
birddown = pygame.transform.scale2x(pygame.image.load('yellowbird-downflap.png').convert_alpha())
birdmid = pygame.transform.scale2x(pygame.image.load('yellowbird-midflap.png').convert_alpha())
birdup = pygame.transform.scale2x(pygame.image.load('yellowbird-upflap.png').convert_alpha())
bird_list = [birddown, birdmid, birdup]
bird_index = 0
bird = bird_list[bird_index]
bird_rect = birdmid.get_rect(center=(100, 384))

 # tạo timmer cho bird
birdflap = pygame.USEREVENT + 1
pygame.time.set_timer(birdflap, 200)

 # tạo ống
pipe_surface = pygame.image.load('pipe-green.png').convert()
pipe_surface = pygame.transform.scale2x(pipe_surface)
pipe_list = []

 # tạo timer
spawn_pipe = pygame.USEREVENT
pygame.time.set_timer(spawn_pipe, 2000)
pipe_height = (300, 400, 500)

 # tốc độ spawn của ống
pipe_speed = 3

 # tạo màn hình kết thúc
game_over_surface = pygame.transform.scale2x(pygame.image.load('gameover.png').convert_alpha())
game_over_rect = game_over_surface.get_rect(center=(216, 384))

 #game begin
game_begin_surface = pygame.transform.scale2x(pygame.image.load('message.png').convert_alpha())
game_begin_rect = game_begin_surface.get_rect(center=(216, 384))

 # chèn âm thanh
flap_sound = pygame.mixer.Sound('sfx_wing.wav')
die_sound = pygame.mixer.Sound('sfx_die.wav')
score_sound = pygame.mixer.Sound('sfx_point.wav')
score_sound_countdown = 100

 # while loop
running = True
while running:
	for event in pygame.event.get():
		if event.type == pygame.QUIT:
			running = False
		elif event.type == pygame.MOUSEBUTTONDOWN:
			if event.button == 1:
				if game_active:
					bird_movement = 0
					bird_movement -= 6.5
					flap_sound.play()
				elif not game_active:
					game_active = True
					pipe_list.clear()
					bird_rect.center = (100, 384)
					bird_movement = 0
					score = 0
		if event.type == spawn_pipe and game_active:
			pipe_list.extend(create_pipe())
		if event.type == birdflap:
			if bird_index < 2:
				bird_index += 1
			else:
				bird_index = 0
			bird, bird_rect = bird_animation()

	 # thêm background
	screen.blit(bg, (0, 0))


	if game_active:
		 # thêm con chim
		bird_movement += gravity
		rotated_bird = rotate_bird(bird)
		bird_rect.centery += bird_movement
		screen.blit(rotated_bird, bird_rect)
		game_active = check_collision(pipe_list)
		 # thêm ống
		pipe_list = move_pipe(pipe_list)
		draw_pipe(pipe_list)
		 # cộng điểm
		for pipes in pipe_list:
			if isinstance(pipes, tuple):
				if pipes[0].right < bird_rect.left:
					score += 1
					score_sound.play()
			else:
				print("pipe_list chứa một giá trị không mong muốn:", pipes)

		score_display('main game')

	else:
		screen.blit(game_over_surface, game_over_rect)
		score_display('game over')
		
	 # thêm sàn
	floor_x_pos -= 1
	drawfloor()
	if floor_x_pos < -432:
		floor_x_pos = 0


	pygame.display.update()
	clock.tick(120)

 # kết thúc pygame
pygame.quit()
sys.exit()
