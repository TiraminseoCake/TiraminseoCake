import tkinter as tk
import turtle, random, time

class RunawayGame:
    def __init__(self, canvas, runner, chaser, catch_radius=50, game_duration=30):
        self.canvas = canvas
        self.runner = runner
        self.chaser = chaser
        self.catch_radius2 = catch_radius**2
        self.game_duration = game_duration  
        self.score = 0 
        self.start_time = None
        self.runner.shape('turtle')
        self.runner.color('blue')
        self.runner.penup()
        self.chaser.shape('turtle')
        self.chaser.color('red')
        self.chaser.penup()
        self.drawer = turtle.RawTurtle(canvas)
        self.drawer.hideturtle()
        self.drawer.penup()
    def is_catched(self):
        p = self.runner.pos()
        q = self.chaser.pos()
        dx, dy = p[0] - q[0], p[1] - q[1]
        return dx**2 + dy**2 < self.catch_radius2
    def start(self, init_dist=400, ai_timer_msec=100):
        # 초기 위치 설정
        self.runner.setpos((-init_dist / 2, 0))
        self.runner.setheading(0)
        self.chaser.setpos((+init_dist / 2, 0))
        self.chaser.setheading(180)
        self.start_time = time.time()  # 게임 시작 시간 저장
        self.ai_timer_msec = ai_timer_msec
        self.canvas.ontimer(self.step, self.ai_timer_msec)
    def step(self):
        elapsed_time = time.time() - self.start_time
        remaining_time = int(self.game_duration - elapsed_time)
        if remaining_time <= 0:
            self.end_game()
            return
        self.runner.run_ai(self.chaser.pos(), self.chaser.heading())
        self.chaser.run_ai(self.runner.pos(), self.runner.heading())
        is_catched = self.is_catched()
        if is_catched:
            self.score -= 1 
        else:
            self.score += 1  
        self.drawer.undo()
        self.drawer.penup()
        self.drawer.setpos(-300, 300)
        self.drawer.write(f'Score: {self.score} | Time left: {remaining_time}s', font=("Arial", 16, "normal"))
        self.canvas.ontimer(self.step, self.ai_timer_msec)
    def end_game(self):
        self.drawer.clear()
        self.drawer.penup()
        self.drawer.setpos(-150, 0)
        self.drawer.write(f'Game Over! Final Score: {self.score}', font=("Arial", 24, "bold"))
        self.runner.hideturtle()
        self.chaser.hideturtle()


class ManualMover(turtle.RawTurtle):
    def __init__(self, canvas, step_move=10, step_turn=10):
        super().__init__(canvas)
        self.step_move = step_move
        self.step_turn = step_turn
        canvas.onkeypress(lambda: self.forward(self.step_move), 'Up')
        canvas.onkeypress(lambda: self.backward(self.step_move), 'Down')
        canvas.onkeypress(lambda: self.left(self.step_turn), 'Left')
        canvas.onkeypress(lambda: self.right(self.step_turn), 'Right')
        canvas.listen()
    def run_ai(self, opp_pos, opp_heading):
        pass


class RandomMover(turtle.RawTurtle):
    def __init__(self, canvas, step_move=10, step_turn=10):
        super().__init__(canvas)
        self.step_move = step_move
        self.step_turn = step_turn
    def run_ai(self, opp_pos, opp_heading):
        mode = random.randint(0, 2)
        if mode == 0:
            self.forward(self.step_move)
        elif mode == 1:
            self.left(self.step_turn)
        elif mode == 2:
            self.right(self.step_turn)

if __name__ == '__main__':
    root = tk.Tk()
    canvas = tk.Canvas(root, width=700, height=700)
    canvas.pack()
    screen = turtle.TurtleScreen(canvas)
    screen.bgcolor("#FF5733")  #배경색 변경
    runner = RandomMover(screen)
    chaser = ManualMover(screen)
    game = RunawayGame(screen, runner, chaser)
    game.start()
    root.mainloop()
