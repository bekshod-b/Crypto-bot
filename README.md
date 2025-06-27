import telebot
import requests
from telebot.types import ReplyKeyboardMarkup, KeyboardButton, WebAppInfo

bot = telebot.TeleBot("8112050451:AAGJyJFlXTlQ0v61btCnHbBXOBLlMY-njgg")  # Tokeningizni yozing

@bot.message_handler(commands=['start'])
def start(message):
    markup = ReplyKeyboardMarkup(resize_keyboard=True)
    markup.add(
        KeyboardButton("🪙 Top 10 Coinlar"),
        KeyboardButton("📊 Bozor statistikasi"),
        KeyboardButton("🌐 Web App", web_app=WebAppInfo(url="https://www.coingecko.com"))
    )
    text = "👋 Salom! Coin nomini yoki symbolini yozing.\nMisol: `Bitcoin` yoki `BTC`"
    bot.send_message(message.chat.id, text, reply_markup=markup)

@bot.message_handler(func=lambda m: True)
def find_coin(message):
    user_input = message.text.lower()

    if user_input == "🪙 top 10 coinlar":
        url = "https://api.coingecko.com/api/v3/coins/markets"
        params = {'vs_currency': 'usd', 'order': 'market_cap_desc', 'per_page': 10, 'page': 1}
        r = requests.get(url, params=params)
        if r.status_code == 200:
            data = r.json()
            msg = "🔝 Top 10 Coinlar:\n\n"
            for coin in data:
                msg += f"{coin['market_cap_rank']}. {coin['name']} ({coin['symbol'].upper()}) - ${coin['current_price']:,}\n"
            bot.send_message(message.chat.id, msg)
        else:
            bot.send_message(message.chat.id, "❌ Ma'lumotlarni olishda xatolik.")
        return

    if user_input == "📊 bozor statistikasi":
        url = "https://api.coingecko.com/api/v3/global"
        r = requests.get(url)
        if r.status_code == 200:
            data = r.json()['data']
            total_market_cap = data['total_market_cap']['usd']
            total_volume = data['total_volume']['usd']
            dominance = data['market_cap_percentage']
            msg = f"🌐 Umumiy bozor statistikasi:\n\n"
            msg += f"💰 Kapitalizatsiya: ${total_market_cap:,.0f}\n"
            msg += f"📊 Hajm (24h): ${total_volume:,.0f}\n"
            msg += f"🔥 Dominans:\n"
            msg += f"   BTC: {dominance['btc']:.2f}%\n"
            msg += f"   ETH: {dominance['eth']:.2f}%\n"
            bot.send_message(message.chat.id, msg)
        else:
            bot.send_message(message.chat.id, "❌ Statistikani olishda xatolik.")
        return

    # Coin izlash (qolgan kod o‘zgarmadi)
    url = f"https://api.coingecko.com/api/v3/search?query={user_input}"
    r = requests.get(url)
    if r.status_code == 200:
        data = r.json()
        if data['coins']:
            coin_id = data['coins'][0]['id']
            coin_data_url = f"https://api.coingecko.com/api/v3/coins/markets"
            params = {'vs_currency': 'usd', 'ids': coin_id}
            response = requests.get(coin_data_url, params=params)
            if response.status_code == 200:
                coin = response.json()[0]
                msg = f"🔍 {coin['name']} ({coin['symbol'].upper()})\n"
                msg += f"💵 Narxi: ${coin['current_price']:,}\n"
                msg += f"📈 24 soatda: {coin['price_change_percentage_24h']:.2f}%\n"
                msg += f"🏦 Kapitalizatsiya: ${coin['market_cap']:,}\n"
                msg += f"📊 Hajm (24h): ${coin['total_volume']:,}\n"
                msg += f"📍 Rank: #{coin['market_cap_rank']}\n"
                msg += f"🔗 CoinGecko: https://www.coingecko.com/en/coins/{coin['id']}"
                bot.send_message(message.chat.id, msg)
            else:
                bot.send_message(message.chat.id, "❌ Coin haqida ma'lumot topilmadi.")
        else:
            bot.send_message(message.chat.id, "❌ Coin topilmadi.")
    else:
        bot.send_message(message.chat.id, "❌ Xatolik yuz berdi. Keyinroq urinib ko‘ring.")

bot.polling()
