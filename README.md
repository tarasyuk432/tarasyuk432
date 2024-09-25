import logging
import requests
from bs4 import BeautifulSoup
from aiogram import Bot, Dispatcher, types
from aiogram.filters import Command
from aiogram.types import Message, CallbackQuery
from aiogram.utils.keyboard import InlineKeyboardBuilder
import asyncio
import aiohttp
import aiohttp
import logging
from bs4 import BeautifulSoup

API_TOKEN = '8122074942:AAG2bjkNEzsCnG2HuJEIRgcHUyXf7MVZXq4'  # Замените на ваш токен

logging.basicConfig(level=logging.INFO)

bot = Bot(token=API_TOKEN)
dp = Dispatcher()

def search_cars_selenium(brand, model, year):
    url = f"https://www.otomoto.pl/osobowe/bmw/seria-5/seg-sedan?search%5Bfilter_enum_fuel_type%5D=diesel"
    
    # Настройка Selenium
    driver = webdriver.Chrome()
    driver.get(url)
    
    # Даем странице время для загрузки
    driver.implicitly_wait(10)
    
    html = driver.page_source
    soup = BeautifulSoup(html, 'html.parser')
    
    cars = soup.find_all('div', class_='offer-item__info')
    if cars:
        print("Найденные автомобили:")
        for car in cars:
            car_name = car.find('h2', class_='offer-item__title').text.strip()
            print(car_name)
    else:
        print("К сожалению, по вашему запросу ничего не найдено.")
    
    driver.quit()

# Данные для фильтров
MARKI_MODELE = {
    "Aston Martin": [
        "DB11", "DBX", "DBS", "Rapide", "Vantage"
    ],
    "Alfa Romeo": [
        "4C", "Giulia", "Giulietta", "Stelvio"
    ],
    "Audi": [
        "A1", "A3", "A4", "A5", "A6", "A7", "A8", "Q2", "Q3", "Q5", "Q7", "Q8", "R8", "TT"
    ],
    "BMW": [
        "1 Series", "2 Series", "3 Series", "4 Series", "5 Series", "6 Series", "7 Series", "8 Series",
        "X1", "X2", "X3", "X4", "X5", "X6", "X7", "Z4", "i3", "i4", "iX"
    ],
    "Bugatti": [
        "Chiron", "Veyron"
    ],
    "Chevrolet": [
        "Aveo", "Camaro", "Corvette", "Cruze", "Equinox", "Impala", "Malibu", "Silverado", "Traverse"
    ],
    "Chrysler": [
        "300", "Pacifica", "Voyager"
    ],
    "Citroën": [
        "Berlingo", "C1", "C2", "C3", "C4", "C5", "C6", "C3 Aircross", "C5 Aircross"
    ],
    "Dacia": [
        "Duster", "Dokker", "Logan", "Lodgy", "Sandero"
    ],
    "Daihatsu": [
        "Cuore", "Sirion", "Terios"
    ],
    "Ferrari": [
        "488", "F8", "Portofino", "Roma", "SF90"
    ],
    "Fiat": [
        "500", "Doblo", "Fiorino", "Freemont", "Panda", "Tipo"
    ],
    "Ford": [
        "Edge", "Explorer", "Fiesta", "Focus", "Kuga", "Mondeo", "Mustang", "Ranger", "Transit"
    ],
    "Honda": [
        "Accord", "Civic", "CR-V", "Fit", "HR-V", "Jazz", "Pilot", "Ridgeline"
    ],
    "Hyundai": [
        "Creta", "Elantra", "i10", "i20", "i30", "Ioniq", "Kona", "Palaisade", "Santa Fe", "Tucson"
    ],
    "Jaguar": [
        "E-PACE", "F-PACE", "F-Type", "I-PACE", "XE", "XF", "XJ"
    ],
    "Kia": [
        "Niro", "Picanto", "Rio", "Sorrento", "Sportage", "Stonic", "Telluride", "Xceed"
    ],
    "Lancia": [
        "Ypsilon"
    ],
    "Land Rover": [
        "Defender", "Discovery", "Evoque", "Range Rover", "Range Rover Sport", "Velar"
    ],
    "Mazda": [
        "2", "3", "6", "CX-30", "CX-3", "CX-5", "CX-50", "CX-60", "MX-5"
    ],
    "Mercedes-Benz": [
        "A-Class", "B-Class", "C-Class", "CLA", "CLS", "E-Class", "EQC", "EQS", "G-Class", "GLA",
        "GLB", "GLC", "GLE", "GLS", "S-Class", "SL", "SLC", "X-Class", "8 Series"
    ],
    "Mini": [
        "Mini", "Mini Clubman", "Mini Countryman", "Mini Electric"
    ],
    "Mitsubishi": [
        "ASX", "Eclipse Cross", "Outlander", "Pajero", "Space Star"
    ],
    "Nissan": [
        "370Z", "Juke", "Leaf", "Micra", "Navara", "Qashqai", "Rogue", "X-Trail"
    ],
    "Peugeot": [
        "2008", "208", "3008", "308", "5008", "508", "Partner", "Expert"
    ],
    "Porsche": [
        "718", "Cayenne", "F8", "Macan", "Panamera", "Taycan", "911"
    ],
    "Renault": [
        "Captur", "Clio", "Kadjar", "Koleos", "Megane", "Talisman", "Zoe"
    ],
    "Rover": [
        "75", "Freelander", "Range Rover"
    ],
    "Seat": [
        "Arona", "Ibiza", "Leon", "Tarraco", "Ateca"
    ],
    "Skoda": [
        "Fabia", "Karoq", "Kodiaq", "Octavia", "Superb", "Enyaq"
    ],
    "Smart": [
        "Fortwo", "Forfour"
    ],
    "SsangYong": [
        "Korando", "Rexton", "Tivoli"
    ],
    "Subaru": [
        "Ascent", "Forester", "Impreza", "Legacy", "Outback", "XV"
    ],
    "Tesla": [
        "Model 3", "Model S", "Model X", "Model Y"
    ],
    "Toyota": [
        "Aygo", "Camry", "C-HR", "Corolla", "Hilux", "Land Cruiser", "Mirai", "Prius", "RAV4", "Yaris"
    ],
    "Volkswagen": [
        "Arteon", "Golf", "ID.3", "ID.4", "Polo", "Passat", "Tiguan", "Touareg", "Jetta"
    ],
    "Volvo": [
        "C40", "S60", "S90", "V40", "V60", "V90", "XC40", "XC60", "XC90"
    ],
    "ZAZ": [
        "Sens", "Tavria"
    ],
    "MG": [
        "MG3", "MG6", "ZS"
    ],
    "Koenigsegg": [
        "Agera", "Jesko", "Regera"
    ],
    "Pagani": [
        "Huayra", "Zonda"
    ],
}

NADWOZIE = ['Sedan', 'Kombi', 'SUV', 'Hatchback', 'Coupe', 'Minivan']
PALIWO = ['Benzyna', 'Diesel', 'Elektryczny', 'Hybryda', 'LPG']
SKRZYNIA = ['Automatyczna', 'Manualna']
NAPED = ['Przedni', 'Tylny', '4x4']
KOLOR = ['Czarny', 'Biały', 'Czerwony', 'Niebieski', 'Szary', 'Zielony', 'Żółty']

# Хранение фильтров для каждого пользователя
user_filters = {}

# Стартовая команда /start
@dp.message(Command("start"))
async def send_welcome(message: Message):
    user_filters[message.from_user.id] = {}  # Инициализация фильтров для пользователя
    await message.answer(
        "Witaj! Wybierz markę samochodu, aby rozpocząć wyszukiwanie.",
        reply_markup=create_marki_keyboard()
    )

# Создание клавиатуры для выбора марки
def create_marki_keyboard():
    keyboard = InlineKeyboardBuilder()
    for marka in MARKI_MODELE.keys():
        keyboard.button(text=marka, callback_data=f"marka:{marka}")
    return keyboard.as_markup()

# Создание клавиатуры для выбора модели
def create_modele_keyboard(marka):
    keyboard = InlineKeyboardBuilder()
    for model in MARKI_MODELE.get(marka, []):  # Защита от KeyError
        keyboard.button(text=model, callback_data=f"model:{marka}:{model}")
    return keyboard.as_markup()

# Обработка выбора марки
@dp.callback_query(lambda call: call.data.startswith("marka:"))
async def handle_marka(call: CallbackQuery):
    marka = call.data.split(":")[1]
    user_filters[call.from_user.id]['marka'] = marka
    await call.message.answer(f"Wybrałeś: {marka}. Teraz wybierz model.", reply_markup=create_modele_keyboard(marka))

# Обработка выбора модели
@dp.callback_query(lambda call: call.data.startswith("model:"))
async def handle_model(call: CallbackQuery):
    model = call.data.split(":")[2]
    user_filters[call.from_user.id]['model'] = model
    await call.message.answer(f"Wybrałeś model: {model}. Теперь выберите тип кузова.", reply_markup=create_nadwozie_keyboard())

# Создание клавиатуры для выбора типа кузова
def create_nadwozie_keyboard():
    keyboard = InlineKeyboardBuilder()
    nadwozie_options = ['Купе', 'Седан', 'Универсал', 'Хэтчбек', 'Кроссовер', 'Минивэн']
    for option in nadwozie_options:
        keyboard.button(text=option, callback_data=f"nadwozie:{option}")
    return keyboard.as_markup()

# Обработка выбора типа кузова
@dp.callback_query(lambda call: call.data.startswith("nadwozie:"))
async def handle_nadwozie(call: CallbackQuery):
    nadwozie = call.data.split(":")[1]
    user_filters[call.from_user.id]['nadwozie'] = nadwozie
    await call.message.answer(f"Wybrałeś тип кузова: {nadwozie}. Теперь выберите тип топлива.", reply_markup=create_paliwo_keyboard())

# Создание клавиатуры для выбора типа топлива
def create_paliwo_keyboard():
    keyboard = InlineKeyboardBuilder()
    paliwo_options = ['Бензин', 'Дизель', 'Электро', 'Гибрид']
    for option in paliwo_options:
        keyboard.button(text=option, callback_data=f"paliwo:{option}")
    return keyboard.as_markup()

# Обработка выбора типа топлива
@dp.callback_query(lambda call: call.data.startswith("paliwo:"))
async def handle_paliwo(call: CallbackQuery):
    paliwo = call.data.split(":")[1]
    user_filters[call.from_user.id]['paliwo'] = paliwo
    await call.message.answer(f"Wybrałeś тип топлива: {paliwo}. Теперь выберите год выпуска.", reply_markup=create_rok_keyboard())

# Создание клавиатуры для выбора года выпуска
def create_rok_keyboard():
    keyboard = InlineKeyboardBuilder()
    rok_options = ['2024', '2023', '2022', '2021', '2020']
    for option in rok_options:
        keyboard.button(text=option, callback_data=f"rok:{option}")
    return keyboard.as_markup()

# Обработка выбора года выпуска
@dp.callback_query(lambda call: call.data.startswith("rok:"))
async def handle_rok(call: CallbackQuery):
    rok = call.data.split(":")[1]
    user_filters[call.from_user.id]['rok'] = rok
    await call.message.answer(f"Wybrałeś год выпуска: {rok}. Начинаю поиск автомобилей...")

    # Получение фильтров
    marka = user_filters[call.from_user.id]['marka']
    model = user_filters[call.from_user.id]['model']

    # Выполнение поиска автомобилей
    await search_cars(marka, model, call.message)

# Асинхронный метод для поиска автомобилей
async def search_cars(brand, model, message):
    # Формируем запрос к API или парсинг сайта
    url = f"https://example.com/api/cars?brand={brand}&model={model}"
    try:
        async with aiohttp.ClientSession() as session:  # Используем контекстный менеджер для сессии
            response = await session.get(url)
            response.raise_for_status()  # Проверка на ошибки HTTP
            data = await response.json()
            
            if data:
                # Обработка данных и отправка пользователю
                await message.answer("Найденные автомобили:")
                for car in data:
                    await message.answer(car['name'])
            else:
                await message.answer("К сожалению, по вашему запросу ничего не найдено.")
    except Exception as e:
        logging.error(f"Ошибка при поиске автомобилей: {e}")
        await message.answer("Произошла ошибка. Попробуйте позже.")

# Запуск бота
if __name__ == '__main__':
    logging.info("Запуск бота...")
    try:
        asyncio.run(dp.start_polling(bot))
    except Exception as e:
        logging.error(f"Ошибка при запуске бота: {e}")

<!---
tarasyuk432/tarasyuk432 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
