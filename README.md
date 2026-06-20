import asyncio
import logging
from aiogram import Bot, Dispatcher, F
from aiogram.filters import CommandStart
from aiogram.types import (
    Message, CallbackQuery,
    InlineKeyboardMarkup, InlineKeyboardButton,
)
from aiogram.fsm.context import FSMContext
from aiogram.fsm.state import State, StatesGroup
from aiogram.fsm.storage.memory import MemoryStorage


BOT_TOKEN = "8774696772:AAH9GWKTA-zvK1ZI51nRU1hPbKdEonPoSc4"
CHANNEL_USERNAME = "@Standfire3"
CHANNEL_INVITE = "https://t.me/Standfire3"
SUPPORT_USERNAME = "xeqcf"
ADMIN_ID = 8517997552
PRIVATE_LINK = "https://t.me/+TUfmUUwqLwRhNzcy"

PRICE_TIERS = [
    (35, 30),
    (50, 50),
    (100, 100),
    (500, 1000),
]

logging.basicConfig(level=logging.INFO)

bot = Bot(token=BOT_TOKEN)
dp = Dispatcher(storage=MemoryStorage())

user_data: dict[int, dict] = {}


class SupportState(StatesGroup):
    waiting_message = State()



TEXTS = {
    "ru": {
        "welcome": (
            "привет друг!\n"
            "рад видеть тебя в моем шопе\n"
            "выбирай что хотел бы купить"
        ),
        "not_subscribed": (
            "эй, сначала подпишись на канал\n"
            "без этого не пущу"
        ),
        "subscribe_btn": "подписаться",
        "check_btn": "я подписался",
        "subscribed_ok": "ок заходи",
        "menu_buy": "купить приватку",
        "menu_profile": "мой профиль",
        "menu_settings": "настройки",
        "menu_support": "поддержка",
        "buy_title": "приватный сервер",
        "buy_text": "выбери срок приватки:",
        "buy_reserve": "⏳ бронируем товар на короткое время...",
        "buy_pay_text": "вы выбрали: {stars} ⭐ - {days} дней\n\nотправьте {stars} звёзд: @{support}\nпосле оплаты нажмите кнопку ниже",
        "buy_sent_btn": "я отправил",
        "buy_request_sent": "заявка отправлена, ожидайте подтверждения оплаты",
        "buy_success": "✅ оплата подтверждена!\nссылка на приват: {link}",
        "buy_declined": "ай ай, мухлевать не хорошо, пожалуйста проведите оплату или товар не будет выдан",
        "profile_title": "твой профиль",
        "profile_text": "id: {uid}\nюз: @{username}",
        "settings_title": "настройки",
        "settings_text": "выбери язык",
        "lang_ru": "русский",
        "lang_ua": "украинский",
        "lang_en": "english",
        "lang_changed": "ок, русский",
        "support_prompt": "введите сообщение для поддержки:",
        "support_cancel_btn": "отмена",
        "support_sent": "сообщение отправлено, скоро ответят",
        "support_cancelled": "отменено",
        "back": "назад",
    },
    "ua": {
        "welcome": (
            "привiт друже!\n"
            "радий бачити тебе в моєму шопi\n"
            "обирай що хочеш купити"
        ),
        "not_subscribed": (
            "спочатку пiдпишись на канал\n"
            "без цього не пущу"
        ),
        "subscribe_btn": "пiдписатись",
        "check_btn": "я пiдписався",
        "subscribed_ok": "окей заходь",
        "menu_buy": "купити приватку",
        "menu_profile": "мiй профiль",
        "menu_settings": "налаштування",
        "menu_support": "пiдтримка",
        "buy_title": "приватний сервер",
        "buy_text": "обери термiн приватки:",
        "buy_reserve": "⏳ бронюємо товар на короткий час...",
        "buy_pay_text": "ви обрали: {stars} ⭐ - {days} днiв\n\nвiдправте {stars} зiрок: @{support}\nпiсля оплати натиснiть кнопку нижче",
        "buy_sent_btn": "я вiдправив",
        "buy_request_sent": "заявку вiдправлено, очiкуйте пiдтвердження оплати",
        "buy_success": "✅ оплату пiдтверджено!\nпосилання на приват: {link}",
        "buy_declined": "ай ай, хитрувати не добре, будь ласка проведiть оплату або товар не буде видано",
        "profile_title": "твiй профiль",
        "profile_text": "id: {uid}\nюз: @{username}",
        "settings_title": "налаштування",
        "settings_text": "обери мову",
        "lang_ru": "росiйська",
        "lang_ua": "украiнська",
        "lang_en": "english",
        "lang_changed": "окей, украiнська",
        "support_prompt": "введiть повiдомлення для пiдтримки:",
        "support_cancel_btn": "скасувати",
        "support_sent": "повiдомлення надiслано, скоро вiдповiдять",
        "support_cancelled": "скасовано",
        "back": "назад",
    },
    "en": {
        "welcome": (
            "hey friend!\n"
            "glad to see you in my shop\n"
            "pick what you wanna buy"
        ),
        "not_subscribed": (
            "subscribe to the channel first\n"
            "cant let you in without that"
        ),
        "subscribe_btn": "subscribe",
        "check_btn": "i subscribed",
        "subscribed_ok": "ok come in",
        "menu_buy": "buy private",
        "menu_profile": "my profile",
        "menu_settings": "settings",
        "menu_support": "support",
        "buy_title": "private server",
        "buy_text": "choose private duration:",
        "buy_reserve": "⏳ reserving the item for a short time...",
        "buy_pay_text": "you chose: {stars} ⭐ - {days} days\n\nsend {stars} stars to: @{support}\nafter payment press the button below",
        "buy_sent_btn": "i sent it",
        "buy_request_sent": "request sent, waiting for payment confirmation",
        "buy_success": "✅ payment confirmed!\nprivate link: {link}",
        "buy_declined": "hey, cheating isn't nice, please complete the payment or the item won't be given",
        "profile_title": "your profile",
        "profile_text": "id: {uid}\nusername: @{username}",
        "settings_title": "settings",
        "settings_text": "pick a language",
        "lang_ru": "russian",
        "lang_ua": "ukrainian",
        "lang_en": "english",
        "lang_changed": "ok, english",
        "support_prompt": "enter your message for support:",
        "support_cancel_btn": "cancel",
        "support_sent": "message sent, they'll reply soon",
        "support_cancelled": "cancelled",
        "back": "back",
    },
}


def get_lang(user_id: int) -> str:
    return user_data.get(user_id, {}).get("lang", "ru")


def t(user_id: int, key: str, **kwargs) -> str:
    lang = get_lang(user_id)
    text = TEXTS[lang].get(key, key)
    return text.format(**kwargs) if kwargs else text



def main_menu_kb(user_id: int) -> InlineKeyboardMarkup:
    return InlineKeyboardMarkup(inline_keyboard=[
        [InlineKeyboardButton(text=t(user_id, "menu_buy"), callback_data="buy")],
        [InlineKeyboardButton(text=t(user_id, "menu_profile"), callback_data="profile")],
        [InlineKeyboardButton(text=t(user_id, "menu_settings"), callback_data="settings")],
        [InlineKeyboardButton(text=t(user_id, "menu_support"), callback_data="support")],
    ])


def subscribe_kb(user_id: int) -> InlineKeyboardMarkup:
    return InlineKeyboardMarkup(inline_keyboard=[
        [InlineKeyboardButton(text=t(user_id, "subscribe_btn"), url=CHANNEL_INVITE)],
        [InlineKeyboardButton(text=t(user_id, "check_btn"), callback_data="check_sub")],
    ])


def back_kb(user_id: int) -> InlineKeyboardMarkup:
    return InlineKeyboardMarkup(inline_keyboard=[
        [InlineKeyboardButton(text=t(user_id, "back"), callback_data="back_menu")],
    ])


def buy_kb(user_id: int) -> InlineKeyboardMarkup:
    rows = [
        [InlineKeyboardButton(
            text=f"{stars} ⭐ - {days} дней",
            callback_data=f"buy_{stars}_{days}"
        )]
        for stars, days in PRICE_TIERS
    ]
    rows.append([InlineKeyboardButton(text=t(user_id, "back"), callback_data="back_menu")])
    return InlineKeyboardMarkup(inline_keyboard=rows)


def buy_pay_kb(user_id: int, stars: int, days: int) -> InlineKeyboardMarkup:
    return InlineKeyboardMarkup(inline_keyboard=[
        [InlineKeyboardButton(text=t(user_id, "buy_sent_btn"), callback_data=f"sent_{stars}_{days}")],
        [InlineKeyboardButton(text=t(user_id, "back"), callback_data="back_menu")],
    ])


def admin_confirm_kb(uid: int, stars: int, days: int) -> InlineKeyboardMarkup:
    return InlineKeyboardMarkup(inline_keyboard=[
        [
            InlineKeyboardButton(text="✅ оплата получена", callback_data=f"appr_{uid}_{stars}_{days}"),
            InlineKeyboardButton(text="❌ оплата не получена", callback_data=f"deny_{uid}"),
        ],
    ])


def settings_kb(user_id: int) -> InlineKeyboardMarkup:
    return InlineKeyboardMarkup(inline_keyboard=[
        [
            InlineKeyboardButton(text=t(user_id, "lang_ru"), callback_data="lang_ru"),
            InlineKeyboardButton(text=t(user_id, "lang_ua"), callback_data="lang_ua"),
            InlineKeyboardButton(text=t(user_id, "lang_en"), callback_data="lang_en"),
        ],
        [InlineKeyboardButton(text=t(user_id, "back"), callback_data="back_menu")],
    ])


def support_cancel_kb(user_id: int) -> InlineKeyboardMarkup:
    return InlineKeyboardMarkup(inline_keyboard=[
        [InlineKeyboardButton(text=t(user_id, "support_cancel_btn"), callback_data="support_cancel")],
    ])


async def check_subscription(user_id: int) -> bool:
    try:
        member = await bot.get_chat_member(CHANNEL_USERNAME, user_id)
        return member.status in ("member", "administrator", "creator")
    except Exception as e:
        logging.error(f"проверка подписки на {CHANNEL_USERNAME} для {user_id} не удалась: {e}")
        return False


@dp.message(CommandStart())
async def cmd_start(message: Message, state: FSMContext):
    await state.clear()
    uid = message.from_user.id
    if uid not in user_data:
        user_data[uid] = {"lang": "ru"}

    subscribed = await check_subscription(uid)
    if not subscribed:
        await message.answer(t(uid, "not_subscribed"), reply_markup=subscribe_kb(uid))
        return

    await message.answer(t(uid, "welcome"), reply_markup=main_menu_kb(uid))

@dp.callback_query(F.data == "check_sub")
async def check_sub_callback(callback: CallbackQuery):
    uid = callback.from_user.id
    subscribed = await check_subscription(uid)
    if not subscribed:
        await callback.answer(t(uid, "not_subscribed"), show_alert=True)
        return
    await callback.message.edit_text(t(uid, "welcome"), reply_markup=main_menu_kb(uid))
    await callback.answer(t(uid, "subscribed_ok"))


@dp.callback_query(F.data == "back_menu")
async def back_menu(callback: CallbackQuery, state: FSMContext):
    await state.clear()
    uid = callback.from_user.id
    await callback.message.edit_text(t(uid, "welcome"), reply_markup=main_menu_kb(uid))
    await callback.answer()


@dp.callback_query(F.data == "buy")
async def buy_menu(callback: CallbackQuery):
    uid = callback.from_user.id
    await callback.message.edit_text(
        f"{t(uid, 'buy_title')}\n\n{t(uid, 'buy_text')}",
        reply_markup=buy_kb(uid)
    )
    await callback.answer()


@dp.callback_query(F.data.startswith("buy_"))
async def choose_tier(callback: CallbackQuery):
    uid = callback.from_user.id
    _, stars, days = callback.data.split("_")
    stars, days = int(stars), int(days)
    text = (
        f"{t(uid, 'buy_reserve')}\n\n"
        f"{t(uid, 'buy_pay_text', stars=stars, days=days, support=SUPPORT_USERNAME)}"
    )
    await callback.message.edit_text(text, reply_markup=buy_pay_kb(uid, stars, days))
    await callback.answer()


@dp.callback_query(F.data.startswith("sent_"))
async def sent_payment(callback: CallbackQuery):
    uid = callback.from_user.id
    _, stars, days = callback.data.split("_")
    stars, days = int(stars), int(days)
    username = callback.from_user.username or "без юза"

    try:
        await bot.send_message(
            ADMIN_ID,
            f"клиент: {username}\n"
            f"юз: @{username}\n"
            f"id: {uid}\n"
            f"звёзд: {stars}\n"
            f"на сколько купил: {days} дней",
            reply_markup=admin_confirm_kb(uid, stars, days)
        )
    except Exception as e:
        logging.error(f"не удалось отправить уведомление админу {ADMIN_ID}: {e}")

    await callback.message.edit_text(t(uid, "buy_request_sent"), reply_markup=back_kb(uid))
    await callback.answer()


@dp.callback_query(F.data.startswith("appr_"))
async def approve_payment(callback: CallbackQuery):
    _, uid, stars, days = callback.data.split("_")
    uid, stars, days = int(uid), int(stars), int(days)

    try:
        await bot.send_message(
            uid,
            t(uid, "buy_success", link=PRIVATE_LINK),
            reply_markup=back_kb(uid)
        )
    except Exception as e:
        logging.error(f"не удалось отправить ссылку покупателю {uid}: {e}")
    await callback.answer()


@dp.callback_query(F.data.startswith("deny_"))
async def deny_payment(callback: CallbackQuery):
    _, uid = callback.data.split("_")
    uid = int(uid)

    try:
        await bot.send_message(uid, t(uid, "buy_declined"))
    except Exception as e:
        logging.error(f"не удалось отправить отказ покупателю {uid}: {e}")
    await callback.answer()


@dp.callback_query(F.data == "profile")
async def profile_menu(callback: CallbackQuery):
    uid = callback.from_user.id
    username = callback.from_user.username or "нет юзернейма"
    await callback.message.edit_text(
        f"{t(uid, 'profile_title')}\n\n{t(uid, 'profile_text', uid=uid, username=username)}",
        reply_markup=back_kb(uid)
    )
    await callback.answer()


@dp.callback_query(F.data == "settings")
async def settings_menu(callback: CallbackQuery):
    uid = callback.from_user.id
    await callback.message.edit_text(
        f"{t(uid, 'settings_title')}\n\n{t(uid, 'settings_text')}",
        reply_markup=settings_kb(uid)
    )
    await callback.answer()


@dp.callback_query(F.data.in_({"lang_ru", "lang_ua", "lang_en"}))
async def change_language(callback: CallbackQuery):
    uid = callback.from_user.id
    lang_map = {"lang_ru": "ru", "lang_ua": "ua", "lang_en": "en"}
    if uid not in user_data:
        user_data[uid] = {}
    user_data[uid]["lang"] = lang_map[callback.data]
    await callback.answer(t(uid, "lang_changed"), show_alert=True)
    await callback.message.edit_text(
        f"{t(uid, 'settings_title')}\n\n{t(uid, 'settings_text')}",
        reply_markup=settings_kb(uid)
    )


@dp.callback_query(F.data == "support")
async def support_menu(callback: CallbackQuery, state: FSMContext):
    uid = callback.from_user.id
    await state.set_state(SupportState.waiting_message)
    await callback.message.edit_text(
        t(uid, "support_prompt"),
        reply_markup=support_cancel_kb(uid)
    )
    await callback.answer()


@dp.callback_query(F.data == "support_cancel", SupportState.waiting_message)
async def support_cancel(callback: CallbackQuery, state: FSMContext):
    await state.clear()
    uid = callback.from_user.id
    await callback.message.edit_text(t(uid, "welcome"), reply_markup=main_menu_kb(uid))
    await callback.answer(t(uid, "support_cancelled"))


@dp.message(SupportState.waiting_message)
async def support_receive(message: Message, state: FSMContext):
    await state.clear()
    uid = message.from_user.id
    username = message.from_user.username or "без юза"
    text = message.text or "[не текст]"

    try:
        await bot.send_message(
            ADMIN_ID,
            f"сообщение от пользователя\n\nid: {uid}\nюз: @{username}\n\n{text}"
        )
    except Exception as e:
        logging.error(f"не удалось переслать сообщение поддержке: {e}")


async def main():
    await dp.start_polling(bot)


if __name__ == "__main__":
    asyncio.run(main())
