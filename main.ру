import random
import telebot
from telebot import types
import firebase_admin
from firebase_admin import credentials, db

# Firebase ulanishi - faqat fayl nomi ko'rsatiladi
cred = credentials.Certificate("serviceAccountKey.json")
firebase_admin.initialize_app(cred, {
    'databaseURL': 'https://sehrli-abakus-default-rtdb.firebaseio.com/'
})

TOKEN = "8733167513:AAGrrlP_cpLODhyManCGBnegG5Yv8lMM4c0"
bot = telebot.TeleBot(TOKEN)

user_data = {}

@bot.message_handler(commands=['start'])
def send_welcome(message):
    user_id = str(message.from_user.id)
    name = message.from_user.first_name
    
    ref = db.reference(f'users/{user_id}')
    user_snapshot = ref.get()
    
    if not user_snapshot:
        ref.set({
            'name': name,
            'score': 0,
            'level': '5-12 yosh'
        })
    
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    markup.add(types.KeyboardButton("🧮 Misol yechish"), types.KeyboardButton("🏆 Reyting"))
    
    bot.send_message(
        message.chat.id, 
        f"Salom, {name}! 🌟 Sehrli Abakus mental arifmetika olamiga xush kelibsiz!\n"
        "Matematik qahramon bo'lish uchun pastdagi tugmani bosing!", 
        reply_markup=markup
    )

@bot.message_handler(func=lambda message: message.text == "🧮 Misol yechish")
def start_math(message):
    num1 = random.randint(1, 30)
    num2 = random.randint(1, 15)
    operator = random.choice(['+', '-'])
    
    if operator == '-' and num1 < num2:
        num1, num2 = num2, num1
        
    correct_answer = num1 + num2 if operator == '+' else num1 - num2
    user_data[message.from_user.id] = correct_answer
    
    bot.send_message(message.chat.id, f"Tezkor misol: **{num1} {operator} {num2} = ?**", parse_mode="Markdown")

@bot.message_handler(func=lambda message: message.text.isdigit() or (message.text.startswith('-') and message.text[1:].isdigit()))
def check_answer(message):
    user_id = message.from_user.id
    if user_id not in user_data:
        bot.send_message(message.chat.id, "Iltimos, avval 'Misol yechish' tugmasini bosing.")
        return
        
    user_answer = int(message.text)
    correct_answer = user_data[user_id]
    
    ref = db.reference(f'users/{user_id}')
    user_data_db = ref.get()
    current_score = user_data_db.get('score', 0) if user_data_db else 0
    
    if user_answer == correct_answer:
        new_score = current_score + 10
        ref.update({'score': new_score})
        bot.send_message(message.chat.id, f"🎉 Barakalla! To'g'ri javob!\nSizning umumiy ballingiz: **{new_score}** ball", parse_mode="Markdown")
    else:
        bot.send_message(message.chat.id, f"❌ Xato. To'g'ri javob {correct_answer} edi. Yana bir bor urinib ko'ring!")
        
    start_math(message)

@bot.message_handler(func=lambda message: message.text == "🏆 Reyting")
def show_leaderboard(message):
    ref = db.reference('users')
    users = ref.order_by_child('score').limit_to_last(5).get()
    
    text = "🏆 **Eng chaqqon bolalar reytingi:**\n\n"
    
    if users:
        sorted_users = sorted(users.items(), key=lambda x: x[1].get('score', 0), reverse=True)
        rank = 1
        for uid, data in sorted_users:
            text += f"{rank}. {data.get('name', 'Nomaʼlum')} — {data.get('score', 0)} ball\n"
            rank += 1
    else:
        text += "Hozircha o'yinchilar yo'q."
        
    bot.send_message(message.chat.id, text, parse_mode="Markdown")

if __name__ == "__main__":
    print("Bot ishga tushdi...")
    bot.infinity_polling()
