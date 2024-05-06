import turtle
import random
import winsound
import time
import threading

turtle.fd(0)
turtle.speed(0)
turtle.bgcolor("black")
turtle.title("SpaceBattle")
turtle.bgpic("background.gif")
turtle.ht()
turtle.setundobuffer(1)
turtle.tracer(0)
turtle.onkey(exit, "q")

class Sprite(turtle.Turtle):
    def __init__(self, spriteshape, color, startx, starty):
        turtle.Turtle.__init__(self, shape=spriteshape)
        self.speed(0)
        self.penup()
        self.color(color)
        self.goto(startx, starty)
        self.speed = 1

    def move(self):
        self.fd(self.speed)

        if self.xcor() > 290:
            self.setx(290)
            self.rt(60)

        if self.xcor() < -290:
            self.setx(-290)
            self.rt(60)

        if self.ycor() > 290:
            self.sety(290)
            self.rt(60)

        if self.ycor() < -290:
            self.sety(-290)
            self.rt(60)

    def is_collision(self, other):
        if (self.xcor() >= (other.xcor() - 20)) and \
                (self.xcor() <= (other.xcor() + 20)) and \
                (self.ycor() >= (other.ycor() - 20)) and \
                (self.ycor() <= (other.ycor() + 20)):
            return True
        else:
            return False


class Player(Sprite):
    def __init__(self, spriteshape, color, startx, starty):
        Sprite.__init__(self, spriteshape, color, startx, starty)
        self.shapesize(stretch_wid=0.6, stretch_len=1.1, outline=None)
        self.speed = 4
        self.lives = 3

    def turn_left(self):
        self.lt(45)

    def turn_right(self):
        self.lt(45)

    def turn_accelerate(self):
        self.speed += 1

    def turn_decelerate(self):
        self.speed -= 1


class Enemy(Sprite):
    def __init__(self, spriteshape, color, startx, starty, initial_speed):
        Sprite.__init__(self, spriteshape,color, startx, starty)
        self.speed = initial_speed
        self.setheading(random.randint(0, 360))
    def move(self):
        self.fd(self.speed)

        # Border checking and changing direction if necessary
        if self.xcor() > 290 or self.xcor() < -290 or \
                self.ycor() > 290 or self.ycor() < -290:
            self.rt(180)  # Turn around if hitting the border
    def update_speed(self, new_speed):
        self.speed = new_speed

class Ally(Sprite):
    def __init__(self, spriteshape, color, startx, starty):
        Sprite.__init__(self, spriteshape,color, startx, starty)
        self.speed = 8
        self.setheading(random.randint(0, 360))

    def move(self):
        self.fd(self.speed)

        if self.xcor() > 290:
            self.setx(290)
            self.lt(60)

        if self.xcor() < -290:
            self.setx(-290)
            self.lt(60)

        if self.ycor() > 290:
            self.sety(290)
            self.lt(60)

        if self.ycor() < -290:
            self.sety(-290)
            self.lt(60)


class Missile(Sprite):
    def __init__(self, spriteshape, color, startx, starty):
        Sprite.__init__(self, spriteshape, color, startx, starty)
        self.shapesize(stretch_wid=0.2, stretch_len=0.4, outline=None)
        self.speed = 20
        self.status = "ready"
        self.goto(-1000, 1000)

    def fire(self):
        if self.status == "ready":
            # play missile sound
            winsound.PlaySound('laser12.wav', winsound.SND_ASYNC)
            self.goto(player.xcor(), player.ycor())
            self.setheading(player.heading())
            self.status = "firing"

    def move(self):

        if self.status == "ready":
            self.goto(-1000, 1000)

        if self.status == "firing":
            self.fd(self.speed)

        if self.xcor() < -290 or self.xcor() > 290 or \
                self.ycor() < -290 or self.ycor() > 290:
            self.goto(-1000, 1000)
            self.status = "ready"


class Particle(Sprite):
    def __init__(self, spriteshape, color, startx, starty):
        Sprite.__init__(self, spriteshape, color, startx, starty)
        self.shapesize(stretch_wid=0.1, stretch_len=0.1, outline=None)
        self.goto(-1000, -1000)
        self.frame = 0

    def explode(self, startx, starty):
        self.goto(startx, starty)
        self.setheading(random.randint(0, 360))
        self.frame = 1

    def move(self):
        if self.frame > 0:
            self.fd(10)
            self.frame += 1
        if self.frame > 15:
            self.frame = 0
            self.goto(-1000, -1000)

def display_message(message, duration):
    turtle.penup()
    turtle.goto(0, 0)
    turtle.color("cyan")
    turtle.write(message, align="center", font=("Arial", 20, "normal"))
        # Define a function to hide the message after the specified duration
    turtle.write(message, align="center", font=("Arial", 20, "normal"))
    # Schedule the clearing of the message after the specified duration
    turtle.ontimer(lambda: turtle.clear(), duration * 1000)




class Game():
    def __init__(self):
        self.level = 1
        self.score = 0
        self.state = "playing"
        self.pen = turtle.Turtle()
        self.lives = 3
        self.enemies = []
        self.allies = []
        self.level_2_enemy_color = "purple"
        self.level_2_ally_color = "cyan"
        self.initial_enemy_speed = 6  # Initial speed for enemies
        self.enemy_speed_increment = 100  # Speed increment per level
        self.score_milestone = 300

    def choose_player_color(self):
        colors = ["white", "pink", "green"]  # List of available colors
        while True:
            chosen_color = turtle.textinput("Choose Player Color!!", "Enter 'white', 'green', or 'pink':").lower()
            if chosen_color in colors:
                return chosen_color
            else:
                print("Invalid color! Please choose from 'white', 'blue', or 'red'.")
    def update_colors(self):
        for enemy in self.enemies:
            if self.level == 2:
                enemy.color("purple")
            else:
                enemy.color("red")

        for ally in self.allies:
            if self.level == 2:
                ally.color("cyan")
            else:
                ally.color("blue")

    def draw_border(self):
        self.pen.speed(0)
        self.pen.color("white")
        self.pen.pensize(3)
        self.pen.penup()
        self.pen.goto(-300, 300)
        self.pen.pendown()
        for side in range(4):
            self.pen.fd(600)
            self.pen.rt(90)
        self.pen.penup()
        self.pen.ht()
        self.pen.pendown()

    def show_status(self):
        self.pen.clear()
        self.pen.undo()
        self.pen.penup()
        self.pen.goto(-300, 310)
        self.pen.write("Level: {}  Score: {}".format(self.level, self.score), font=("Arial", 16, "normal"))

        if self.score >= 300 * self.level:
            self.level += 1
            #self.score -= 100 * (self.level - 1)  # Adjust score for next level
            #self.enemies.append(Enemy("circle", "red", -100, 0))
            self.pen.goto(0, 0)
            self.pen.write("Level Up! You've reached Level {}".format(self.level), align="center",
                           font=("Arial", 24, "normal"))
            time.sleep(1)
            self.update_colors()
            self.update_enemy_speed()
            print("Level increased to:", self.level)

            if self.level == 2:
                turtle.bgpic("background3.gif")  #
            elif self.level == 3:
                turtle.bgpic("background4.gif")
    def create_enemies(self, num_enemies):
        initial_speed = self.initial_enemy_speed + (self.level - 1) * self.enemy_speed_increment  # Increase speed based on level
        for _ in range(num_enemies):
            self.enemies.append(Enemy("circle", "red", random.randint(-250, 250), random.randint(-250, 250), initial_speed))
    def update_enemy_speed(self):
        new_speed = self.initial_enemy_speed + (self.level - 1) * self.enemy_speed_increment
        for enemy in self.enemies:
            enemy.update_speed(new_speed)
        print("Enemy speed updated to:", new_speed)
    def handle_collisions(self):
        # Your existing collision detection code
        for enemy in self.enemies:
            # Handle collision with enemies
            pass

        for ally in self.allies:
            # Handle collision with allies
            pass


game = Game()
game.draw_border()
game.show_status()
player_color = game.choose_player_color() 
player = Player("triangle", player_color, 0, 0)
player.color(player_color)
missile = Missile("triangle", "yellow", 0, 0)
initial_speed_level_1 = 6
enemies = []
for i in range(6):
    enemies.append(Enemy("circle", "red", -100, 0, initial_speed_level_1))
allies = []
for i in range(2):
    allies.append(Ally("square", "blue", 100, 0))
particles = []
for i in range(20):
    particles.append(Particle("circle", "orange", 0, 0))

turtle.onkey(player.turn_left, "Left")
turtle.onkey(player.turn_right, "Right")
turtle.onkey(player.turn_accelerate, "Up")
turtle.onkey(player.turn_decelerate, "Down")
turtle.onkey(missile.fire, "space")
turtle.listen()

while True:
    turtle.update()
    time.sleep(0.03)
    player.move()
    missile.move()

    for enemy in enemies:
        enemy.move()
        if player.is_collision(enemy):
            winsound.PlaySound('explosion12.wav', winsound.SND_ASYNC)
            x = random.randint(-250, 250)
            y = random.randint(-250, 250)
            enemy.goto(x, y)
            game.score -= 100
            game.show_status()
            display_message("Ohh!Bad luck!", 1)  # Display "Wow!" for 2 seconds
           
        if missile.is_collision(enemy):
            winsound.PlaySound('explosion12.wav', winsound.SND_ASYNC)
            x = random.randint(-250, 250)
            y = random.randint(-250, 250)
            enemy.goto(x, y)
            missile.status = "ready"
            game.score += 100
            game.show_status()
            display_message("Wow!", 1)
           
            for particle in particles:
                particle.explode(missile.xcor(), missile.ycor())

    for ally in allies:
        ally.move()
        if missile.is_collision(ally):
            x = random.randint(-250, 250)
            y = random.randint(-250, 250)
            ally.goto(x, y)
            missile.status = "ready"
            game.score -= 50
            game.show_status()

    for particle in particles:
        particle.move()

    if game.score >= 1199:
        turtle.clear()
        turtle.penup()
        turtle.goto(0, 0)
        turtle.color("white")
        winsound.PlaySound('victory.wav', winsound.SND_ASYNC)
        turtle.write("Congratulations! You Win!", align="center", font=("Arial", 30, "normal"))
        break
       

turtle.done()
