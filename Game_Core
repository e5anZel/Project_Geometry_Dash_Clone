import pygame
import sys
import json


# Constants
WIDTH = 1000
HEIGHT = 600
FPS = 60
GRAVITY = 0.5
JUMP_FORCE = -12
PL_SPEED = 5
PL_SIZE = 40

WHITE = (255, 255, 255)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)
YELLOW = (255, 255, 0)
BLACK = (0, 0, 0)
ORANGE = (255, 165, 0)
DARK_GRAY = (40, 40, 40)


class Player:
    def __init__(self):
        self.rect = pygame.Rect(100, HEIGHT // 2, PL_SIZE, PL_SIZE)
        self.vel_x = 0
        self.vel_y = 0
        self.on_ground = False
        self.color = GREEN
        self.jump_count = 0

    def jump(self, force=JUMP_FORCE):
        if self.on_ground:
            self.vel_y = force
            self.on_ground = False
            self.jump_count += 1

    def update(self, dt):
        self.rect.x += self.vel_x
        self.vel_y += GRAVITY
        self.rect.y += self.vel_y

        if self.rect.bottom > HEIGHT:
            self.rect.bottom = HEIGHT
            self.on_ground = True
            self.vel_y = 0
        else:
            self.on_ground = False


class Object:
    def __init__(self, type_, x, y, width=100, height=20, jump_force=JUMP_FORCE):
        self.type = type_
        self.rect = pygame.Rect(x, y, width, height)
        self.jump_force = jump_force

        if type_ == 'platform':
            self.color = BLUE
        elif type_ == 'jump_pad':
            self.color = YELLOW
        elif type_ == 'spike':
            self.color = RED
        else:
            self.color = WHITE


class LevelEditor:
    def __init__(self, game):
        self.game = game
        self.selected_tool = 'platform'
        self.objects = []
        self.dragging = False
        self.selected_obj = None
        self.razn_betw_obj_curs = (0, 0)
        self.show_grid = True
        self.grid_size = 50
        self.camera_x = 0
        self.camera_y = 0
        self.last_pos = (0, 0)
        self.floor_y = HEIGHT

    # Event handling methods
    def handle_events(self):
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

            if event.type == pygame.MOUSEBUTTONDOWN:
                if event.button == 1:
                    self.handle_left_click(pygame.mouse.get_pos())
                elif event.button == 2:
                    self.last_pos = pygame.mouse.get_pos()
                elif event.button == 3:
                    self.handle_right_click(pygame.mouse.get_pos())

            if event.type == pygame.MOUSEBUTTONUP:
                if event.button == 1:
                    self.dragging = False
                    self.selected_obj = None
                elif event.button == 2:
                    self.last_pos = None

            if event.type == pygame.MOUSEMOTION:
                if self.dragging:
                    self.drag_object(pygame.mouse.get_pos())
                elif event.buttons[1]:
                    x, y = pygame.mouse.get_pos()
                    dx = x - self.last_pos[0]
                    dy = y - self.last_pos[1]
                    self.camera_x -= dx
                    self.camera_y -= dy
                    self.last_pos = (x, y)

            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_TAB:
                    self.show_grid = not self.show_grid
                elif event.key == pygame.K_h:
                    self.save_level()
                elif event.key == pygame.K_l:
                    self.load_level()
                elif event.key == pygame.K_t:
                    self.game.levels = [{'Objects': self.objects.copy()}]
                    self.game.current_level = -1
                    self.game.state = 'game'
                    self.game.reset_level()
                elif event.key == pygame.K_ESCAPE:
                    self.game.state = 'start'
                elif event.key == pygame.K_PLUS or event.key == pygame.K_EQUALS:
                    self.grid_size = min(200, self.grid_size + 10)
                elif event.key == pygame.K_MINUS:
                    self.grid_size = max(10, self.grid_size - 10)
                elif event.key in (pygame.K_1, pygame.K_2, pygame.K_3):
                    tools = ['platform', 'jump_pad', 'spike']
                    self.selected_tool = tools[event.key - pygame.K_1]
                elif self.selected_obj:
                    if event.key == pygame.K_LEFT:
                        self.selected_obj.rect.width = max(10, self.selected_obj.rect.width - 10)
                    elif event.key == pygame.K_RIGHT:
                        self.selected_obj.rect.width += 10
                    elif event.key == pygame.K_UP:
                        self.selected_obj.rect.height = max(10, self.selected_obj.rect.height - 10)
                    elif event.key == pygame.K_DOWN:
                        self.selected_obj.rect.height += 10
                    elif event.key == pygame.K_LEFTBRACKET and self.selected_obj.type == 'jump_pad':
                        self.selected_obj.jump_force -= 1
                    elif event.key == pygame.K_RIGHTBRACKET and self.selected_obj.type == 'jump_pad':
                        self.selected_obj.jump_force += 1

    def handle_left_click(self, mouse_pos):
        if mouse_pos[1] < 80:
            return

        world_x = mouse_pos[0] + self.camera_x
        world_y = mouse_pos[1] + self.camera_y

        if world_y > self.floor_y:
            return

        if self.selected_tool == 'platform':
            width, height = 100, 20
        elif self.selected_tool == 'jump_pad':
            width, height = 50, 20
        elif self.selected_tool == 'spike':
            width, height = 40, 40

        new_obj = Object(self.selected_tool, world_x, world_y, width, height)
        self.objects.append(new_obj)
        self.selected_obj = new_obj

    def handle_right_click(self, mouse_pos):
        world_x = mouse_pos[0] + self.camera_x
        world_y = mouse_pos[1] + self.camera_y
        for obj in reversed(self.objects):
            if obj.rect.collidepoint(world_x, world_y):
                self.objects.remove(obj)
                break

    def drag_object(self, mouse_pos):
        if self.selected_obj:
            new_x = mouse_pos[0] + self.camera_x - self.razn_betw_obj_curs[0]
            new_y = mouse_pos[1] + self.camera_y - self.razn_betw_obj_curs[1]

            if new_y + self.selected_obj.rect.height > self.floor_y:
                new_y = self.floor_y - self.selected_obj.rect.height

            self.selected_obj.rect.x = new_x
            self.selected_obj.rect.y = new_y

    # Save/load methods
    def save_level(self):
        level_data = []
        for obj in self.objects:
            data = {
                'type': obj.type,
                'x': obj.rect.x,
                'y': obj.rect.y,
                'width': obj.rect.width,
                'height': obj.rect.height,
                'jump_force': obj.jump_force
            }
            level_data.append(data)

        try:
            with open('custom_level.json', 'w') as f:
                json.dump(level_data, f, indent=4)
            print("Уровень успешно сохранен!")
        except Exception as e:
            print(f"Ошибка сохранения: {e}")

    def load_level(self):
        try:
            with open('custom_level.json', 'r') as f:
                level_data = json.load(f)
            self.objects = []
            for data in level_data:
                obj = Object(
                    data['type'],
                    data['x'],
                    data['y'],
                    data['width'],
                    data['height'],
                    data.get('jump_force')
                )
                self.objects.append(obj)
            print("Уровень загружен!")
        except Exception as e:
            print(f'Ошибка загрузки: {e}')

    # Update and draw methods
    def update(self):
        keys = pygame.key.get_pressed()
        speed = 15
        if keys[pygame.K_a]:
            self.camera_x -= speed
        if keys[pygame.K_d]:
            self.camera_x += speed
        if keys[pygame.K_w]:
            self.camera_y -= speed
        if keys[pygame.K_s]:
            self.camera_y += speed

    def draw_grid(self):
        if self.show_grid:
            for x in range(-self.camera_x % self.grid_size, WIDTH, self.grid_size):
                pygame.draw.line(self.game.screen, (60, 60, 60), (x, 0), (x, HEIGHT))
            for y in range(-self.camera_y % self.grid_size, HEIGHT, self.grid_size):
                pygame.draw.line(self.game.screen, (60, 60, 60), (0, y), (WIDTH, y))

    def draw_interface(self):
        pygame.draw.rect(self.game.screen, (30, 30, 30), (0, 0, WIDTH, 80))
        tools = ['platform', 'jump_pad', 'spike']
        tool_colors = {
            'platform': BLUE,
            'jump_pad': YELLOW,
            'spike': RED
        }
        for i, tool in enumerate(tools):
            base_x = 10 + i * 110
            color = tool_colors[tool]
            if self.selected_tool == tool:
                color = tuple(min(c + 40, 255) for c in color)
            pygame.draw.rect(self.game.screen, color, (base_x, 10, 100, 60), border_radius=5)
            display_text = tool.replace('_', ' ').capitalize()
            self.game.draw_text(display_text, base_x + 20, 30, color=BLACK)
            self.game.draw_text(str(i + 1), base_x + 80, 10, color=BLACK)

        self.game.draw_text(f"Grid: {self.grid_size}px (+/-)", 700, 20, color=WHITE)
        self.game.draw_text("[H] Сохранить", 700, 40, color=WHITE)
        self.game.draw_text("[T] Протестировать [ESC] Вернуться", 700, 60, color=WHITE)
        self.game.draw_text("[L] Загрузить", 700, 0, color=WHITE)

        if self.selected_obj:
            info = [
                f"Тип объекта: {self.selected_obj.type}",
                f"Координаты: ({self.selected_obj.rect.x}, {self.selected_obj.rect.y})",
                f"Размер: {self.selected_obj.rect.width}x{self.selected_obj.rect.height}"
            ]
            if self.selected_obj.type == 'jump_pad':
                info.append(f"Сила прыжка: {self.selected_obj.jump_force}")

            y = HEIGHT - 30 - len(info) * 20
            for line in info:
                self.game.draw_text(line, 10, y, color=WHITE)
                y += 20

    def draw(self):
        self.game.screen.fill(DARK_GRAY)
        self.draw_grid()
        pygame.draw.line(
            self.game.screen, WHITE,
            (0 - self.camera_x, self.floor_y - self.camera_y),
            (WIDTH - self.camera_x, self.floor_y - self.camera_y), 2
        )
        for obj in self.objects:
            rect = obj.rect.copy()
            rect.x -= self.camera_x
            rect.y -= self.camera_y
            pygame.draw.rect(self.game.screen, obj.color, rect, border_radius=5)
        self.draw_interface()


class Game:
    def __init__(self):
        self.screen = pygame.display.set_mode((WIDTH, HEIGHT))
        self.clock = pygame.time.Clock()
        self.font = pygame.font.Font(None, 24)
        self.bold_font = pygame.font.Font(None, 36)
        self.levels = [
            {
                'Objects': [
                    Object('platform', 200, 500, 300, 20),
                    Object('jump_pad', 350, 480, 50, 20, jump_force=JUMP_FORCE * 2),
                    Object('spike', 600, 540, 40, 40)
                ]
            },
            {
                'Objects': [
                    Object('platform', 100, 500, 200, 20),
                    Object('jump_pad', 250, 480, 50, 20, jump_force=JUMP_FORCE * 2),
                    Object('platform', 400, 400, 200, 20),
                    Object('spike', 500, 360, 40, 40)
                ]
            },
            {
                'Objects': [
                    Object('platform', 150, 500, 200, 20),
                    Object('spike', 400, 540, 40, 40),
                    Object('spike', 450, 540, 40, 40),
                    Object('spike', 500, 540, 40, 40),
                    Object('spike', 550, 540, 40, 40)
                ]
            },
            {
                'Objects': [
                    Object('platform', 50, 500, 150, 20),
                    Object('jump_pad', 200, 480, 50, 20, jump_force=JUMP_FORCE * 2.5),
                    Object('platform', 300, 450, 150, 20),
                    Object('spike', 500, 540, 40, 40),
                    Object('platform', 600, 400, 200, 20),
                    Object('jump_pad', 750, 380, 50, 20, jump_force=JUMP_FORCE * 3)
                ]
            }
        ]
        self.current_level = 0
        self.state = 'start'
        self.editor = LevelEditor(self)
        self.scroll_x = 0

    def reset_level(self):
        self.player = Player()
        self.scroll_x = self.player.rect.centerx - WIDTH // 2
        self.start_time = pygame.time.get_ticks()

    def handle_collisions(self):
        for obs in self.levels[self.current_level]['Objects']:
            if obs.type == 'spike' and self.player.rect.colliderect(obs.rect):
                self.reset_level()
                return

        for obs in self.levels[self.current_level]['Objects']:
            if self.player.rect.colliderect(obs.rect):
                dx = (self.player.rect.x - obs.rect.x) / obs.rect.width
                dy = (self.player.rect.y - obs.rect.y) / obs.rect.height

                if abs(dx) > abs(dy):
                    if self.player.vel_x > 0:
                        self.player.rect.right = obs.rect.left
                    elif self.player.vel_x < 0:
                        self.player.rect.left = obs.rect.right
                else:
                    if self.player.vel_y > 0:
                        self.player.rect.bottom = obs.rect.top
                        self.player.vel_y = 0
                        self.player.on_ground = True
                        if obs.type == 'jump_pad':
                            self.player.jump(obs.jump_force)
                    elif self.player.vel_y < 0:
                        self.player.rect.top = obs.rect.bottom
                        self.player.vel_y = 0

    def draw_button(self, text, x, y, width, height, base_color, hover_color):
        mouse_pos = pygame.mouse.get_pos()
        color = hover_color if (x < mouse_pos[0] < x + width and
                                y < mouse_pos[1] < y + height) else base_color
        pygame.draw.rect(self.screen, color, (x, y, width, height), border_radius=15)
        text_surf = self.bold_font.render(text, True, WHITE)
        text_rect = text_surf.get_rect(center=(x + width // 2, y + height // 2))
        self.screen.blit(text_surf, text_rect)
        return pygame.mouse.get_pressed()[0] and color == hover_color

    def start_screen(self):
        self.screen.fill(DARK_GRAY)
        self.draw_text("Geometry Clone", WIDTH // 2 - 140, 100, font=self.bold_font)

        if self.draw_button("Editor", 400, 200, 200, 60, BLUE, (30, 144, 255)):
            self.state = 'editor'

        if self.draw_button("Start Game", 400, 280, 200, 60, GREEN, (50, 205, 50)):
            self.state = 'level_select'

        if self.draw_button("Quit", 400, 360, 200, 60, RED, (178, 34, 34)):
            pygame.quit()
            sys.exit()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

    def level_select(self):
        self.screen.fill(DARK_GRAY)
        self.draw_text("Select Level", WIDTH // 2 - 100, 50, font=self.bold_font)

        if self.draw_button("Back", 700, 500, 200, 60, RED, (178, 34, 34)):
            self.state = 'start'

        if self.draw_button("Level 1", 150, 150, 200, 60, BLUE, (30, 144, 255)):
            self.current_level = 0
            self.state = 'game'
            self.reset_level()

        if self.draw_button("Level 2", 150, 250, 200, 60, BLUE, (30, 144, 255)):
            self.current_level = 1
            self.state = 'game'
            self.reset_level()

        if self.draw_button("Level 3", 150, 350, 200, 60, BLUE, (30, 144, 255)):
            self.current_level = 2
            self.state = 'game'
            self.reset_level()

        if self.draw_button("Level 4", 150, 450, 200, 60, BLUE, (30, 144, 255)):
            self.current_level = 3
            self.state = 'game'
            self.reset_level()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

    def draw_text(self, text, x, y, color=WHITE, font=None):
        font = font or self.font
        text_surface = font.render(text, True, color)
        self.screen.blit(text_surface, (x, y))

    def draw_hud(self):
        self.draw_text(f"Jumps: {self.player.jump_count}", WIDTH - 150, 10, color=WHITE)
        elapsed_time = (pygame.time.get_ticks() - self.start_time) / 1000
        self.draw_text(f"Time: {elapsed_time:.1f}s", WIDTH - 150, 30, color=WHITE)
        fps = int(self.clock.get_fps())
        self.draw_text(f"FPS: {fps}", WIDTH - 150, 50, color=WHITE)
        self.draw_text("Player X: " + str(self.player.rect.x), WIDTH - 150, 70, color=WHITE)
        self.draw_text("Player Y: " + str(self.player.rect.y), WIDTH - 150, 90, color=WHITE)
        if self.current_level == -1:
            self.draw_text("Level: editor", WIDTH - 150, 110, color=WHITE)
        else:
            self.draw_text("Level: " + str(self.current_level + 1), WIDTH - 150, 110, color=WHITE)

    def run(self):
        self.reset_level()
        while True:
            dt = self.clock.get_time()
            self.screen.fill(BLACK)

            if self.state == 'start':
                self.start_screen()
            elif self.state == 'level_select':
                self.level_select()
            elif self.state == 'game':
                self.scroll_x = self.player.rect.centerx - WIDTH // 2

                keys = pygame.key.get_pressed()
                self.player.vel_x = (keys[pygame.K_d] - keys[pygame.K_a]) * PL_SPEED
                if keys[pygame.K_ESCAPE]:
                    self.state = 'start'

                for event in pygame.event.get():
                    if event.type == pygame.QUIT:
                        pygame.quit()
                        sys.exit()
                    if event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE:
                        self.player.jump()

                self.player.update(dt)
                self.handle_collisions()

                self.screen.fill(BLACK)
                for obs in self.levels[self.current_level]['Objects']:
                    rect = obs.rect.copy()
                    rect.x -= self.scroll_x
                    pygame.draw.rect(self.screen, obs.color, rect, border_radius=5)

                player_rect = self.player.rect.copy()
                player_rect.x -= self.scroll_x
                pygame.draw.rect(self.screen, self.player.color, player_rect, border_radius=10)

                self.draw_hud()

            elif self.state == 'editor':
                self.editor.handle_events()
                self.editor.update()
                self.editor.draw()

            pygame.display.flip()
            self.clock.tick(FPS)


if __name__ == "__main__":
    pygame.init()
    game = Game()
    game.run()
