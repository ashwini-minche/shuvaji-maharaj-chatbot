from flask import Flask, request, render_template_string
import re

app = Flask(__name__)

# ---------------- KNOWLEDGE ----------------
KNOWLEDGE = {
    "name": "I am Chhatrapati Shivaji Maharaj, the founder of the Maratha Empire.",
    "mother": "Jijabai (also called Rajmata Jijabai)",
    "father": "Shahaji Bhosale",
    "who was chhatrapati shivaji maharaj": "Shivaji Maharaj was a great Maratha warrior king and founder of the Maratha Empire.",
    "born": "Shivaji Maharaj was born on 19 February 1630 at Shivneri Fort.",
    "birthdate": "Shivaji Maharaj was born on 19 February 1630 at Shivneri Fort.",
    "parents": "His father was Shahaji Bhosale, and his mother was Jijabai.",
    "died": "He died on 3 April 1680.",
    "title": "He was honored with the title 'Chhatrapati' by the people of Maharashtra.",
    "childhood": "Shivaji Maharaj spent his childhood at Shivneri Fort and nearby forts.",
    "education": "He was educated in martial arts, administration, and was inspired by epics like Ramayana and Mahabharata.",
    "forts": "He captured forts like Torna, Rajgad, Raigad, Pratapgad, Sindhudurg, and many others.",
    "military": "He organized a guerrilla warfare army and a strong navy to defend his kingdom.",
    "battle of pratapgad": "The Battle of Pratapgad in 1659 was a significant victory against Afzal Khan.",
    "guerrilla warfare": "He pioneered guerrilla tactics to outsmart bigger armies.",
    "navy": "He built one of the first Indian navies to protect coastal areas.",
    "ashta pradhan": "Ashta Pradhan was his council of eight ministers to help govern the kingdom efficiently.",
    "taxation": "He implemented a fair taxation system including Chauth and Sardeshmukhi.",
    "religious tolerance": "He promoted religious tolerance and respected all faiths.",
    "wives": "His notable wives were Saibai, Sakvarbai, and Putalabai. His son was Sambhaji Maharaj.",
    "personal values": "He was known for courage, justice, frugality, and devotion to Bhavani Mata.",
    "legacy": "Shivaji Maharaj is considered a hero who inspired later freedom fighters and is celebrated in festivals and literature.",
    "weapons": "He used swords, bows, spears, and firearms during battles.",
    "kingdom": "His kingdom covered large parts of Maharashtra and surrounding regions.",
    "rivals": "His main rivals included the Mughals, Bijapur Sultanate, and others.",
    "aurangzeb": "Aurangzeb was his main Mughal adversary, leading many campaigns against him.",
    "escapes": "He famously escaped from Agra and other enemy strongholds.",
    "diplomacy": "He used diplomacy wisely to form alliances and strengthen his kingdom."
}

# ---------------- HTML ----------------
HTML = """
<!DOCTYPE html>
<html>
<head>
    <title>Virtual Museum Guide - Shivaji Maharaj</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: #fff8e1;
            margin: 0; padding: 0;
            display: flex;
            justify-content: center;
            align-items: flex-start;
            min-height: 100vh;
            padding-top: 40px;
        }
        .chatbox {
            background: white;
            border-radius: 15px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
            width: 100%;
            max-width: 700px;
            padding: 30px;
        }
        h2 {
            text-align: center;
            margin-bottom: 15px;
            color: #d84315;
            font-weight: 700;
            font-size: 28px;
        }
        .chatbox img {
            display: block;
            margin: 0 auto 25px;
            max-width: 180px;
            border-radius: 15px;
        }
        .chat-entry {
            margin-bottom: 18px;
            display: flex;
            flex-direction: column;
        }
        .chat-entry.user {
            align-items: flex-end;
        }
        .chat-entry .message {
            display: inline-block;
            padding: 12px 18px;
            border-radius: 20px;
            max-width: 80%;
            font-size: 16px;
            line-height: 1.4;
        }
        .chat-entry.user .message {
            background: #efebe9;
            color: #3e2723;
            border-bottom-right-radius: 4px;
        }
        .chat-entry.bot .message {
            background: #d84315;
            color: white;
            border-bottom-left-radius: 4px;
        }
        form {
            display: flex;
            margin-top: 20px;
            border-top: 1px solid #eee;
            padding-top: 20px;
        }
        input[type=text] {
            flex-grow: 1;
            font-size: 16px;
            padding: 12px 15px;
            border: 2px solid #d84315;
            border-radius: 25px 0 0 25px;
            outline: none;
        }
        input[type=text]:focus {
            border-color: #bf360c;
        }
        button {
            background: #d84315;
            border: none;
            padding: 0 25px;
            color: white;
            font-weight: 700;
            border-radius: 0 25px 25px 0;
            cursor: pointer;
            transition: background-color 0.3s ease;
            font-size: 16px;
        }
        button:hover {
            background: #bf360c;
        }
        @media (max-width: 600px) {
            .chatbox {
                padding: 20px;
                width: 95%;
            }
        }
    </style>
</head>
<body>
    <div class="chatbox">
        <h2>🏛️ Chat with Chhatrapati Shivaji Maharaj</h2>
        <img src="{{ url_for('static', filename='shivaji.jpg') }}" alt="Shivaji Maharaj">
        {% for role, message in chat %}
            <div class="chat-entry {{ role }}">
                <div class="message"><strong>{{ role.title() }}:</strong> {{ message }}</div>
            </div>
        {% endfor %}
        <form method="post">
            <input name="question" placeholder="Ask about Shivaji Maharaj's life..." autocomplete="off" required>
            <button type="submit">Ask</button>
        </form>
    </div>
</body>
</html>
"""

chat_history = []

# ---------------- CHAT LOGIC ----------------
def normalize(text):
    text = text.lower()
    text = re.sub(r'[^\w\s]', '', text)  # remove punctuation
    return text.strip()

def get_response(question):
    q = normalize(question)
    for key in KNOWLEDGE:
        if key in q:
            return KNOWLEDGE[key]
    return ("Sorry, I don't have information on that. "
            "Please ask about Shivaji Maharaj's early life, military, administration, or legacy.")

# ---------------- ROUTE ----------------
@app.route("/", methods=["GET", "POST"])
def chat():
    if request.method == "POST":
        question = request.form["question"]
        answer = get_response(question)

        chat_history.append(("user", question))
        chat_history.append(("bot", answer))

    return render_template_string(HTML, chat=chat_history)

# ---------------- RUN ----------------
if __name__ == "__main__":
    app.run(host="0.0.0.0",port=5000)
