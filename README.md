from flask import Flask, render_template_string, request, jsonify
import random

app = Flask(__name__)

# Store game sessions
games = {}

@app.route('/')
def home():
    return render_template_string('''
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Cloud Number Guessing Game</title>
        <style>
            body { font-family: Arial, sans-serif; text-align: center; padding: 20px; }
            input, button { font-size: 1.2em; padding: 10px; margin: 5px; }
        </style>
        <script>
            let sessionId = "user123";  // Change this for real multi-user support

            async function startGame() {
                let response = await fetch('/start', { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ session_id: sessionId }) });
                let data = await response.json();
                document.getElementById("message").innerText = data.message;
            }

            async function makeGuess() {
                let guess = document.getElementById("guess").value;
                let response = await fetch('/guess', { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ session_id: sessionId, guess: guess }) });
                let data = await response.json();
                document.getElementById("message").innerText = data.message;
            }
        </script>
    </head>
    <body>
        <h1>Cloud Number Guessing Game</h1>
        <button onclick="startGame()">Start Game</button>
        <p>Enter your guess (between 1 and 100):</p>
        <input type="number" id="guess" min="1" max="100">
        <button onclick="makeGuess()">Submit Guess</button>
        <p id="message"></p>
    </body>
    </html>
    ''')

@app.route('/start', methods=['POST'])
def start_game():
    session_id = request.json.get('session_id')
    games[session_id] = {
        "number": random.randint(1, 100),
        "attempts": 5
    }
    return jsonify({"message": "Game started! Guess a number between 1 and 100."})

@app.route('/guess', methods=['POST'])
def guess():
    session_id = request.json.get('session_id')
    user_guess = int(request.json.get('guess'))

    if session_id not in games:
        return jsonify({"message": "No active game session. Click Start Game."}), 400

    game = games[session_id]
    game["attempts"] -= 1

    if user_guess == game["number"]:
        del games[session_id]
        return jsonify({"message": "🎉 Congratulations! You guessed the correct number!"})
    elif game["attempts"] <= 0:
        del games[session_id]
        return jsonify({"message": f"❌ Game Over! The correct number was {game['number']}."})
    elif user_guess < game["number"]:
        return jsonify({"message": f"⬆ Too low! Attempts left: {game['attempts']}"})
    else:
        return jsonify({"message": f"⬇ Too high! Attempts left: {game['attempts']}"})

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)
