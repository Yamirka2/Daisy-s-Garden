import random
import time
import turtle

turtle.register_shape("smaller_heart.gif")

# settings for cannon, laser and aliens
FRAME_RATE = 30  # Frames per second
TIME_FOR_1_FRAME = 1 / FRAME_RATE  # Seconds

#setting up the different sizes and speed
CANNON_STEP = 10
LASER_LENGTH = 20
LASER_SPEED = 60
ALIEN_SPAWN_INTERVAL = 1  # Seconds
ALIEN_SPEED = 2.5
MAX_HEALTH = 3  # Maximum health value for the player

# set up windows for initial game
window = turtle.Screen()
window.tracer(0)
window.setup(0.5,  0.95)

#insterting the gifs for the turtles
window.bgpic("gardenbackground.png")
window.title("Daisy's Garden")

# establishing the different phrase's and what they mean for positioning in the window
LEFT = -window.window_width() / 2
RIGHT = window.window_width() / 2
TOP = window.window_height() / 2
BOTTOM = -window.window_height() / 2
FLOOR_LEVEL = 0.9 * BOTTOM
GUTTER = 0.025 * window.window_width()

#importing images
turtle.register_shape("pixeldaisy.gif")
turtle.register_shape("vines.gif")

# Create laser cannon
cannon = turtle.Turtle()
cannon.penup()
cannon.color(1, 1, 1)
cannon.shape("pixeldaisy.gif")
cannon.setposition(0, FLOOR_LEVEL + 145)
cannon.cannon_movement = 0  # -1, 0 or 1 for left, stationary, right

# Create lives indicator (hearts)
lives = []
for i in range(MAX_HEALTH):
    life = turtle.Turtle()
    life.penup()
    life.color("red")
    life.shape("smaller_heart.gif")
    life.setposition(RIGHT - 30 - i * 30, TOP - 30)
    life.showturtle()
    lives.append(life)

# Create turtle for writing text
text = turtle.Turtle()
text.penup()
text.hideturtle()
text.setposition(LEFT * 0.8, TOP * 0.8)
text.color(1, 1, 1)

lasers = []
aliens = []

player_lives = MAX_HEALTH


def move_left():  #cannon movement left
    cannon.cannon_movement = -1


def move_right():#cannon movement right
    cannon.cannon_movement = 1


def stop_cannon_movement():#cannon stopping
    cannon.cannon_movement = 0

#creating a laser
def create_laser():
    laser = turtle.Turtle()
    laser.penup()
    laser.color(0, 0, 1)
    laser.hideturtle()
    laser.setposition(cannon.xcor() + 90, cannon.ycor() + 15) #positioning of laser
    laser.setheading(90)
    # Move laser to just above cannon tip
    laser.forward(10)
    # Prepare to draw the laser
    laser.pendown()
    laser.pensize(7)

    lasers.append(laser)

#establishing the movement of the laser
def move_laser(laser):
    laser.clear()
    laser.forward(LASER_SPEED)
    # Draw the laser
    laser.forward(LASER_LENGTH)
    laser.forward(-LASER_LENGTH)

#creating the vines and there movement
def create_alien():
    alien = turtle.Turtle()
    alien.penup()
    alien.turtlesize(0.05)
    alien.setposition(
        random.randint(
            int(LEFT + 100),
            int(RIGHT - 100),
        ),
        TOP,
    )

    alien.shape("vines.gif")
    alien.setheading(-90)
    aliens.append(alien)

#removing the turtles when they go off screen
def remove_sprite(sprite, sprite_list):
    sprite.clear()
    sprite.hideturtle()
    window.update()
    sprite_list.remove(sprite)
    turtle.turtles().remove(sprite)


# Key bindings
window.onkeypress(move_left, "Left")
window.onkeypress(move_right, "Right")
window.onkeyrelease(stop_cannon_movement, "Left")
window.onkeyrelease(stop_cannon_movement, "Right")
window.onkeypress(create_laser, "space")
window.onkeypress(turtle.bye, "q")
window.listen()

# Game loop
alien_timer = 0
game_timer = time.time()
score = 0
game_running = True
while game_running and player_lives > 0:
    timer_this_frame = time.time()

    time_elapsed = time.time() - game_timer
    text.clear()
    text.write(
        f"Time: {time_elapsed:5.1f}s\nScore: {score:5}",
        font=("Courier", 20, "bold"),
    )
    # Move cannon
    new_x = cannon.xcor() + CANNON_STEP * cannon.cannon_movement
    if LEFT + GUTTER <= new_x <= RIGHT - GUTTER:
        cannon.setx(new_x)

    # Move all lasers
    for laser in lasers.copy():
        move_laser(laser)
        # Remove laser if it goes off screen
        if laser.ycor() > TOP:
            remove_sprite(laser, lasers)
            break
        # Checking for collision with aliens
        for alien in aliens.copy():
            if laser.distance(alien) < 20:
                remove_sprite(laser, lasers)
                remove_sprite(alien, aliens)
                score += 1
                break
    # Spawn new aliens when time interval elapsed
    if time.time() - alien_timer > ALIEN_SPAWN_INTERVAL:
        create_alien()
        alien_timer = time.time()

    # Move all aliens LIFE HEART!!!
    for alien in aliens:
        alien.forward(ALIEN_SPEED)
        # Check for game over
        if alien.ycor() < FLOOR_LEVEL:
            player_lives -= 1
            if player_lives == -1:
                game_running = False
            else:
                # Remove a heart from the display
                life = lives.pop()
                life.clear()
                life.hideturtle()
            remove_sprite(alien, aliens)

    # Check for collision with player
    for alien in aliens.copy():
        if alien.distance(cannon) < 20:  # Adjust collision distance as needed
            player_lives -= 1  # Decrease player's lives upon collision
            if player_lives == -1:
                game_running = False
            else:
                # Remove a heart from the display
                life = lives.pop()
                life.clear()
                life.hideturtle()

            remove_sprite(alien, aliens)
     # the timer
    time_for_this_frame = time.time() - timer_this_frame
    if time_for_this_frame < TIME_FOR_1_FRAME:
        time.sleep(TIME_FOR_1_FRAME - time_for_this_frame)
    window.update()

#putting in the game over screen when the game ends
splash_text = turtle.Turtle()
splash_text.hideturtle()
splash_text.color(1, 1, 1)
splash_text.write("GAME OVER", font=("Courier", 40, "bold"), align="center")

#finshes the program and puts it on the screen to play
turtle.done()
window.update()
