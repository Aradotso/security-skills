---
name: ai-security-trainer-2026
description: Browser-based cybersecurity education platform with guided lessons, quizzes, gamified progression, and adaptive learning scenarios for information security practice.
triggers:
  - set up ai security trainer platform
  - configure cybersecurity learning environment
  - create security training lessons and quizzes
  - implement gamified security education
  - build interactive security training scenarios
  - customize ai security trainer features
  - deploy security education platform
  - integrate security learning progress tracking
---

# AI Security Trainer v2026 Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

AI Security Trainer v2026 is a browser-based cybersecurity education platform that provides interactive security training through guided lessons, quizzes, gamified progression, and adaptive learning scenarios. The platform is built with HTML/JavaScript frontend and Flask backend, offering features like XP systems, badges, leaderboards, skills mapping, and user profile management.

## Installation

### Local Setup

```bash
# Clone the repository
git clone https://github.com/dylanhwiireed3200/ai-security-trainer-2026.git
cd ai-security-trainer-2026

# Install Python dependencies (typically requires Flask)
pip install -r requirements.txt

# Start the Flask application
python app.py
# or
flask run

# Access the platform at http://localhost:5000
```

### Docker Deployment

```bash
# Build the Docker image
docker build -t ai-security-trainer:2026 .

# Run the container
docker run -d -p 5000:5000 \
  -v $(pwd)/data:/app/data \
  --name security-trainer \
  ai-security-trainer:2026

# Access at http://localhost:5000
```

## Project Structure

```
ai-security-trainer-2026/
├── app.py                 # Flask application entry point
├── static/
│   ├── css/              # Styling (dark theme)
│   ├── js/               # Frontend logic
│   └── images/           # Assets and avatars
├── templates/
│   ├── index.html        # Main interface
│   ├── lessons.html      # Lesson content
│   ├── dashboard.html    # User progress
│   └── skills.html       # Skills map
├── data/                 # User data and progress storage
├── lessons/              # Lesson content files
├── Dockerfile
└── requirements.txt
```

## Core Concepts

### User Progression System

The platform tracks learning through:
- **XP (Experience Points)**: Earned by completing lessons and quizzes
- **Levels**: Automatic progression based on accumulated XP
- **Badges**: Achievement milestones for completing challenges
- **Leaderboard**: Competitive ranking system

### Lesson Structure

Lessons follow a structured format with:
- Learning objectives
- Interactive content
- Embedded quizzes
- Practical scenarios
- Progress checkpoints

## Key Features Implementation

### Creating Custom Lessons

```javascript
// static/js/lesson-creator.js
const Lesson = {
  id: "sql-injection-basics",
  title: "SQL Injection Fundamentals",
  category: "web-security",
  difficulty: "intermediate",
  xpReward: 150,
  
  content: [
    {
      type: "text",
      content: "SQL injection allows attackers to manipulate database queries..."
    },
    {
      type: "code",
      language: "sql",
      content: "SELECT * FROM users WHERE username = '' OR '1'='1' --"
    },
    {
      type: "quiz",
      question: "What does the SQL comment operator '--' do?",
      options: [
        "Starts a multi-line comment",
        "Comments out the rest of the query",
        "Ends the SQL statement",
        "Escapes special characters"
      ],
      correctAnswer: 1
    }
  ],
  
  scenario: {
    type: "interactive",
    description: "Identify vulnerable SQL code",
    challenge: "Find the SQL injection vulnerability in the following code",
    solution: "The user input is directly concatenated into the query"
  }
};

// Add lesson to the system
function addLesson(lesson) {
  fetch('/api/lessons', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${localStorage.getItem('authToken')}`
    },
    body: JSON.stringify(lesson)
  })
  .then(response => response.json())
  .then(data => {
    console.log('Lesson created:', data.lessonId);
  });
}
```

### User Profile Management

```javascript
// static/js/profile.js
class UserProfile {
  constructor() {
    this.userId = localStorage.getItem('userId');
    this.profile = null;
  }
  
  async loadProfile() {
    const response = await fetch(`/api/users/${this.userId}/profile`);
    this.profile = await response.json();
    return this.profile;
  }
  
  async updateAvatar(avatarFile) {
    const formData = new FormData();
    formData.append('avatar', avatarFile);
    
    const response = await fetch(`/api/users/${this.userId}/avatar`, {
      method: 'POST',
      body: formData
    });
    
    return await response.json();
  }
  
  async getProgress() {
    const response = await fetch(`/api/users/${this.userId}/progress`);
    return await response.json();
  }
  
  calculateLevel(xp) {
    // Level calculation: level = sqrt(xp / 100)
    return Math.floor(Math.sqrt(xp / 100));
  }
}

// Usage
const profile = new UserProfile();
await profile.loadProfile();
console.log(`Level: ${profile.calculateLevel(profile.profile.xp)}`);
```

### Quiz System Implementation

```javascript
// static/js/quiz-engine.js
class QuizEngine {
  constructor(lessonId) {
    this.lessonId = lessonId;
    this.currentQuestion = 0;
    this.score = 0;
    this.answers = [];
  }
  
  async loadQuiz() {
    const response = await fetch(`/api/lessons/${this.lessonId}/quiz`);
    this.quiz = await response.json();
    return this.quiz;
  }
  
  submitAnswer(questionIndex, answerIndex) {
    const question = this.quiz.questions[questionIndex];
    const isCorrect = answerIndex === question.correctAnswer;
    
    this.answers.push({
      questionIndex,
      answerIndex,
      isCorrect,
      timestamp: Date.now()
    });
    
    if (isCorrect) {
      this.score += question.points || 10;
    }
    
    return {
      isCorrect,
      explanation: question.explanation,
      correctAnswer: question.correctAnswer
    };
  }
  
  async completeQuiz() {
    const result = {
      lessonId: this.lessonId,
      score: this.score,
      totalQuestions: this.quiz.questions.length,
      answers: this.answers,
      completedAt: new Date().toISOString()
    };
    
    const response = await fetch('/api/quiz/submit', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(result)
    });
    
    return await response.json();
  }
}
```

### Skills Map Tracking

```javascript
// static/js/skills-map.js
class SkillsMap {
  constructor() {
    this.skills = {
      'web-security': ['XSS', 'SQL Injection', 'CSRF', 'SSRF'],
      'network-security': ['Packet Analysis', 'Firewall Rules', 'VPN'],
      'cryptography': ['Symmetric', 'Asymmetric', 'Hashing', 'PKI'],
      'incident-response': ['Detection', 'Analysis', 'Containment', 'Recovery']
    };
  }
  
  async getUserSkills(userId) {
    const response = await fetch(`/api/users/${userId}/skills`);
    return await response.json();
  }
  
  calculateSkillProgress(category, completedLessons) {
    const totalSkills = this.skills[category].length;
    const masteredSkills = completedLessons.filter(
      lesson => lesson.category === category && lesson.score >= 80
    ).length;
    
    return {
      category,
      total: totalSkills,
      mastered: masteredSkills,
      percentage: (masteredSkills / totalSkills) * 100
    };
  }
  
  renderSkillsMap(userProgress) {
    const categories = Object.keys(this.skills);
    const skillsData = categories.map(cat => 
      this.calculateSkillProgress(cat, userProgress)
    );
    
    // Render visualization (radar chart, progress bars, etc.)
    return skillsData;
  }
}
```

## Flask Backend Implementation

### Main Application Setup

```python
# app.py
from flask import Flask, render_template, request, jsonify, session
from flask_cors import CORS
import json
import os
from datetime import datetime

app = Flask(__name__)
app.secret_key = os.environ.get('SECRET_KEY', 'dev-secret-key')
CORS(app)

# Configuration
app.config['UPLOAD_FOLDER'] = 'static/uploads/avatars'
app.config['MAX_CONTENT_LENGTH'] = 2 * 1024 * 1024  # 2MB max
app.config['DATA_FOLDER'] = 'data'

# Ensure directories exist
os.makedirs(app.config['UPLOAD_FOLDER'], exist_ok=True)
os.makedirs(app.config['DATA_FOLDER'], exist_ok=True)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/api/lessons', methods=['GET', 'POST'])
def lessons():
    if request.method == 'GET':
        lessons_file = os.path.join(app.config['DATA_FOLDER'], 'lessons.json')
        with open(lessons_file, 'r') as f:
            lessons = json.load(f)
        return jsonify(lessons)
    
    elif request.method == 'POST':
        lesson_data = request.json
        lesson_data['id'] = generate_lesson_id()
        lesson_data['created_at'] = datetime.utcnow().isoformat()
        
        # Save lesson
        lessons_file = os.path.join(app.config['DATA_FOLDER'], 'lessons.json')
        with open(lessons_file, 'r+') as f:
            lessons = json.load(f)
            lessons.append(lesson_data)
            f.seek(0)
            json.dump(lessons, f, indent=2)
        
        return jsonify({'lessonId': lesson_data['id']}), 201

@app.route('/api/users/<user_id>/progress', methods=['GET', 'POST'])
def user_progress(user_id):
    progress_file = os.path.join(app.config['DATA_FOLDER'], f'user_{user_id}_progress.json')
    
    if request.method == 'GET':
        if os.path.exists(progress_file):
            with open(progress_file, 'r') as f:
                return jsonify(json.load(f))
        return jsonify({'xp': 0, 'level': 1, 'completedLessons': [], 'badges': []})
    
    elif request.method == 'POST':
        progress_data = request.json
        with open(progress_file, 'w') as f:
            json.dump(progress_data, f, indent=2)
        return jsonify({'success': True})

@app.route('/api/quiz/submit', methods=['POST'])
def submit_quiz():
    quiz_data = request.json
    user_id = session.get('user_id')
    
    # Calculate XP based on score
    xp_earned = calculate_xp(quiz_data['score'], quiz_data['totalQuestions'])
    
    # Update user progress
    progress_file = os.path.join(app.config['DATA_FOLDER'], f'user_{user_id}_progress.json')
    with open(progress_file, 'r+') as f:
        progress = json.load(f)
        progress['xp'] += xp_earned
        progress['completedLessons'].append({
            'lessonId': quiz_data['lessonId'],
            'score': quiz_data['score'],
            'completedAt': quiz_data['completedAt']
        })
        
        # Check for level up
        new_level = calculate_level(progress['xp'])
        if new_level > progress.get('level', 1):
            progress['level'] = new_level
            progress['badges'].append({
                'type': 'level_up',
                'level': new_level,
                'earnedAt': datetime.utcnow().isoformat()
            })
        
        f.seek(0)
        json.dump(progress, f, indent=2)
    
    return jsonify({
        'xpEarned': xp_earned,
        'totalXp': progress['xp'],
        'level': progress['level'],
        'leveledUp': new_level > progress.get('level', 1)
    })

def calculate_xp(score, total):
    percentage = (score / total) * 100
    base_xp = 100
    return int(base_xp * (percentage / 100))

def calculate_level(xp):
    import math
    return math.floor(math.sqrt(xp / 100))

def generate_lesson_id():
    import uuid
    return str(uuid.uuid4())[:8]

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)
```

### Adaptive Learning Engine

```python
# adaptive_learning.py
import json
from datetime import datetime, timedelta

class AdaptiveLearningEngine:
    def __init__(self, user_id):
        self.user_id = user_id
        self.user_progress = self.load_progress()
    
    def load_progress(self):
        try:
            with open(f'data/user_{self.user_id}_progress.json', 'r') as f:
                return json.load(f)
        except FileNotFoundError:
            return {'completedLessons': [], 'weakAreas': [], 'strengths': []}
    
    def analyze_performance(self):
        """Analyze user performance to identify strengths and weaknesses"""
        lessons = self.user_progress.get('completedLessons', [])
        
        category_scores = {}
        for lesson in lessons:
            category = lesson.get('category', 'general')
            if category not in category_scores:
                category_scores[category] = []
            category_scores[category].append(lesson['score'])
        
        weak_areas = []
        strengths = []
        
        for category, scores in category_scores.items():
            avg_score = sum(scores) / len(scores)
            if avg_score < 70:
                weak_areas.append({
                    'category': category,
                    'avgScore': avg_score,
                    'attempts': len(scores)
                })
            elif avg_score > 90:
                strengths.append({
                    'category': category,
                    'avgScore': avg_score,
                    'attempts': len(scores)
                })
        
        return {'weakAreas': weak_areas, 'strengths': strengths}
    
    def recommend_next_lessons(self, available_lessons):
        """Recommend lessons based on performance and progress"""
        analysis = self.analyze_performance()
        completed_ids = [l['lessonId'] for l in self.user_progress.get('completedLessons', [])]
        
        recommendations = []
        
        # Prioritize weak areas
        for weak in analysis['weakAreas']:
            related_lessons = [
                l for l in available_lessons 
                if l['category'] == weak['category'] 
                and l['id'] not in completed_ids
                and l.get('difficulty') in ['beginner', 'intermediate']
            ]
            recommendations.extend(related_lessons[:2])
        
        # Add progressive lessons from strengths
        for strength in analysis['strengths']:
            advanced_lessons = [
                l for l in available_lessons
                if l['category'] == strength['category']
                and l['id'] not in completed_ids
                and l.get('difficulty') == 'advanced'
            ]
            recommendations.extend(advanced_lessons[:1])
        
        # Fill with uncompleted lessons
        remaining = [l for l in available_lessons if l['id'] not in completed_ids]
        recommendations.extend(remaining[:5 - len(recommendations)])
        
        return recommendations[:5]
    
    def adjust_difficulty(self, lesson_id, user_score):
        """Adjust lesson difficulty based on performance"""
        if user_score >= 90:
            return 'increase'  # User is ready for harder content
        elif user_score < 60:
            return 'decrease'  # User needs more foundational work
        else:
            return 'maintain'  # Current difficulty is appropriate
```

## Configuration

### Environment Variables

```bash
# .env
SECRET_KEY=your-secret-key-here
DATABASE_URL=sqlite:///data/security_trainer.db
UPLOAD_FOLDER=static/uploads/avatars
MAX_CONTENT_LENGTH=2097152
FLASK_ENV=production
FLASK_DEBUG=False
```

### Docker Configuration

```dockerfile
# Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

RUN mkdir -p data static/uploads/avatars

EXPOSE 5000

CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "app:app"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "5000:5000"
    environment:
      - SECRET_KEY=${SECRET_KEY}
      - FLASK_ENV=production
    volumes:
      - ./data:/app/data
      - ./static/uploads:/app/static/uploads
    restart: unless-stopped
```

## Common Patterns

### Creating Interactive Scenarios

```javascript
// static/js/scenario-engine.js
class SecurityScenario {
  constructor(scenarioConfig) {
    this.config = scenarioConfig;
    this.state = { phase: 0, score: 0, actions: [] };
  }
  
  async loadScenario() {
    const response = await fetch(`/api/scenarios/${this.config.id}`);
    this.scenario = await response.json();
    this.renderPhase(0);
  }
  
  renderPhase(phaseIndex) {
    const phase = this.scenario.phases[phaseIndex];
    
    // Display phase description
    document.getElementById('scenario-description').textContent = phase.description;
    
    // Render available actions
    const actionsContainer = document.getElementById('scenario-actions');
    actionsContainer.innerHTML = '';
    
    phase.actions.forEach((action, index) => {
      const button = document.createElement('button');
      button.textContent = action.label;
      button.onclick = () => this.executeAction(index);
      actionsContainer.appendChild(button);
    });
  }
  
  executeAction(actionIndex) {
    const phase = this.scenario.phases[this.state.phase];
    const action = phase.actions[actionIndex];
    
    this.state.actions.push({
      phase: this.state.phase,
      action: actionIndex,
      timestamp: Date.now()
    });
    
    // Update score based on action correctness
    if (action.isOptimal) {
      this.state.score += 20;
      this.showFeedback('Excellent choice!', 'success');
    } else if (action.isAcceptable) {
      this.state.score += 10;
      this.showFeedback('Good, but there might be a better approach', 'warning');
    } else {
      this.showFeedback('This action could have negative consequences', 'error');
    }
    
    // Move to next phase or complete scenario
    if (this.state.phase < this.scenario.phases.length - 1) {
      this.state.phase++;
      this.renderPhase(this.state.phase);
    } else {
      this.completeScenario();
    }
  }
  
  async completeScenario() {
    const result = await fetch('/api/scenarios/complete', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        scenarioId: this.config.id,
        score: this.state.score,
        actions: this.state.actions
      })
    });
    
    const data = await result.json();
    this.showResults(data);
  }
  
  showFeedback(message, type) {
    const feedback = document.createElement('div');
    feedback.className = `feedback feedback-${type}`;
    feedback.textContent = message;
    document.getElementById('scenario-feedback').appendChild(feedback);
  }
  
  showResults(data) {
    document.getElementById('scenario-score').textContent = `Score: ${this.state.score}`;
    document.getElementById('scenario-xp-earned').textContent = `XP Earned: ${data.xpEarned}`;
  }
}
```

### Leaderboard System

```python
# leaderboard.py
from datetime import datetime, timedelta
import json
import os

class Leaderboard:
    def __init__(self, data_folder='data'):
        self.data_folder = data_folder
        self.leaderboard_file = os.path.join(data_folder, 'leaderboard.json')
    
    def get_global_leaderboard(self, limit=10):
        """Get top users by total XP"""
        users = self._load_all_users()
        sorted_users = sorted(users, key=lambda x: x['xp'], reverse=True)
        return sorted_users[:limit]
    
    def get_weekly_leaderboard(self, limit=10):
        """Get top users by XP earned this week"""
        users = self._load_all_users()
        week_ago = datetime.now() - timedelta(days=7)
        
        weekly_scores = []
        for user in users:
            weekly_xp = sum(
                lesson['xpEarned'] for lesson in user.get('completedLessons', [])
                if datetime.fromisoformat(lesson['completedAt']) > week_ago
            )
            if weekly_xp > 0:
                weekly_scores.append({
                    'userId': user['userId'],
                    'username': user['username'],
                    'weeklyXp': weekly_xp,
                    'avatar': user.get('avatar')
                })
        
        return sorted(weekly_scores, key=lambda x: x['weeklyXp'], reverse=True)[:limit]
    
    def get_user_rank(self, user_id):
        """Get specific user's ranking"""
        leaderboard = self.get_global_leaderboard(limit=None)
        for index, user in enumerate(leaderboard, 1):
            if user['userId'] == user_id:
                return {
                    'rank': index,
                    'xp': user['xp'],
                    'level': user['level'],
                    'totalUsers': len(leaderboard)
                }
        return None
    
    def _load_all_users(self):
        """Load all user progress files"""
        users = []
        for filename in os.listdir(self.data_folder):
            if filename.startswith('user_') and filename.endswith('_progress.json'):
                with open(os.path.join(self.data_folder, filename), 'r') as f:
                    user_data = json.load(f)
                    user_id = filename.replace('user_', '').replace('_progress.json', '')
                    user_data['userId'] = user_id
                    users.append(user_data)
        return users
```

## Troubleshooting

### Flask Not Starting

```bash
# Check Python version
python --version  # Should be 3.7+

# Verify dependencies
pip install -r requirements.txt

# Check for port conflicts
lsof -i :5000  # On Linux/Mac
netstat -ano | findstr :5000  # On Windows

# Run with debug mode
export FLASK_DEBUG=1
flask run
```

### Data Not Persisting

```python
# Verify data folder permissions
import os
data_folder = 'data'
if not os.path.exists(data_folder):
    os.makedirs(data_folder, exist_ok=True)
    print(f"Created {data_folder}")

# Check write permissions
test_file = os.path.join(data_folder, 'test.txt')
try:
    with open(test_file, 'w') as f:
        f.write('test')
    os.remove(test_file)
    print("Write permissions OK")
except PermissionError:
    print("Permission denied - check folder permissions")
```

### Avatar Upload Issues

```python
# app.py - Enhanced avatar upload with validation
from werkzeug.utils import secure_filename
import imghdr

ALLOWED_EXTENSIONS = {'png', 'jpg', 'jpeg', 'gif'}

def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

@app.route('/api/users/<user_id>/avatar', methods=['POST'])
def upload_avatar(user_id):
    if 'avatar' not in request.files:
        return jsonify({'error': 'No file provided'}), 400
    
    file = request.files['avatar']
    if file.filename == '':
        return jsonify({'error': 'No file selected'}), 400
    
    if not allowed_file(file.filename):
        return jsonify({'error': 'Invalid file type'}), 400
    
    # Validate image
    file_bytes = file.read()
    if imghdr.what(None, h=file_bytes) not in ['png', 'jpg', 'jpeg', 'gif']:
        return jsonify({'error': 'File is not a valid image'}), 400
    
    file.seek(0)  # Reset file pointer
    
    filename = secure_filename(f"{user_id}_avatar.{file.filename.rsplit('.', 1)[1]}")
    filepath = os.path.join(app.config['UPLOAD_FOLDER'], filename)
    file.save(filepath)
    
    return jsonify({
        'avatarUrl': f'/static/uploads/avatars/{filename}',
        'success': True
    })
```

### Quiz Score Not Updating

```javascript
// Ensure proper async handling
async function submitQuizAndUpdate() {
  try {
    const quizEngine = new QuizEngine('lesson-123');
    const result = await quizEngine.completeQuiz();
    
    // Verify response
    if (!result.xpEarned) {
      console.error('XP not returned from server');
      return;
    }
    
    // Update UI
    updateProgressBar(result.totalXp);
    displayLevelUp(result.level, result.leveledUp);
    
    // Refresh profile
    const profile = new UserProfile();
    await profile.loadProfile();
    
  } catch (error) {
    console.error('Quiz submission failed:', error);
    alert('Failed to save quiz results. Please try again.');
  }
}
```

## Best Practices

1. **Always validate user input** on both client and server side
2. **Use environment variables** for sensitive configuration
3. **Implement proper error handling** for all API calls
4. **Store user data securely** with appropriate file permissions
5. **Regularly backup** the data folder
6. **Test quiz and lesson content** before deployment
7. **Monitor XP calculations** to prevent gaming the system
8. **Use transaction-like operations** when updating user progress
9. **Implement rate limiting** to prevent abuse
10. **Log important events** for debugging and analytics
