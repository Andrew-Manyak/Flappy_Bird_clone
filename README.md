
# Project Flappi

![image](https://github.com/Andrew-Manyak/Flappy_Bird_clone/assets/118578713/da6f12fe-0b3a-49d1-afc1-cdc9d8da7ca2)

# Refresh rate and window size
clock = pygame.time.Clock()
win_height = 650
win_width = 520
window = pygame.display.set_mode((win_width, win_height))
# load all images, create list for 3 bird versions
bird_pic = [pygame.image.load("pics/bird_down.png"),
            pygame.image.load("pics/bird_mid.png"),
            pygame.image.load("pics/bird_up.png")]
background_pic = pygame.image.load("pics/background.png")
fossil_pic = pygame.image.load("pics/fossil.png")
ground_image = pygame.image.load("pics/ground.png")
top_pipe_pic = pygame.image.load("pics/pipe_top.png")
bot_pipe_pic = pygame.image.load("pics/pipe_bottom.png")
start_pic = pygame.image.load("pics/start.png")
start_pic = pygame.transform.scale(start_pic, (340, 135))
end_pic = pygame.image.load("pics/endd.png")
end_pic = pygame.transform.scale(end_pic, (550, 230))
# extra
plane_pic = pygame.image.load("pics/plane.png")
# Game settings
bird_start_pos = (100, 255)
scroll_speed = 2
score = 0
font = pygame.font.SysFont('Comic Sans', 26)
game_stopped = True


class Bird(pygame.sprite.Sprite):
    def __init__(self):
        pygame.sprite.Sprite.__init__(self)
        self.image = bird_pic[0]
        self.rect = self.image.get_rect()
        self.rect.center = bird_start_pos
        self.image_index = 0
        self.vel = 0
        self.flap = False
        self.alive = True

    def update(self, user_input):
        if self.alive:
            self.image_index += 1
        if self.image_index >= 30:
            self.image_index = 0
        self.image = bird_pic[self.image_index // 10]
        self.vel += 0.5
        if self.vel >= 7:
            self.vel = 7
        if self.rect.y < 520:
            self.rect.y += int(self.vel)
        if self.vel == 0:
            self.flap = False
        self.image = pygame.transform.rotate(self.image, (self.vel - 1) * -7)
        if user_input[pygame.K_SPACE] and not self.flap and self.rect.y > 0 and self.alive:
            self.flap = True
            self.vel = -7


class Pipe(pygame.sprite.Sprite):
    def __init__(self, x, y, image, pipe_type):
        pygame.sprite.Sprite.__init__(self)
        self.image = image
        self.rect = self.image.get_rect()
        self.rect.x, self.rect.y = x, y
        self.enter, self.exit, self.point = False, False, False
        self.pipe_type = pipe_type

    def update(self):
        self.rect.x -= scroll_speed
        if self.rect.x <= -win_width:
            self.kill()

        global score
        if self.pipe_type == 'bot':
            if bird_start_pos[0] > self.rect.topleft[0] and not self.point:
                self.enter = True
            if bird_start_pos[0] > self.rect.topright[0] and not self.point:
                self.exit = True
            if self.enter and self.exit and not self.point:
                self.point = True
                score += 1


class Plane(pygame.sprite.Sprite):
    def __init__(self, x, y):
        pygame.sprite.Sprite.__init__(self)
        if y < 150:
            self.image = pygame.transform.scale(plane_pic, (55, 27))
        elif y < 300:
            self.image = pygame.transform.scale(plane_pic, (40, 20))
        else:
            self.image = pygame.transform.scale(plane_pic, (35, 17))
        self.rect = self.image.get_rect()
        self.rect.x, self.rect.y = x, y

    def update(self):
        # Move Ground
        self.rect.x += (scroll_speed + 0.6)
        if self.rect.x <= -win_width:
            self.kill()


class Ground(pygame.sprite.Sprite):
    def __init__(self, x, y):
        pygame.sprite.Sprite.__init__(self)
        self.image = ground_image
        self.rect = self.image.get_rect()
        self.rect.x, self.rect.y = x, y

    def update(self):
        self.rect.x -= scroll_speed
        if self.rect.x <= -win_width:
            self.kill()


class Fossil(pygame.sprite.Sprite):
    def __init__(self, x, y):
        pygame.sprite.Sprite.__init__(self)
        self.image = fossil_pic
        self.rect = self.image.get_rect()
        self.rect.x, self.rect.y = x, y

    def update(self):
        self.rect.x -= scroll_speed
        if self.rect.x <= -win_width:
            self.kill()


def quit_flappy():
    for x in pygame.event.get():
        if x.type == pygame.QUIT:
            pygame.quit()
            exit()


def main():
    global score
    bird = pygame.sprite.GroupSingle()
    bird.add(Bird())

    pipe_time = 0
    fossil_time = 0
    plane_time = random.randint(3, 6)
    pipes = pygame.sprite.Group()

    x_pos_ground, y_pos_ground = 0, 520
    ground = pygame.sprite.Group()
    ground.add(Ground(x_pos_ground, y_pos_ground))
    plane = pygame.sprite.Group()
    fossil = pygame.sprite.Group()

    start_game = True
    while start_game:
        quit_flappy()
        window.fill((0, 0, 0))
        user_input = pygame.key.get_pressed()

        window.blit(background_pic, (0, 0))
        if len(ground) <= 2:
            ground.add(Ground(win_width, y_pos_ground))
        plane.draw(window)
        pipes.draw(window)
        ground.draw(window)
        bird.draw(window)
        fossil.draw(window)

        score_text = font.render('Score: ' + str(score), True, pygame.Color(255, 255, 255))
        window.blit(score_text, (10, 5))

        if bird.sprite.alive:
            ground.update()
            plane.update()
            pipes.update()
            fossil.update()

        bird.update(user_input)

        collision_pipe = pygame.sprite.spritecollide(bird.sprites()[0], pipes, False)
        collision_ground = pygame.sprite.spritecollide(bird.sprites()[0], ground, False)

        if collision_pipe or collision_ground:
            bird.sprite.alive = False
        if collision_ground:
            window.blit(end_pic, (win_width // 2 - end_pic.get_width() // 2,
                                  win_height // 2 - end_pic.get_height() // 2))

        if user_input[pygame.K_r]:
            score = 0
            break

        if pipe_time <= 0 and bird.sprite.alive:
            x_top, x_bot = 550, 550
            y_top = random.randint(-700, -480)
            y_bot = y_top + bot_pipe_pic.get_height()
            if score < 23:
                y_bot += random.randint(90, 130)
            elif score < 33:
                y_bot += random.randint(90, 110)
            elif score < 53:
                y_bot += random.randint(90, 99)
            else:
                y_bot += random.randint(90, 95)
            pipes.add(Pipe(x_top, y_top, top_pipe_pic, 'top'))
            pipes.add(Pipe(x_bot, y_bot, bot_pipe_pic, 'bot'))
            pipe_time = random.randint(190, 240)
            plane_time -= 1
        pipe_time -= 1.9
        if plane_time <= 0 and bird.sprite.alive:
            plane_y = random.randint(20, 450)
            plane.add(Plane(-100, plane_y))
            plane_time = random.randint(2, 7)

        if fossil_time <= 0 and bird.sprite.alive:
            fossil_time = random.randint(500, 1200)
            fossil.add(Fossil(700, 443))
        fossil_time -= 1

        clock.tick(60)
        pygame.display.update()


def menu():
    global game_stopped

    while game_stopped:
        quit_flappy()
        window.fill((0, 0, 0))
        window.blit(background_pic, (0, 0))
        window.blit(ground_image, Ground(0, 520))
        window.blit(bird_pic[0], (100, 250))
        window.blit(start_pic, (win_width // 2 - start_pic.get_width() // 2,
                                win_height // 2 - start_pic.get_height() // 2))
        user_input = pygame.key.get_pressed()
        if user_input[pygame.K_SPACE]:
            main()
        pygame.display.update()


menu()

