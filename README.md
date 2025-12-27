# pygame.kosmos
первая игра про космос
import pygame
import random
import sys

pygame.init()

# ---------- НАЛАШТУВАННЯ ----------
WIDTH, HEIGHT = 1550, 800
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Space Shooter")
clock = pygame.time.Clock()

# ---------- ШРИФТ ----------
font = pygame.font.SysFont(None, 36)
big_font = pygame.font.SysFont(None, 72)

# ---------- ЗОБРАЖЕННЯ ----------
background = pygame.image.load("bacground.jpg").convert()
background = pygame.transform.scale(background, (WIDTH, HEIGHT))

ship = pygame.image.load("corabel.png").convert_alpha()
ship = pygame.transform.scale(ship, (60, 60))

# ---------- ФУНКЦІЯ СТАРТУ ----------
def reset_game():
    return {
        "ship_x": WIDTH // 2 - 30,
        "ship_y": HEIGHT - 80,
        "bullets": [],
        "enemies": [],
        "bg_y": 0,
        "lives": 3,
        "score": 0,
        "spawn_timer": 0,
        "game_over": False
    }

state = reset_game()

ship_speed = 6
bullet_speed = 10
enemy_speed = 4
bg_speed = 2
spawn_delay = 40

running = True

# ================== ГОЛОВНИЙ ЦИКЛ ==================
while running:
    clock.tick(60)

    # ---------- ПОДІЇ ----------
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

        if event.type == pygame.KEYDOWN:
            if not state["game_over"] and event.key == pygame.K_SPACE:
                bullet_x = state["ship_x"] + ship.get_width() // 2 - 2
                bullet_y = state["ship_y"]
                state["bullets"].append([bullet_x, bullet_y])

            if state["game_over"] and event.key == pygame.K_r:
                state = reset_game()

    # ---------- ЯКЩО НЕ GAME OVER ----------
    if not state["game_over"]:
        # --- керування ---
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT] and state["ship_x"] > 0:
            state["ship_x"] -= ship_speed
        if keys[pygame.K_RIGHT] and state["ship_x"] < WIDTH - ship.get_width():
            state["ship_x"] += ship_speed
        if keys[pygame.K_UP] and state["ship_y"] > 0:
            state["ship_y"] -= ship_speed
        if keys[pygame.K_DOWN] and state["ship_y"] < HEIGHT - ship.get_height():
            state["ship_y"] += ship_speed

        # --- фон ---
        state["bg_y"] += bg_speed
        if state["bg_y"] >= HEIGHT:
            state["bg_y"] = 0

        # --- кулі ---
        for bullet in state["bullets"]:
            bullet[1] -= bullet_speed
        state["bullets"] = [b for b in state["bullets"] if b[1] > 0]

        # --- створення ворогів ---
        state["spawn_timer"] += 1
        if state["spawn_timer"] >= spawn_delay:
            enemy_x = random.randint(0, WIDTH - 40)
            state["enemies"].append([enemy_x, -40])
            state["spawn_timer"] = 0

        # --- рух ворогів ---
        for enemy in state["enemies"]:
            enemy[1] += enemy_speed
        state["enemies"] = [e for e in state["enemies"] if e[1] < HEIGHT]

        # --- зіткнення куля ↔ ворог ---
        for bullet in state["bullets"][:]:
            bullet_rect = pygame.Rect(bullet[0], bullet[1], 4, 10)
            for enemy in state["enemies"][:]:
                enemy_rect = pygame.Rect(enemy[0], enemy[1], 40, 40)
                if bullet_rect.colliderect(enemy_rect):
                    state["bullets"].remove(bullet)
                    state["enemies"].remove(enemy)
                    state["score"] += 1
                    break

        # --- зіткнення ворог ↔ корабель ---
        ship_rect = pygame.Rect(
            state["ship_x"], state["ship_y"],
            ship.get_width(), ship.get_height()
        )

        for enemy in state["enemies"][:]:
            enemy_rect = pygame.Rect(enemy[0], enemy[1], 40, 40)
            if ship_rect.colliderect(enemy_rect):
                state["enemies"].remove(enemy)
                state["lives"] -= 1
                if state["lives"] <= 0:
                    state["game_over"] = True

    # ---------- МАЛЮВАННЯ ----------
    screen.blit(background, (0, state["bg_y"]))
    screen.blit(background, (0, state["bg_y"] - HEIGHT))

    screen.blit(ship, (state["ship_x"], state["ship_y"]))

    for bullet in state["bullets"]:
        pygame.draw.rect(screen, (255, 255, 0), (bullet[0], bullet[1], 4, 12))

    for enemy in state["enemies"]:
        pygame.draw.circle(screen, (150, 150, 150), (enemy[0] + 20, enemy[1] + 20), 20)

    # --- UI ---
    lives_text = font.render(f"Lives: {state['lives']}", True, (255, 0, 0))
    score_text = font.render(f"Score: {state['score']}", True, (255, 255, 255))
    screen.blit(lives_text, (10, 10))
    screen.blit(score_text, (10, 40))

    # --- GAME OVER ---
    if state["game_over"]:
        text = big_font.render("GAME OVER", True, (255, 0, 0))
        restart = font.render("Press R to Restart", True, (255, 255, 255))
        screen.blit(text, (WIDTH // 2 - text.get_width() // 2, HEIGHT // 2 - 50))
        screen.blit(restart, (WIDTH // 2 - restart.get_width() // 2, HEIGHT // 2 + 20))

    pygame.display.update()

pygame.quit()
sys.exit()
