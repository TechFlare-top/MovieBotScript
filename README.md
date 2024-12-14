import requests
from telegram import Update
from telegram.ext import Updater, CommandHandler, MessageHandler, Filters, CallbackContext

# Replace with your OMDB API key
OMDB_API_KEY = 'http://www.omdbapi.com/?i=tt3896198&apikey=f497410e'

# Replace with your bot token from BotFather
BOT_TOKEN = '7306530168:AAEeW61uT5YADFmX58LwBpRVip2fLl-E3jE'

def start(update: Update, context: CallbackContext) -> None:
    """Send a welcome message when the /start command is issued."""
    update.message.reply_text(
        "🎬 Welcome to Movie Finder Bot! 🎥\n"
        "Type a movie name to find details, or use /help for instructions."
    )

def help_command(update: Update, context: CallbackContext) -> None:
    """Send a help message when the /help command is issued."""
    update.message.reply_text(
        "🛠 How to use Movie Finder Bot:\n"
        "- Type a movie name directly to search for movies.\n"
        "- Use /movie [movie_name] to search for a specific movie.\n\n"
        "Example:\n"
        "/movie Inception"
    )

def search_movie(update: Update, context: CallbackContext) -> None:
    """Handle the /movie command and search for movies."""
    if not context.args:
        update.message.reply_text("❗ Please provide a movie name. Example: /movie Inception")
        return

    movie_name = ' '.join(context.args)
    url = f"http://www.omdbapi.com/?s={movie_name}&apikey={OMDB_API_KEY}"
    response = requests.get(url).json()

    if response.get("Response") == "True":
        results = response.get("Search", [])
        reply = f"🎬 Movies found for '{movie_name}':\n\n"
        for movie in results:
            reply += f"- {movie.get('Title')} ({movie.get('Year')})\n"
        update.message.reply_text(reply)
    else:
        update.message.reply_text(f"❌ No movies found for '{movie_name}'.")

def handle_text(update: Update, context: CallbackContext) -> None:
    """Handle text messages to automatically search for movies."""
    movie_name = update.message.text
    url = f"http://www.omdbapi.com/?s={movie_name}&apikey={OMDB_API_KEY}"
    response = requests.get(url).json()

    if response.get("Response") == "True":
        results = response.get("Search", [])
        reply = f"🎬 Movies found for '{movie_name}':\n\n"
        for movie in results:
            reply += f"- {movie.get('Title')} ({movie.get('Year')})\n"
        update.message.reply_text(reply)
    else:
        update.message.reply_text(f"❌ No movies found for '{movie_name}'.")

def main() -> None:
    """Start the bot."""
    updater = Updater(BOT_TOKEN)

    dispatcher = updater.dispatcher

    # Command handlers
    dispatcher.add_handler(CommandHandler("start", start))
    dispatcher.add_handler(CommandHandler("help", help_command))
    dispatcher.add_handler(CommandHandler("movie", search_movie))

    # Text message handler
    dispatcher.add_handler(MessageHandler(Filters.text & ~Filters.command, handle_text))

    # Start the bot
    updater.start_polling()
    updater.idle()

if __name__ == "__main__":
    main()
