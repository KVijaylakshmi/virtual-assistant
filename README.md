# virtual-assistant
import os
import speech_recognition as sr
import pyttsx3
import wikipedia
import pywhatkit
import requests
import pyjokes
import datetime

def speak(text):
    engine = pyttsx3.init()
    engine.say(text)
    engine.runAndWait()
    print(text)  # Display text output

def listen():
    recognizer = sr.Recognizer()
    with sr.Microphone() as source:
        print("Listening...")
        recognizer.adjust_for_ambient_noise(source)
        audio = recognizer.listen(source)
        try:
            command = recognizer.recognize_google(audio).lower()
            print("You said:", command)
            return command
        except sr.UnknownValueError:
            response = "Sorry, I couldn't understand that."
            speak(response)
            return ""
        except sr.RequestError:
            response = "Network error. Please check your connection."
            speak(response)
            return ""

def get_news(country="in"):
    # Retrieve the API key from an environment variable
    api_key = os.environ.get("NEWS_API_KEY")
    if not api_key:
        return "News API key is not set. Please set your News API key."
    url = f"https://newsapi.org/v2/top-headlines?country={country}&apiKey={api_key}"
    try:
        response = requests.get(url, timeout=5)
        response.raise_for_status()
        data = response.json()
        articles = data.get("articles", [])
        if articles:
            # Format news with title and description for the top 5 articles
            news = "\n".join([f"{idx+1}. {article.get('title', 'No Title')}\n   {article.get('description', 'No Description')}"
                              for idx, article in enumerate(articles[:5])])
            return news
        else:
            return "I couldn't find any news at the moment. Please try again later."
    except requests.exceptions.RequestException:
        return "There was an error fetching the news. Please check your internet connection."

def get_fact():
    try:
        response = requests.get("https://uselessfacts.jsph.pl/random.json?language=en", timeout=5)
        response.raise_for_status()
        return response.json().get("text", "No fact available right now.")
    except requests.exceptions.RequestException:
        return "Unable to fetch a fact right now. Please check your internet connection."

def assistant():
    speak("Hello! How can I assist you today? todays weather "
          " the weather is expected to be Haze with a maximum temperature of 34°C and a minimum of 23°C."
          " Sunrise in Hyderabad is set for 06:17 AM and sunset at 06:27 PM "
          "Atmospheric pressure will be around 101.9 kPa with humidity at 59."
          "9% probability of rain in Hyderabad and the UV index is at 9. "
          "Tomorrow, expect the temperature range of 35°C to 22°C. ")
    while True:
        command = listen()

        if "wikipedia" in command:
            speak("Searching Wikipedia...")
            query = command.replace("wikipedia", "").strip()
            try:
                result = wikipedia.summary(query, sentences=4)
                speak(result)
            except wikipedia.exceptions.DisambiguationError:
                speak("There are multiple entries for this topic. Please be more specific.")
            except wikipedia.exceptions.PageError:
                speak("I couldn't find any results for that topic.")

        elif "news" in command:
            speak("Fetching today's news...")
            news = get_news()
            speak(news)

        elif "fact" in command:
            fact = get_fact()
            speak(fact)

        elif "joke" in command:
            joke = pyjokes.get_joke()
            speak(joke)

        elif "play" in command:
            song = command.replace("play", "").strip()
            speak(f"Playing {song} on YouTube")
            pywhatkit.playonyt(song)

        elif "time" in command:
            current_time = datetime.datetime.now().strftime("%I:%M %p")
            speak(f"The time is {current_time}")

        elif "exit" in command or "stop" in command:
            speak("Goodbye! Have a great day!")
            break

        else:
            response = "I didn't understand. Please try again."
            speak(response)

if __name__ == "__main__":
    assistant()
