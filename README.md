import kivy
kivy.require('2.1.0')

from kivy.app import App
from kivy.uix.widget import Widget
from kivy.uix.label import Label
from kivy.uix.button import Button
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.floatlayout import FloatLayout
from kivy.graphics import Rectangle, Color
from kivy.clock import Clock
from kivy.core.window import Window

import random

# Конфигурация
SCREEN_WIDTH = 800
SCREEN_HEIGHT = 600
CAR_WIDTH = 60
CAR_HEIGHT = 100
LANE_WIDTH = SCREEN_WIDTH // 3

class Car:
    def __init__(self, x, y, color):
        self.x = x
        self.y = y
        self.color = color
        self.speed = 5
        self.fuel = 100
        self.coins = 0

    def move_left(self):
        if self.x > LANE_WIDTH:
            self.x -= LANE_WIDTH

    def move_right(self):
        if self.x < 2 * LANE_WIDTH:
            self.x += LANE_WIDTH

    def update(self):
        self.y += self.speed
        self.fuel -= 0.05
        if self.y > SCREEN_HEIGHT:
            self.y = -CAR_HEIGHT

class Obstacle:
    def __init__(self):
        self.x = random.choice([LANE_WIDTH, 2 * LANE_WIDTH])
        self.y = -100
        self.width = 40
        self.height = 60
        self.color = (0.8, 0.2, 0.2)  # Красный

    def update(self):
        self.y += 5
        return self.y > SCREEN_HEIGHT

class GameWidget(Widget):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.car = Car(LANE_WIDTH, SCREEN_HEIGHT // 2, (0, 0, 1))  # Синий
        self.obstacles = []
        self.score = 0
        self.game_over = False

        # Интерфейс
        self.fuel_label = Label(text=f"Топливо: {int(self.car.fuel)}", pos=(10, SCREEN_HEIGHT - 40))
        self.coins_label = Label(text=f!Монеты: {self.car.coins}", pos=(10, SCREEN_HEIGHT - 80))
        self.add_widget(self.fuel_label)
        self.add_widget(self.coins_label)

        # Кнопки управления
        self.left_btn = Button(text="←", size_hint=(None, None), size=(80, 80), pos=(50, 50))
        self.right_btn = Button(text="→", size_hint=(None, None), size=(80, 80), pos=(SCREEN_WIDTH - 130, 50))
        self.add_widget(self.left_btn)
        self.add_widget(self.right_btn)

        self.left_btn.bind(on_press=self.car.move_left)
        self.right_btn.bind(on_press=self.car.move_right)

        Clock.schedule_interval(self.update, 1.0 / 60.0)

    def update(self, dt):
        if not self.game_over:
            self.car.update()

            # Генерация препятствий
            if random.randint(1, 100) < 3:
                self.obstacles.append(Obstacle())

            # Обновление препятствий
            for obs in self.obstacles:
                if obs.update():
                    self.obstacles.remove(obs)
                    self.score += 10
                    self.car.coins += 1

            # Проверка столкновений
            for obs in self.obstacles:
                if (abs(self.car.x - obs.x) < CAR_WIDTH // 2 + obs.width // 2 and
                        abs(self.car.y - obs.y) < CAR_HEIGHT // 2 + obs.height // 2):
                    self.game_over = True

            # Проверка топлива
            if self.car.fuel <= 0:
                self.game_over = True

            # Обновление интерфейса
            self.fuel_label.text = f"Топливо: {int(self.car.fuel)}"
            self.coins_label.text = f!Монеты: {self.car.coins}"

        else:
            self.fuel_label.text = "Игра окончена! (Пробел для перезапуска)"

    def on_key_down(self, key, scancode, codepoint, modifier):
        if key == 32 and self.game_over:  # Пробел
            self.restart()

    def restart(self):
        self.car = Car(LANE_WIDTH, SCREEN_HEIGHT // 2, (0, 0, 1))
        self.obstacles = []
        self.score = 0
        self.game_over = False

class StoreWidget(Widget):
    def __init__(self, car, **kwargs):
        super().__init__(**kwargs)
        self.car = car
        layout = BoxLayout(orientation='vertical', padding=20, spacing=10)

        title = Label(text="Магазин", font_size=24)
        layout.add_widget(title)

        # Покупка монет
        btn_coins = Button(text="Купить 100 монет за $0.99", size_hint_y=None, height=60)
        btn_coins.bind(on_press=self.buy_coins)
        layout.add_widget(btn_coins)

        # Список машин
        for i, color in enumerate([(1, 0, 0), (0, 1, 0), (1, 1, 0), (0.5, 0, 0.5)]):
            btn_car = Button(
                text=f!Машина {i+1} (50 монет)",
                size_hint_y=None,
                height=60,
                background_color=color
            )
            btn_car.bind(on_press=lambda btn, c=color: self.buy_car(c))
            layout.add_widget(btn_car)

        self.add_widget(layout)

    def buy_coins(self, instance):
        self.car.coins += 100
        print("Куплено 100 монет!")

    def buy_car(self, color):
        if self.car.coins >= 50:
            self.car.coins -= 50
            self.car.color = color
            print("Машина куплена!")
        else:
            print("Не хватает монет!")

class RacingApp(App):
    def build(self):
        Window.size = (SCREEN_WIDTH, SCREEN_HEIGHT)
        
        root = FloatLayout()
        game = GameWidget()
        store = StoreWidget(game.car)
        
        # Переключение между экранами
        def show_store():
            root.clear_widgets()
            root.add_widget(store)
        
        def show_game():
            root.clear_widgets()
            root.add_widget(game)
        
        game.left_btn.text = "← (Магазин)"
        game.left_btn.unbind(on_press)
        game.left_btn.bind(on_press=show_store)
        
        store_btn = Button(text="←
