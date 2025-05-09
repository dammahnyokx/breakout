import asyncio
import pygame
import random
import platform

# Initialize Pygame
pygame.init()

# Screen dimensions
WIDTH = 800
HEIGHT = 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Breakout")

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
BLUE = (0, 0, 255)

# Paddle settings
PADDLE_WIDTH = 100
PADDLE_HEIGHT = 10
paddle_x = WIDTH // 2 - PADDLE_WIDTH // 2
paddle_y = HEIGHT - 40
paddle_speed = 10

# Ball settings
BALL_SIZE = 10
ball_x = WIDTH // 2
ball_y = HEIGHT // 2
ball_dx = 5
ball_dy = -5

# Brick settings
BRICK_WIDTH = 80
BRICK_HEIGHT = 30
BRICK_ROWS = 5
BRICK_COLS = WIDTH // BRICK_WIDTH
bricks = []

# Game state
score = 0
game_over = False
font = pygame.font.Font(None, 36)

def setup():
    global bricks, ball_x, ball_y, ball_dx, ball_dy, paddle_x, score, game_over
    # Reset game state
    bricks.clear()
    score = 0
    game_over = False
    ball_x = WIDTH // 2
    ball_y = HEIGHT // 2
    ball_dx = 5
    ball_dy = -5
    paddle_x = WIDTH // 2 - PADDLE_WIDTH // 2
    
    # Create bricks
    for row in range(BRICK_ROWS):
        for col in range(BRICK_COLS):
            brick = pygame.Rect(col * BRICK_WIDTH, row * BRICK_HEIGHT + 50, BRICK_WIDTH - 2, BRICK_HEIGHT - 2)
            bricks.append(brick)

def update_loop():
    global paddle_x, ball_x, ball_y, ball_dx, ball_dy, score, game_over
    
    if game_over:
        return
    
    # Handle input
    keys = pygame.key.get_pressed()
    if keys[pygame.K_LEFT] and paddle_x > 0:
        paddle_x -= paddle_speed
    if keys[pygame.K_RIGHT] and paddle_x < WIDTH - PADDLE_WIDTH:
        paddle_x += paddle_speed
    if keys[pygame.K_SPACE] and game_over:
        setup()
    
    # Update ball position
    ball_x += ball_dx
    ball_y += ball_dy
    
    # Ball collisions with walls
    if ball_x <= 0 or ball_x >= WIDTH - BALL_SIZE:
        ball_dx = -ball_dx
    if ball_y <= 0:
        ball_dy = -ball_dy
    if ball_y >= HEIGHT:
        game_over = True
    
    # Ball collision with paddle
    paddle_rect = pygame.Rect(paddle_x, paddle_y, PADDLE_WIDTH, PADDLE_HEIGHT)
    ball_rect = pygame.Rect(ball_x, ball_y, BALL_SIZE, BALL_SIZE)
    if ball_rect.colliderect(paddle_rect):
        ball_dy = -abs(ball_dy)  # Ensure ball goes upward
        # Adjust ball direction based on hit position
        hit_pos = (ball_x + BALL_SIZE / 2 - paddle_x) / PADDLE_WIDTH
        ball_dx = 10 * (hit_pos - 0.5)  # Vary horizontal speed
    
    # Ball collision with bricks
    for brick in bricks[:]:
        if ball_rect.colliderect(brick):
            bricks.remove(brick)
            ball_dy = -ball_dy
            score += 10
            break
    
    # Check win condition
    if not bricks:
        game_over = True
    
    # Draw everything
    screen.fill(BLACK)
    
    # Draw paddle
    pygame.draw.rect(screen, BLUE, (paddle_x, paddle_y, PADDLE_WIDTH, PADDLE_HEIGHT))
    
    # Draw ball
    pygame.draw.rect(screen, WHITE, (ball_x, ball_y, BALL_SIZE, BALL_SIZE))
    
    # Draw bricks
    for brick in bricks:
        pygame.draw.rect(screen, RED, brick)
    
    # Draw score
    score_text = font.render(f"Score: {score}", True, WHITE)
    screen.blit(score_text, (10, 10))
    
    # Draw game over or win message
    if game_over:
        message = "You Win!" if not bricks else "Game Over! Press SPACE to restart"
        text = font.render(message, True, WHITE)
        text_rect = text.get_rect(center=(WIDTH / 2, HEIGHT / 2))
        screen.blit(text, text_rect)
    
    pygame.display.flip()

async def main():
    setup()
    clock = pygame.time.Clock()
    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                return
        update_loop()
        await asyncio.sleep(1.0 / 60)  # 60 FPS

if platform.system() == "Emscripten":
    asyncio.ensure_future(main())
else:
    if __name__ == "__main__":
        asyncio.run(main())
