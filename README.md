# captchabotaiogram
Мне нужна помощь с кодом, если вы поняли ошибки пишите @fennixexe в телеграмм
Если надо объяснить в чем проблема пишите туда же

import random
import asyncio
from aiogram import Bot, Dispatcher, types

# Создание бота
bot = Bot(token='')
dp = Dispatcher(bot)

# Словарь для хранения статуса пользователей (новый или не новый)
users = {}
# Словарь для хранения таймеров
timers = {}

def restart_kick_timer(user_id, chat_id):
    # Ваш код для перезапуска таймера выгнания пользователя
    # Например:
    if user_id in timers:
        timer = timers[user_id]['timer']
        timer.cancel()
        del timers[user_id]

    send_captcha(user_id, chat_id)

# Обработчик новых сообщений
@dp.message_handler(content_types=types.ContentType.TEXT)
async def handle_message(message: types.Message):
    user_id = message.from_user.id
    chat_id = message.chat.id

    if user_id not in users:
        # Если пользователь новый, отправляем капчу
        await send_captcha(user_id, chat_id)
        users[user_id] = True
    else:
        # Ваша логика для обработки сообщений от уже проверенных пользователей
        if user_id in timers:
            # Удаляем сообщение пользователя
            await message.delete()
            # Перезапускаем таймер для выгнания пользователя
            await restart_kick_timer(user_id, chat_id)


async def send_captcha(user_id, chat_id):
    if user_id not in users:
        captcha = generate_captcha()  # Генерация капчи
        markup = create_captcha_markup(captcha)  # Создание клавиатуры для капчи

        # Отправка сообщения с капчей
        message = await bot.send_message(chat_id, "Для подтверждения, решите следующий пример:")
        await bot.send_message(chat_id, captcha['question'], reply_markup=markup)

        # Запуск таймера для выгнания пользователя
        timer = asyncio.TimerHandle(15, kick_user, user_id, chat_id)
        timers[user_id] = {'timer': timer, 'message_id': message.message_id}
        loop = asyncio.get_running_loop()
        loop.call_later(15, kick_user, user_id, chat_id)

def generate_captcha():
    num1 = random.randint(1, 10)
    num2 = random.randint(1, 10)
    operator = random.choice(['+', '-', '*'])
    question = f"{num1} {operator} {num2} = ?"

    if operator == '+':
        answer = num1 + num2
    elif operator == '-':
        answer = num1 - num2
    else:
        answer = num1 * num2

    captcha = {'question': question, 'answer': answer}
    return captcha

# Создание клавиатуры для капчи
def create_captcha_markup(captcha):
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    answers = generate_answer_options(captcha['answer'])

    # Создание кнопок с вариантами ответов
    markup.row(*answers[:3])
    markup.row(*answers[3:])

    return markup

# Генерация вариантов ответов
def generate_answer_options(answer):
    answers = [str(answer)]

    # Генерация трёх неправильных вариантов ответов
    while len(answers) < 4:
        wrong_answer = str(answer + random.randint(-5, 5))
        if wrong_answer != str(answer) and wrong_answer not in answers:
            answers.append(wrong_answer)
            random.shuffle(answers)
        return answers


async def kick_user(user_id, chat_id):
  if user_id in users:
    await bot.kick_chat_member(chat_id, user_id)
  del timers[user_id]

async def restart_kick_timer(user_id, chat_id):
  if user_id in timers:
   timer = timers[user_id]['timer']
   timer.cancel()
  del timers[user_id]

def kick_user(user_id, chat_id):
    if user_id in users:
        bot.kick_chat_member(chat_id, user_id)
        del timers[user_id]

if __name__ == '__main__':
  loop = asyncio.get_event_loop()
try:
  loop.create_task(dp.start_polling())
  loop.run_forever()
finally:
  loop.stop()
