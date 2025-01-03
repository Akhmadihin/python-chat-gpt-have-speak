from g4f.client import Client
import speech_recognition as sr
import pyttsx3

# Инициализация движка для синтеза речи
engine = pyttsx3.init()

def speak(text):
    """Произносит текст."""
    engine.say(text)
    engine.runAndWait()

def listen():
    """Прослушивает и распознает речь, ожидая активационного слова."""
    r = sr.Recognizer()
    with sr.Microphone() as source:
        r.adjust_for_ambient_noise(source, duration=1)
        r.energy_threshold = 400
        print("Ожидаю активационное слово \"Ралина\"...")
        while True:
            audio = r.listen(source)
            try:
                query = r.recognize_google(audio, language="ru-RU")
                if query.lower() == "ралина":
                    print("Слушаю...")
                    audio = r.listen(source)
                    query = r.recognize_google(audio, language="ru-RU")
                    print(f"Вы сказали: {query}")
                    return query
                else:
                    print(f"Вы сказали: {query}. Ожидаю \"Ралина\".")
            except sr.UnknownValueError:
                print("Не распознал речь. Повторите, пожалуйста.")
            except sr.RequestError as e:
                print(f"Ошибка сервиса распознавания: {e}")
            except Exception as e:
                print(f"Произошла ошибка: {e}")
            

def main():
    client = Client()
    try:
        while True:
            query = listen()
            if query:
                response = client.chat.completions.create(
                    model="gpt-3.5-turbo",
                    messages=[{"role": "user", "content": query}]
                )
                print(response.choices[0].message.content)
                speak(response.choices[0].message.content)
    except Exception as e:
        print(f"Ошибка: {e}")


if __name__ == "__main__":
    main()
