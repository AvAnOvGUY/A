from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.button import Button
from kivy.uix.label import Label
from kivy.uix.screenmanager import ScreenManager, Screen
from kivy.uix.gridlayout import GridLayout
from kivy.core.window import Window
from datetime import datetime, timedelta
import json
import os

# Устанавливаем размер окна (для удобства)
Window.size = (400, 600)

# Файл для сохранения данных
DATA_FILE = "saved_data.json"

# Загрузка данных из файла
def load_data():
    if os.path.exists(DATA_FILE):
        with open(DATA_FILE, "r", encoding="utf-8") as file:
            return json.load(file)
    return {}

# Сохранение данных в файл
def save_data(data):
    with open(DATA_FILE, "w", encoding="utf-8") as file:
        json.dump(data, file, ensure_ascii=False, indent=4)

# Главный экран с календарем
class CalendarScreen(Screen):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.layout = BoxLayout(orientation='vertical')
        self.add_widget(self.layout)
        self.current_date = datetime.now()
        self.update_calendar()

    def update_calendar(self):
        self.layout.clear_widgets()

        # Отображение текущего месяца и года
        month_year_label = Label(text=self.current_date.strftime("%B %Y"), size_hint=(1, 0.1))
        self.layout.add_widget(month_year_label)

        # Кнопки для навигации по месяцам
        nav_layout = BoxLayout(size_hint=(1, 0.1))
        prev_button = Button(text="<")
        prev_button.bind(on_press=self.prev_month)
        next_button = Button(text=">")
        next_button.bind(on_press=self.next_month)
        nav_layout.add_widget(prev_button)
        nav_layout.add_widget(next_button)
        self.layout.add_widget(nav_layout)

        # Сетка для календаря
        grid = GridLayout(cols=7, size_hint=(1, 0.7))
        days = ["Пн", "Вт", "Ср", "Чт", "Пт", "Сб", "Вс"]
        for day in days:
            grid.add_widget(Label(text=day))

        # Заполнение календаря
        first_day_of_month = self.current_date.replace(day=1)
        start_day = first_day_of_month.weekday()  # 0 - понедельник, 6 - воскресенье
        for _ in range(start_day):
            grid.add_widget(Label(text=""))

        days_in_month = 31 if self.current_date.month in [1, 3, 5, 7, 8, 10, 12] else 30
        if self.current_date.month == 2:
            days_in_month = 29 if self.current_date.year % 4 == 0 else 28

        for day in range(1, days_in_month + 1):
            date = self.current_date.replace(day=day)
            button = Button(text=str(day))
            button.bind(on_press=lambda instance, dt=date: self.on_date_selected(dt))
            grid.add_widget(button)

        self.layout.add_widget(grid)

    def prev_month(self, instance):
        self.current_date = self.current_date.replace(day=1) - timedelta(days=1)
        self.update_calendar()

    def next_month(self, instance):
        next_month = self.current_date.replace(day=28) + timedelta(days=4)  # Переход на следующий месяц
        self.current_date = next_month.replace(day=1)
        self.update_calendar()

    def on_date_selected(self, date):
        app.screen_manager.current = 'day_screen'
        app.day_screen.set_date(date)

# Экран с кнопками для выбранного дня
class DayScreen(Screen):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.layout = BoxLayout(orientation='vertical')
        self.add_widget(self.layout)
        self.selected_date = None

    def set_date(self, date):
        self.selected_date = date.strftime("%Y-%m-%d")  # Сохраняем дату в формате строки
        self.layout.clear_widgets()
        self.layout.add_widget(Label(text=f"Выбранная дата: {self.selected_date}", size_hint=(1, 0.1)))

        # Загружаем сохраненные данные
        saved_data = load_data()
        date_data = saved_data.get(self.selected_date, {})

        # Создаем 6 кнопок
        buttons = ["Иртэнге", "Ойлэ", "Икенде", "Акшам", "Ясту", "Витр"]
        for button_text in buttons:
            button = Button(text=button_text, size_hint=(1, 0.15))
            # Восстанавливаем состояние кнопки
            if date_data.get(button_text, False):
                button.background_color = [0, 1, 0, 1]  # Зеленый цвет (выделено)
            else:
                button.background_color = [1, 1, 1, 1]  # Белый цвет (сброс)
            button.bind(on_press=lambda instance, btn_text=button_text: self.toggle_button(instance, btn_text))
            self.layout.add_widget(button)

        # Кнопка "Назад"
        back_button = Button(text="Назад", size_hint=(1, 0.1))
        back_button.bind(on_press=self.go_back)
        self.layout.add_widget(back_button)

    def toggle_button(self, instance, button_text):
        # Изменение цвета кнопки при нажатии
        if instance.background_color == [1, 1, 1, 1]:  # Белый цвет (не выделено)
            instance.background_color = [0, 1, 0, 1]  # Зеленый цвет (выделено)
        else:
            instance.background_color = [1, 1, 1, 1]  # Белый цвет (сброс)

        # Сохраняем состояние кнопки
        saved_data = load_data()
        if self.selected_date not in saved_data:
            saved_data[self.selected_date] = {}
        saved_data[self.selected_date][button_text] = instance.background_color == [0, 1, 0, 1]
        save_data(saved_data)

    def go_back(self, instance):
        # Возврат на экран календаря
        app.screen_manager.current = 'calendar_screen'

# Основное приложение
class MyApp(App):
    def build(self):
        self.screen_manager = ScreenManager()

        # Добавляем экраны
        self.calendar_screen = CalendarScreen(name='calendar_screen')
        self.day_screen = DayScreen(name='day_screen')

        self.screen_manager.add_widget(self.calendar_screen)
        self.screen_manager.add_widget(self.day_screen)

        return self.screen_manager

# Запуск приложения
if __name__ == '__main__':
    app = MyApp()
    app.run()
