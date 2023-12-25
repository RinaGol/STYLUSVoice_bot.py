from telegram import Update
from telegram.ext import Updater, CommandHandler, MessageHandler, Filters, CallbackContext
from telegram.ext import ConversationHandler

# Замініть 'YOUR_BOT_TOKEN' на токен вашого бота
TOKEN = '6615244918:AAGLLzAH6TzlfvxBa84vPCoFPfOOdzGCg1I'

# Стани для конверсаційного обробника
CHOOSING, PROPOSAL, COMPLAINT = range(3)

# Визначте клавіатуру для вибору типу повідомлення
keyboard = [['Пропозиція', 'Скарга']]

# Словник для зберігання інформації про користувачів
user_data = {}

def start(update: Update, context: CallbackContext) -> int:
    """Початкове повідомлення та вибір типу повідомлення"""
    update.message.reply_text(
        "Привіт! Я STYLUSVoice, твій голос у нашій Компанії.\n"
        "Якщо у тебе є ідеї та пропозиції для покращення роботи, або ти хочеш залишити скаргу на щось або когось - "
        "пиши мені!\n"
        "Вибери тип повідомлення:",
        reply_markup={'keyboard': keyboard, 'one_time_keyboard': True}
    )

    return CHOOSING

def choice(update: Update, context: CallbackContext) -> int:
    """Обробка вибору типу повідомлення"""
    user = update.message.from_user
    text = update.message.text
    user_data['choice'] = text
    update.message.reply_text(f"Ти обрав(-ла): {text}. Тепер напиши своє повідомлення:")

    return PROPOSAL if text == 'Пропозиція' else COMPLAINT

def received_information(update: Update, context: CallbackContext) -> int:
    """Обробка отриманої інформації та відправка відповіді"""
    user = update.message.from_user
    text = update.message.text
    user_data['message'] = text

    # Отримання ніку користувача
    user_nick = user.username if user.username else f"{user.first_name} {user.last_name}"

    # Відправлення відповіді
    update.message.reply_text(
        f"Дякуємо за звернення, {user_nick}!\n"
        "Наша HR-команда найближчим часом зв'яжеться з тобою."
    )

    # Скидання даних користувача
    user_data.clear()

    return ConversationHandler.END

def cancel(update: Update, context: CallbackContext) -> int:
    """Скасування конверсації"""
    update.message.reply_text("Конверсацію скасовано.")
    user_data.clear()

    return ConversationHandler.END

def main():
    """Основна функція"""
    # Створення об'єкта Updater та передача токену
    updater = Updater(TOKEN, use_context=True)

    # Отримання об'єкту диспетчера для реєстрації обробників
    dp = updater.dispatcher

    # Створення конверсаційного обробника
    conv_handler = ConversationHandler(
        entry_points=[CommandHandler('start', start)],
        states={
            CHOOSING: [MessageHandler(Filters.regex('^(Пропозиція|Скарга)$'), choice)],
            PROPOSAL: [MessageHandler(Filters.text & ~Filters.command, received_information)],
            COMPLAINT: [MessageHandler(Filters.text & ~Filters.command, received_information)],
        },
        fallbacks=[CommandHandler('cancel', cancel)],
    )

    # Додавання конверсаційного обробника до диспетчера
    dp.add_handler(conv_handler)

    # Запуск бота
    updater.start_polling()

    # Зупинка бота при натисканні Ctrl+C
    updater.idle()

if __name__ == '__main__':
    main()
