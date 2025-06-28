from telegram.ext import ApplicationBuilder, MessageHandler, CommandHandler, filters
from telegram import Update
from telegram.ext import ContextTypes
import requests
import os

# 🔐 API kalitlar
API_TOKEN = "8049470097:AAF_LftadIsI5pg4NTAcXyHQxvGhtViuIig"
REMOVE_BG_API = "AJ4NnXuSm8FmbKiYefDPRmNy"

# /start komandasi
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("👋 Salom! Menga rasm yuboring — men fonini olib tashlayman 😊")

# Fonni remove.bg orqali olib tashlash
def remove_background(image_path):
    with open(image_path, 'rb') as img:
        response = requests.post(
            'https://api.remove.bg/v1.0/removebg',
            files={'image_file': img},
            data={'size': 'auto'},
            headers={'X-Api-Key': REMOVE_BG_API}
        )
    if response.status_code == 200:
        return response.content
    else:
        print("❌ Xatolik:", response.status_code, response.text)
        return None

# Rasm kelganda ishlaydigan funksiya
async def handle_photo(update: Update, context: ContextTypes.DEFAULT_TYPE):
    photo_file = await update.message.photo[-1].get_file()
    photo_path = "input.jpg"
    await photo_file.download_to_drive(photo_path)

    await update.message.reply_text("⏳ Rasm tahlil qilinmoqda, kuting...")

    result = remove_background(photo_path)
    if result:
        await update.message.reply_photo(photo=result, caption="✅ Fon olib tashlandi!")
    else:
        await update.message.reply_text("❌ Xatolik yuz berdi, keyinroq urinib ko‘ring.")

# Botni ishga tushurish
if __name__ == '__main__':
    app = ApplicationBuilder().token(API_TOKEN).build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(MessageHandler(filters.PHOTO, handle_photo))

    print("🤖 Bot ishga tushdi...")
    app.run_polling()
