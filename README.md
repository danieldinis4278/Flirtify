# Flirtify
date
from flask import Flask, request, jsonify
from flask_cors import CORS

app = Flask(__name__)
CORS(app)  # Permitir conexões do frontend

# Simulated database
users = []
matches = []

@app.route('/register', methods=['POST'])
def register():
    """Register a new user."""
    data = request.json
    if not data.get('name') or not data.get('age') or not data.get('bio'):
        return jsonify({'error': 'Missing required fields'}), 400

    user = {
        'id': len(users) + 1,
        'name': data['name'],
        'age': data['age'],
        'bio': data['bio'],
        'likes': [],
        'liked_by': []
    }
    users.append(user)
    return jsonify({'message': 'User registered successfully', 'user': user}), 201

@app.route('/profiles', methods=['GET'])
def profiles():
    """Retrieve profiles of all users."""
    current_user_id = request.args.get('user_id', type=int)
    if not current_user_id:
        return jsonify({'error': 'user_id is required'}), 400

    available_profiles = [user for user in users if user['id'] != current_user_id]
    return jsonify(available_profiles)

@app.route('/like', methods=['POST'])
def like():
    """Like a user profile."""
    data = request.json
    user_id = data.get('user_id')
    target_id = data.get('target_id')

    if not user_id or not target_id:
        return jsonify({'error': 'user_id and target_id are required'}), 400

    user = next((u for u in users if u['id'] == user_id), None)
    target = next((u for u in users if u['id'] == target_id), None)

    if not user or not target:
        return jsonify({'error': 'User not found'}), 404

    if target_id in user['likes']:
        return jsonify({'message': 'You already liked this user'}), 400

    user['likes'].append(target_id)
    target['liked_by'].append(user_id)

    if user_id in target['likes']:
        matches.append((user_id, target_id))
        return jsonify({'message': 'It\'s a match!', 'match': (user_id, target_id)}), 200

    return jsonify({'message': 'Profile liked successfully'}), 200

@app.route('/matches', methods=['GET'])
def get_matches():
    """Retrieve all matches for a user."""
    user_id = request.args.get('user_id', type=int)
    if not user_id:
        return jsonify({'error': 'user_id is required'}), 400

    user_matches = [pair for pair in matches if user_id in pair]
    return jsonify(user_matches)

if __name__ == '__main__':
    app.run(debug=True)

