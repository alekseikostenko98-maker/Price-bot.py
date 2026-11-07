import telebot

API_TOKEN = '9130492771:AAHHSeIKNQD_HyKEke-rInWpUt_pVm1KZ-k'
bot = telebot.TeleBot(API_TOKEN)

@bot.message_handler(commands=['start', 'help'])
def send_welcome(message):
    bot.reply_to(message, "Привет! Я калькулятор скальпера.\n"
                          "Отправь мне данные в формате: \n"
                          "купил продал баланс пара\n"
                          "Пример: 1.072 1.084 10 gram\n"
                          "Или: 87.08 89.50 0.12 sol")

@bot.message_handler(func=lambda message: True)
def calculate_trade(message):
    try:
        text = message.text.lower().split()
        if len(text) < 4:
            bot.reply_to(message, "❌ Не хватает данных! Введи: `купил продал баланс пара`")
            return

  buy_price = float(text[0])
        sell_price = float(text[1])
        balance = float(text[2])
        pair = text[3]

        # Комиссия: 0.1% для крупных, 0.5% для остальных
   fee_rate = 0.001 if pair in ['sol', 'ton', 'btc', 'eth'] else 0.005

  buy_fee = balance * fee_rate
        crypto_amount = (balance - buy_fee) / buy_price
        
  gross_return = crypto_amount * sell_price
        sell_fee = gross_return * fee_rate
        
  net_return = gross_return - sell_fee
        profit_usdt = net_return - balance
        profit_percent = (profit_usdt / balance) * 100

   status = "🟢 ПЛЮС" if profit_usdt > 0 else "🔴 МИНУС"
        
  response = (
            f"📊 **Анализ сделки ({pair.upper()}/USDT):**\n\n"
            f"Общая комиссия: {buy_fee + sell_fee:.6f} USDT\n"
            f"Чистыми вернется: {net_return:.6f} USDT\n\n"
            f"Результат: {status} {profit_usdt:.6f} USDT ({profit_percent:.2f}%)\n"
        )
        if profit_usdt <= 0:
            response += "\n⚠️ Спред слишком маленький, комиссия сожрала прибыль!"

  bot.reply_to(message, response, parse_mode="Markdown")

   except Exception as e:
        bot.reply_to(message, f"❌ Ошибка в расчетах! Проверь формат ввода.\nТехнический лог: {str(e)}")

bot.infinity_polling()
