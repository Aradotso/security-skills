---
name: ai-security-knowledge-base
description: Comprehensive AI security knowledge base covering ML algorithms, threat modeling, offensive AI tactics, and defensive strategies including OWASP LLM Top 10 and adversarial ML
triggers:
  - "teach me about AI security vulnerabilities"
  - "how do I defend against prompt injection attacks"
  - "explain adversarial machine learning techniques"
  - "show me AI red team offensive tactics"
  - "what are the OWASP LLM top 10 risks"
  - "help me understand AI model poisoning"
  - "guide me through AI security best practices"
  - "how to detect deepfake and voice cloning attacks"
---

# AI Security Knowledge Base Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

The AI Security Knowledge Base (AI_Security_Top) is a comprehensive Chinese-language security documentation project that covers the complete spectrum of AI security from foundational machine learning algorithms to advanced threats like prompt injection, adversarial attacks, model evasion, and deepfake exploitation. This project serves as both a reference guide and practical resource for security researchers, penetration testers, and AI developers.

### Core Coverage Areas

- **AI Fundamentals**: Deep learning architectures, supervised/unsupervised learning, reinforcement learning
- **Threat Modeling**: OWASP ML/LLM Top 10, MCP security, skill security frameworks
- **Red Team Operations**: Offensive AI tactics, adversarial ML, deepfake generation, AI-powered penetration testing
- **Blue Team Defense**: ML-based threat detection, automated auditing, defensive algorithms

## Installation

This is a documentation/knowledge repository, not a code library. Clone it for reference:

```bash
# Clone the repository
git clone https://github.com/GhostWolfLab/AI_Security_Top.git
cd AI_Security_Top

# The repository contains markdown documentation files
ls *.md
# Expected output: AI.md, 深度学习.md, 监督学习算法.md, AI安全.md, MCP安全.md, 进攻性AI.md, 算法赋能安全.md, etc.
```

## Key Documentation Structure

### 1. Foundational AI Concepts

**AI.md** - Artificial Intelligence overview, history, and core concepts

**深度学习.md** - Deep learning architectures (CNN, RNN, Transformer)

**监督学习算法.md** - Supervised learning: Linear/Logistic Regression, SVM, Decision Trees, Random Forests, Naive Bayes

**无监督学习算法.md** - Unsupervised learning: K-Means, GMM clustering, association rules

**强化学习.md** - Reinforcement learning: Agent-based decision making and game theory

### 2. AI Security Frameworks

**AI安全.md** - OWASP ML/LLM Top 10 vulnerabilities including:
- Prompt Injection (direct and indirect)
- Insecure Output Handling
- Training Data Poisoning
- Model Denial of Service
- Supply Chain Vulnerabilities
- Sensitive Information Disclosure
- Insecure Plugin Design
- Excessive Agency
- Overreliance
- Model Theft

**MCP安全.md** - Model Context Protocol security:
- Tool poisoning attacks
- Cursor IDE vulnerabilities
- Protocol-level exploitation

**skill安全.md** - AI skill security configurations and defensive practices

### 3. Offensive AI Tactics

**进攻性AI.md** - Red team operations including:

#### AI-Powered Vulnerability Scanning
```python
# Example: Using AI for automated reconnaissance
# Tools referenced: Shennina, Shannon

import os
from ai_scanner import AutomatedPentest

# Initialize AI-driven scanner
scanner = AutomatedPentest(
    target=os.getenv('TARGET_URL'),
    api_key=os.getenv('AI_API_KEY')
)

# Run inference-based vulnerability detection
results = scanner.scan(
    modes=['xss', 'sqli', 'ssrf'],
    reasoning_depth='deep'
)

# Generate exploitation suggestions
exploits = scanner.suggest_exploits(results)
```

#### Adversarial ML Evasion
```python
# Example: Malware mutation for AV evasion
# Tools referenced: Pesidious, MalwareGAN

from adversarial_ml import RLMalwareMutator

# Load base malware sample
mutator = RLMalwareMutator(
    model_path='models/evasion_rl.pth'
)

# Generate evasion variants
mutated_samples = mutator.evolve(
    original_binary='payload.exe',
    target_av=['defender', 'crowdstrike'],
    iterations=100
)

# Test evasion success rate
detection_rate = mutator.test_against_av(mutated_samples)
```

#### Deepfake Generation
```python
# Example: Real-time face swapping
# Tools referenced: Deep-Live-Cam

from deepfake import LiveFaceSwap
import cv2

# Initialize real-time deepfake model
swapper = LiveFaceSwap(
    source_image='target_face.jpg',
    model='models/faceswap_256.onnx'
)

# Process video stream
cap = cv2.VideoCapture(0)
while True:
    ret, frame = cap.read()
    deepfaked = swapper.swap_face(frame)
    cv2.imshow('Deepfake Output', deepfaked)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

#### Voice Cloning
```python
# Example: Voice synthesis for social engineering
# Tools referenced: RVC-WebUI, Voice-Pro

from voice_clone import VoiceSynthesizer

# Train on target voice samples
synthesizer = VoiceSynthesizer()
synthesizer.train(
    samples_dir='voice_samples/',
    target_speaker='ceo_voice',
    epochs=500
)

# Generate synthetic audio
synthetic_audio = synthesizer.speak(
    text="Please approve the wire transfer immediately.",
    output_path='phishing_audio.wav'
)
```

### 4. Defensive AI Applications

**算法赋能安全.md** - Blue team ML applications:

#### Anomaly Detection
```python
# Example: Network traffic anomaly detection
from sklearn.ensemble import IsolationForest
import pandas as pd
import numpy as np

# Load network traffic features
traffic_data = pd.read_csv('network_logs.csv')
features = ['packet_size', 'duration', 'protocol', 'port']

# Train isolation forest
detector = IsolationForest(
    contamination=0.1,
    random_state=42
)
detector.fit(traffic_data[features])

# Detect anomalies in real-time
def detect_threat(packet):
    features = np.array([[
        packet['size'],
        packet['duration'],
        packet['protocol_id'],
        packet['port']
    ]])
    prediction = detector.predict(features)
    return prediction[0] == -1  # -1 indicates anomaly
```

#### Automated Threat Hunting
```python
# Example: ML-based log analysis
from transformers import AutoTokenizer, AutoModelForSequenceClassification
import torch

# Load pre-trained security log classifier
tokenizer = AutoTokenizer.from_pretrained('security-logs-bert')
model = AutoModelForSequenceClassification.from_pretrained('security-logs-bert')

def classify_log_event(log_entry):
    inputs = tokenizer(
        log_entry,
        return_tensors='pt',
        truncation=True,
        max_length=512
    )
    
    with torch.no_grad():
        outputs = model(**inputs)
        prediction = torch.argmax(outputs.logits, dim=1)
    
    labels = ['benign', 'suspicious', 'malicious']
    return labels[prediction.item()]

# Analyze security logs
log = "Failed login attempt from 192.168.1.100 after 50 retries"
threat_level = classify_log_event(log)
```

## Common Attack Patterns

### Prompt Injection Defense

```python
# Example: Detecting and sanitizing prompt injections
import re

def detect_prompt_injection(user_input):
    """
    Detect common prompt injection patterns
    """
    injection_patterns = [
        r'ignore\s+(previous|above|prior)\s+instructions',
        r'system\s*:',
        r'<\|.*?\|>',
        r'###\s*instruction',
        r'you\s+are\s+now',
        r'forget\s+everything'
    ]
    
    for pattern in injection_patterns:
        if re.search(pattern, user_input, re.IGNORECASE):
            return True
    return False

def sanitize_input(user_input):
    """
    Sanitize potential injection attempts
    """
    if detect_prompt_injection(user_input):
        # Log the attempt
        print(f"[ALERT] Prompt injection detected: {user_input[:100]}")
        return None
    
    # Additional sanitization
    sanitized = user_input.replace('```', '').strip()
    return sanitized

# Usage in LLM application
user_query = input("Enter your question: ")
safe_query = sanitize_input(user_query)

if safe_query:
    # Process with LLM
    response = llm.generate(safe_query)
else:
    response = "Invalid input detected. Please rephrase your query."
```

### Model Poisoning Detection

```python
# Example: Detecting poisoned training data
from sklearn.ensemble import IsolationForest
import numpy as np

def detect_data_poisoning(training_data, labels):
    """
    Identify potential poisoned samples in training data
    """
    # Extract feature statistics
    feature_stats = np.column_stack([
        training_data.mean(axis=1),
        training_data.std(axis=1),
        training_data.max(axis=1),
        training_data.min(axis=1)
    ])
    
    # Detect outliers
    detector = IsolationForest(contamination=0.05)
    predictions = detector.fit_predict(feature_stats)
    
    # Identify poisoned indices
    poisoned_indices = np.where(predictions == -1)[0]
    
    return poisoned_indices

# Clean training data
X_train = np.load('training_features.npy')
y_train = np.load('training_labels.npy')

poisoned_idx = detect_data_poisoning(X_train, y_train)
print(f"Detected {len(poisoned_idx)} potentially poisoned samples")

# Remove poisoned data
X_clean = np.delete(X_train, poisoned_idx, axis=0)
y_clean = np.delete(y_train, poisoned_idx, axis=0)
```

## Configuration Best Practices

### Secure LLM Deployment

```python
# Example: Secure LLM API configuration
from fastapi import FastAPI, HTTPException, Depends
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
import os
import hashlib
import time

app = FastAPI()
security = HTTPBearer()

# Rate limiting
request_cache = {}

def verify_token(credentials: HTTPAuthorizationCredentials = Depends(security)):
    """Verify API token"""
    expected_token = os.getenv('API_SECRET_TOKEN')
    token_hash = hashlib.sha256(credentials.credentials.encode()).hexdigest()
    expected_hash = hashlib.sha256(expected_token.encode()).hexdigest()
    
    if token_hash != expected_hash:
        raise HTTPException(status_code=403, detail="Invalid token")
    return credentials.credentials

def rate_limit(client_id: str, max_requests: int = 10, window: int = 60):
    """Implement rate limiting"""
    current_time = time.time()
    
    if client_id in request_cache:
        requests = [t for t in request_cache[client_id] if current_time - t < window]
        if len(requests) >= max_requests:
            raise HTTPException(status_code=429, detail="Rate limit exceeded")
        requests.append(current_time)
        request_cache[client_id] = requests
    else:
        request_cache[client_id] = [current_time]

@app.post("/generate")
async def generate_text(
    prompt: str,
    token: str = Depends(verify_token)
):
    client_id = hashlib.md5(token.encode()).hexdigest()
    rate_limit(client_id)
    
    # Input validation
    if len(prompt) > 2000:
        raise HTTPException(status_code=400, detail="Prompt too long")
    
    # Sanitize and process
    from prompt_guard import sanitize_input
    safe_prompt = sanitize_input(prompt)
    
    # Generate with guardrails
    response = llm.generate(
        safe_prompt,
        max_tokens=500,
        temperature=0.7,
        top_p=0.9
    )
    
    return {"response": response}
```

## Troubleshooting

### Common Issues

**Issue**: Prompt injection bypassing detection
```python
# Solution: Implement multi-layer defense
def advanced_prompt_guard(user_input, context=""):
    # Layer 1: Pattern matching
    if detect_prompt_injection(user_input):
        return None
    
    # Layer 2: Semantic analysis
    from transformers import pipeline
    classifier = pipeline("text-classification", model="prompt-injection-detector")
    result = classifier(user_input)[0]
    
    if result['label'] == 'INJECTION' and result['score'] > 0.8:
        return None
    
    # Layer 3: Context validation
    if context and not validate_context(user_input, context):
        return None
    
    return user_input
```

**Issue**: Model evasion in production
```python
# Solution: Implement adversarial training
from art.attacks.evasion import FastGradientMethod
from art.estimators.classification import PyTorchClassifier

def adversarial_training(model, train_loader, epochs=10):
    classifier = PyTorchClassifier(
        model=model,
        loss=torch.nn.CrossEntropyLoss(),
        optimizer=torch.optim.Adam(model.parameters()),
        input_shape=(3, 224, 224),
        nb_classes=10
    )
    
    # Generate adversarial examples
    attack = FastGradientMethod(estimator=classifier, eps=0.1)
    
    for epoch in range(epochs):
        for batch_x, batch_y in train_loader:
            # Train on both clean and adversarial examples
            adv_x = attack.generate(x=batch_x.numpy())
            
            # Combined training
            combined_x = torch.cat([batch_x, torch.from_numpy(adv_x)])
            combined_y = torch.cat([batch_y, batch_y])
            
            classifier.fit(combined_x, combined_y, nb_epochs=1)
```

**Issue**: Deepfake detection failure
```python
# Solution: Multi-modal deepfake detection
import cv2
import numpy as np
from transformers import ViTForImageClassification

def detect_deepfake(video_path):
    """
    Multi-modal deepfake detection combining:
    - Visual artifacts analysis
    - Temporal inconsistency detection
    - Audio-visual mismatch
    """
    cap = cv2.VideoCapture(video_path)
    
    # Visual deepfake detector
    model = ViTForImageClassification.from_pretrained('deepfake-detector-vit')
    
    frame_predictions = []
    frame_count = 0
    
    while cap.isOpened() and frame_count < 100:
        ret, frame = cap.read()
        if not ret:
            break
        
        # Extract face regions
        faces = extract_faces(frame)
        
        for face in faces:
            # Predict on each face
            prediction = model(face)
            frame_predictions.append(prediction['score'])
        
        frame_count += 1
    
    cap.release()
    
    # Aggregate predictions
    avg_score = np.mean(frame_predictions)
    is_deepfake = avg_score > 0.7
    
    return {
        'is_deepfake': is_deepfake,
        'confidence': avg_score,
        'frames_analyzed': frame_count
    }
```

## Integration Examples

### Automated Security Scanning Pipeline

```python
# Complete AI security scanning workflow
from ai_security_tools import (
    VulnerabilityScanner,
    PromptInjectionTester,
    ModelRobustnessTester
)

class AISecurityPipeline:
    def __init__(self, target_model, target_api):
        self.model = target_model
        self.api = target_api
        self.scanner = VulnerabilityScanner()
        self.prompt_tester = PromptInjectionTester()
        self.robustness_tester = ModelRobustnessTester()
    
    def run_full_assessment(self):
        results = {
            'api_vulnerabilities': [],
            'prompt_injection_tests': [],
            'model_robustness': []
        }
        
        # 1. API security scan
        print("[*] Scanning API endpoints...")
        api_vulns = self.scanner.scan_api(self.api)
        results['api_vulnerabilities'] = api_vulns
        
        # 2. Prompt injection testing
        print("[*] Testing prompt injection vectors...")
        injection_tests = self.prompt_tester.test_all_vectors(self.api)
        results['prompt_injection_tests'] = injection_tests
        
        # 3. Model robustness testing
        print("[*] Testing model robustness...")
        robustness = self.robustness_tester.test_adversarial(self.model)
        results['model_robustness'] = robustness
        
        return self.generate_report(results)
    
    def generate_report(self, results):
        critical = sum(1 for v in results['api_vulnerabilities'] if v['severity'] == 'critical')
        
        report = f"""
        AI Security Assessment Report
        ============================
        
        API Vulnerabilities Found: {len(results['api_vulnerabilities'])}
        - Critical: {critical}
        
        Prompt Injection Success Rate: {self.calculate_success_rate(results['prompt_injection_tests'])}%
        
        Model Robustness Score: {results['model_robustness']['score']}/100
        
        Recommendations:
        {self.generate_recommendations(results)}
        """
        
        return report

# Usage
pipeline = AISecurityPipeline(
    target_model='models/production_llm.pt',
    target_api=os.getenv('TARGET_API_URL')
)

report = pipeline.run_full_assessment()
print(report)
```

## References

- **OWASP LLM Top 10**: https://owasp.org/www-project-top-10-for-large-language-model-applications/
- **Adversarial Robustness Toolbox**: https://github.com/Trusted-AI/adversarial-robustness-toolbox
- **Project Repository**: https://github.com/GhostWolfLab/AI_Security_Top

---

**License**: MIT  
**Language**: Chinese (Documentation), Python (Code Examples)  
**Last Updated**: 2026-03-06
