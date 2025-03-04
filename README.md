import pygame
import random
import sys

# Initialize Pygame
pygame.init()

# Screen dimensions
WIDTH, HEIGHT = 480, 640
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Flappy Car")

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
GREEN = (0, 255, 0)

# Car properties
car_img = pygame.image.load("car.png")  # Replace with your car image
car_img = pygame.transform.scale(car_img, (50, 30)) #scale to your car size
car_rect = car_img.get_rect()
car_rect.center = (WIDTH // 4, HEIGHT // 2)
car_y_velocity = 0
gravity = 0.5
flap_strength = -10

# Road properties
road_img = pygame.image.load("road.png")  # Replace with your road image
road_img = pygame.transform.scale(road_img, (WIDTH, HEIGHT)) #scale road to screen
road_x = 0
road_scroll_speed = 5

# Obstacle properties (road dividers)
obstacle_width = 50
obstacle_gap = 200
obstacle_speed = 5
obstacles = []

def create_obstacle():
    obstacle_height = random.randint(100, HEIGHT - obstacle_gap - 100)
    top_obstacle = pygame.Rect(WIDTH, 0, obstacle_width, obstacle_height)
    bottom_obstacle = pygame.Rect(WIDTH, obstacle_height + obstacle_gap, obstacle_width, HEIGHT - obstacle_height - obstacle_gap)
    return top_obstacle, bottom_obstacle

def draw_obstacles(obstacles):
    for obstacle in obstacles:
        pygame.draw.rect(screen, GREEN, obstacle)

def move_obstacles(obstacles):
    for i in range(len(obstacles)):
        obstacles[i].x -= obstacle_speed
    return [obstacle for obstacle in obstacles if obstacle.x > -obstacle_width]

def check_collision(car_rect, obstacles):
    for obstacle in obstacles:
        if car_rect.colliderect(obstacle):
            return True
    return False

def draw_text(text, font, color, surface, x, y):
    textobj = font.render(text, 1, color)
    textrect = textobj.get_rect()
    textrect.topleft = (x, y)
    surface.blit(textobj, textrect)

# Game loop
def main():
    global car_y_velocity, road_x, obstacles
    clock = pygame.time.Clock()
    score = 0
    font = pygame.font.Font(None, 36)
    game_over = False

    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.MOUSEBUTTONDOWN and not game_over:
                car_y_velocity = flap_strength
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE and not game_over:
                    car_y_velocity = flap_strength
                if event.key == pygame.K_r and game_over:
                    main() #restart game

        if not game_over:
            # Road scrolling
            road_x -= road_scroll_speed
            if road_x <= -WIDTH:
                road_x = 0

            # Car movement
            car_y_velocity += gravity
            car_rect.y += car_y_velocity

            # Obstacle generation
            if len(obstacles) == 0 or obstacles[-1].x < WIDTH - 200:
                obstacles.extend(create_obstacle())

            # Obstacle movement
            obstacles = move_obstacles(obstacles)

            # Collision detection
            if check_collision(car_rect, obstacles) or car_rect.top < 0 or car_rect.bottom > HEIGHT:
                game_over = True

            # Score update
            if len(obstacles) > 0 and obstacles[0].x + obstacle_width < car_rect.x and len(obstacles) %2 == 0:
                score += 1

            # Drawing
            screen.blit(road_img, (road_x, 0))
            screen.blit(road_img, (road_x + WIDTH, 0))
            draw_obstacles(obstacles)
            screen.blit(car_img, car_rect)
            draw_text("Score: " + str(score), font, BLACK, screen, 10, 10)

        else:
            draw_text("Game Over! Score: " + str(score), font, BLACK, screen, WIDTH // 4, HEIGHT // 3)
            draw_text("Press R to Restart", font, BLACK, screen, WIDTH // 4, HEIGHT // 2)

        pygame.display.flip()
        clock.tick(60)

if __name__ == "__main__":
    main()
