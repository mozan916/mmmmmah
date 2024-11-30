import telebot
import requests
import json

# أدخل رمز التوكن الخاص بك هنا
TOKEN = "7436703013:AAHE7tRfmIk-HpFQk6FyWL5St1REm4Xhb0I"
bot = telebot.TeleBot(TOKEN)

# وظيفة للحصول على معلومات اللاعب
def get_player_info(player_id):
    url = f'https://rx-ff.vercel.app/info/{player_id}?key=free30day'
    try:
        response = requests.get(url)
        response.raise_for_status()  # Raise exception for bad status codes
        player_data = response.json()
        return player_data
    except requests.exceptions.RequestException as e:
        return f"خطأ أثناء جلب معلومات اللاعب: {e}"
    except json.JSONDecodeError as e:
        return f"خطأ في قراءة استجابة JSON: {e}"
    except Exception as e:
        return f"حدث خطأ غير متوقع: {e}"

# مستمع للأوامر، مثل /start
@bot.message_handler(commands=['start'])
def send_welcome(message):
    bot.reply_to(message, "مرحبًا! أرسل لي معرف اللاعب لتحصل على معلوماته.")

# مستمع للرسائل النصية
@bot.message_handler(func=lambda message: True)
def handle_player_info_request(message):
    player_id = message.text.strip()  # معرف اللاعب
    player_info = get_player_info(player_id)

    if isinstance(player_info, dict):  # إذا كان الرد بيانات صالحة
        player_name = player_info.get('PlayerName', 'لا يوجد اسم')
        player_level = player_info.get('PlayerLevel', 'لا يوجد مستوى')
        player_likes = player_info.get('PlayerLikes', 'لا يوجد إعجابات')
        player_server = player_info.get('PlayerServer', 'لا يوجد خادم')

        response = (
            f"🔹 اسم اللاعب: {player_name}\n"
            f"🔹 مستوى اللاعب: {player_level}\n"
            f"🔹 عدد الإعجابات: {player_likes}\n"
            f"🔹 الخادم: {player_server}"
        )
    else:  # إذا كان الرد رسالة خطأ
        response = player_info

    bot.reply_to(message, response)

# بدء تشغيل البوت
print("البوت يعمل الآن...")
bot.infinity_polling()
