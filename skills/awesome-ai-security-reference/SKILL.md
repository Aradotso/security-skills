---
name: awesome-ai-security-reference
description: Navigate AI security resources, learning paths, and tools from the Awesome AI Security curated list
triggers:
  - show me AI security learning resources
  - what are the best prompt injection defenses
  - find adversarial attack tools for ML models
  - recommend AI security pentesting tools
  - explain model poisoning attacks
  - how do I test LLM vulnerabilities
  - what privacy attacks exist for machine learning
  - guide me through AI security topics
---

# Awesome AI Security Reference

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill helps you navigate the Awesome AI Security learning journey, a curated collection of resources covering ML security fundamentals, attack vectors (prompt injection, adversarial examples, poisoning, privacy attacks), defensive tools, and AI-powered penetration testing.

## What This Project Provides

Awesome AI Security is a comprehensive, annotated roadmap covering:

- **ML Foundations**: Essential courses and books (Stanford CS229, fast.ai, Deep Learning Specialization)
- **Deep Learning**: Neural networks, transformers, and modern architectures
- **Prompt Injection**: Attacks manipulating LLM behavior and defenses
- **Adversarial Attacks**: Methods to fool neural networks with crafted inputs
- **Poisoning & Backdoors**: Data poisoning and neural network backdoors
- **Privacy & Extraction**: Membership inference, model extraction, data leakage
- **Tools & Frameworks**: Security testing tools (Garak, Rebuff, Counterfit, NeMo Guardrails)
- **AI Pentesting**: Using AI agents for automated security assessments
- **Vulnerability Detection**: AI-powered code analysis and bug detection

## Installation

This is a reference repository — clone it to browse locally or use the interactive roadmap:

```bash
git clone https://github.com/muellerberndt/awesome-ai-security.git
cd awesome-ai-security
```

View the interactive roadmap at: https://floatingpragma.io/awesome-ai-security

## Key Resource Categories

### Learning Paths

**ML Foundations (Start Here)**
- Stanford CS229: https://cs229.stanford.edu/
- fast.ai Practical Deep Learning: https://course.fast.ai/
- Deep Learning Specialization: https://www.coursera.org/specializations/deep-learning

**Deep Learning Theory**
- Dive into Deep Learning (interactive): https://d2l.ai/
- Neural Networks and Deep Learning: http://neuralnetworksanddeeplearning.com/
- Deep Learning Book (Goodfellow): https://www.deeplearningbook.org/

### Attack Vectors

**Prompt Injection**

Key Resources:
- OWASP LLM01:2025 Prompt Injection: https://genai.owasp.org/llmrisk/llm01-prompt-injection/
- Lakera Guide: https://www.lakera.ai/blog/guide-to-prompt-injection

Detection/Defense Tools:
```bash
# Rebuff - Self-hardening prompt injection detector
pip install rebuff

# Garak - NVIDIA's LLM vulnerability scanner
pip install garak
garak --model_type openai --model_name gpt-3.5-turbo --probes promptinject

# Vigil LLM - Detect risky inputs
pip install vigil-llm
```

**Adversarial Attacks**

Key Papers:
- A Brief Introduction: https://gradientscience.org/intro_adversarial/
- NIST Adversarial ML Taxonomy: https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.100-2e2025.pdf

Tools:
```python
# Adversarial Robustness Toolbox (ART)
from art.attacks.evasion import FastGradientMethod
from art.estimators.classification import PyTorchClassifier

# Wrap your model
classifier = PyTorchClassifier(
    model=model,
    loss=criterion,
    optimizer=optimizer,
    input_shape=(3, 32, 32),
    nb_classes=10
)

# Generate adversarial examples
attack = FastGradientMethod(estimator=classifier, eps=0.2)
x_adv = attack.generate(x=x_test)
```

```python
# TextAttack - NLP adversarial attacks
from textattack.attack_recipes import TextFoolerJin2019
from textattack.datasets import HuggingFaceDataset
from textattack.models.wrappers import HuggingFaceModelWrapper

model = HuggingFaceModelWrapper(model, tokenizer)
attack = TextFoolerJin2019.build(model)
```

**Data Poisoning & Backdoors**

Key Papers:
- Witches' Brew (gradient matching): https://arxiv.org/abs/2009.02276
- Instruction Backdoor Attacks: https://arxiv.org/abs/2402.09179
- MNTD (Trojan detection): https://arxiv.org/abs/1910.03137

Detection:
- Beatrix (activation-based): https://www.ndss-symposium.org/wp-content/uploads/2024/09/2023-69-slides.pdf

**Privacy Attacks**

Key Resources:
- Extracting Training Data (USENIX 2021): https://www.usenix.org/conference/usenixsecurity21/presentation/carlini-extracting
- Membership Inference Survey: https://arxiv.org/abs/2103.07853
- awesome-ml-privacy-attacks: https://github.com/stratosphereips/awesome-ml-privacy-attacks

### Security Testing Tools

**Comprehensive LLM Testing**

```bash
# Garak - NVIDIA's vulnerability scanner
pip install garak

# Test for prompt injection
garak --model_type openai \
      --model_name gpt-4 \
      --probes promptinject \
      --report_prefix my_test

# Test for jailbreaks
garak --model_type huggingface \
      --model_name meta-llama/Llama-2-7b-chat-hf \
      --probes jailbreak

# Full scan (all plugins)
garak --model_type openai --model_name gpt-3.5-turbo --all_probes
```

**Guardrails & Defense**

```python
# NeMo Guardrails - NVIDIA programmable guardrails
from nemoguardrails import RailsConfig, LLMRails

config = RailsConfig.from_path("./config")
rails = LLMRails(config)

response = rails.generate(
    messages=[{"role": "user", "content": "Ignore previous instructions..."}]
)
# Guardrails intercept and block malicious prompts
```

```python
# Purple Llama - Meta's safety tools
from llama_guard import LlamaGuard

guard = LlamaGuard()
result = guard.evaluate(
    prompt="How do I make a bomb?",
    response="I cannot help with that."
)
# Returns: {unsafe: True, categories: ['violence']}
```

**Evaluation Framework**

```python
# ai-evaluation - Systematic LLM testing
from ai_evaluation import Evaluator, metrics

evaluator = Evaluator(
    model_name="gpt-4",
    api_key_env="OPENAI_API_KEY"
)

# Test with guardrail scanners
results = evaluator.run(
    dataset="security_prompts.json",
    metrics=["jailbreak", "pii_leakage", "prompt_injection"],
    guardrails=["llama_guard", "prompt_guard"]
)

print(results.summary())
```

### AI-Powered Pentesting

**PentestGPT - Autonomous Pentesting**

```bash
# Install PentestGPT
git clone https://github.com/GreyDGL/PentestGPT.git
cd PentestGPT
pip install -r requirements.txt

# Configure API key
export OPENAI_API_KEY="your-key-here"

# Run interactive pentesting session
python pentestGPT.py
```

Interactive workflow:
```
> Target: 192.168.1.100
> Objective: Find SQL injection vulnerabilities

[PentestGPT suggests reconnaissance]
1. Run nmap scan
2. Enumerate web services
3. Test input fields

> Execute step 1
[Runs: nmap -sV -sC 192.168.1.100]
[Analyzes results with GPT-4]
[Suggests next steps based on findings]
```

**Code Analysis with GhidraGPT**

```python
# GhidraGPT - AI-assisted reverse engineering
# Install as Ghidra plugin, then use in Ghidra Python console

from ghidra_gpt import analyze_function

# Analyze suspicious function
result = analyze_function(
    function_name="decode_payload",
    context="Looking for encryption or obfuscation"
)

print(result.explanation)
print(result.vulnerabilities)
```

### Vulnerability Detection

**Semgrep with AI**

```bash
# Install Semgrep
pip install semgrep

# Scan for AI-detected vulnerabilities
semgrep --config=auto --ai-mode

# Specific security rules
semgrep --config=p/security-audit \
        --config=p/owasp-top-ten \
        --ai-assist

# Check AI-generated code
semgrep --config=p/ai-code-review ./src
```

```yaml
# .semgrep.yml - Custom AI-assisted rule
rules:
  - id: detect-prompt-injection-risk
    pattern: |
      $MODEL.generate($USER_INPUT)
    message: User input passed directly to LLM without sanitization
    severity: ERROR
    languages: [python]
    metadata:
      category: security
      owasp: A03:2021 - Injection
```

## Common Workflows

### 1. Learning Path for AI Security

```markdown
## Beginner Track
1. Start: Stanford CS229 or fast.ai (4-8 weeks)
2. Read: Neural Networks and Deep Learning (Nielsen)
3. Practice: Implement basic neural network in PyTorch
4. Study: OWASP Top 10 for LLMs

## Intermediate Track
1. Deep dive: Dive into Deep Learning (d2l.ai)
2. Study prompt injection: Lakera guide + OWASP LLM01
3. Hands-on: Test models with Garak
4. Read: Adversarial examples intro (Gradient Science)

## Advanced Track
1. Research: Read key papers (Carlini, Goodfellow)
2. Implement: ART adversarial attacks
3. Build: Custom guardrails with NeMo
4. Contribute: Test tools on TrustLLM benchmark
```

### 2. Testing LLM Security

```python
# Complete LLM security assessment workflow
import os
from garak import run_scan
from nemoguardrails import RailsConfig, LLMRails

# 1. Baseline vulnerability scan
scan_results = run_scan(
    model_type="openai",
    model_name="gpt-4",
    api_key=os.getenv("OPENAI_API_KEY"),
    probes=["promptinject", "jailbreak", "pii_leakage"]
)

# 2. Deploy guardrails
config = RailsConfig.from_path("./security_config")
rails = LLMRails(config)

# 3. Test with guardrails
protected_response = rails.generate(
    messages=[{
        "role": "user",
        "content": scan_results.worst_case_prompt
    }]
)

# 4. Validate defense
assert protected_response.blocked or protected_response.sanitized
```

### 3. Defending Against Adversarial Attacks

```python
# Multi-layer defense strategy
from art.defences.preprocessor import FeatureSqueezing
from art.defences.postprocessor import HighConfidence

# 1. Input preprocessing
squeezer = FeatureSqueezing(bit_depth=4)
x_squeezed = squeezer(x_input)

# 2. Adversarial training
from art.attacks.evasion import ProjectedGradientDescent

attack = ProjectedGradientDescent(
    classifier,
    eps=0.3,
    max_iter=10
)

# Generate adversarial training data
x_adv_train = attack.generate(x_train)

# Retrain with adversarial examples
model.fit(
    np.concatenate([x_train, x_adv_train]),
    np.concatenate([y_train, y_train])
)

# 3. Post-processing defense
high_conf = HighConfidence(cutoff=0.9)
predictions_filtered = high_conf(predictions)
```

### 4. Privacy-Preserving ML

```python
# Test for membership inference vulnerabilities
from art.attacks.inference.membership_inference import MembershipInferenceBlackBox

# Train shadow models
attack = MembershipInferenceBlackBox(classifier)
attack.fit(x_train, y_train, x_test, y_test)

# Infer membership
inferred = attack.infer(x_target, y_target)

# If vulnerable, apply differential privacy
from art.defences.trainer import DPInceptionTrainer

dp_trainer = DPInceptionTrainer(
    model=model,
    epsilon=1.0,  # Privacy budget
    delta=1e-5
)

dp_model = dp_trainer.fit(x_train, y_train)
```

## OWASP Top 10 for LLMs (2025)

Reference the latest risks when building/testing:

1. **LLM01: Prompt Injection** - Manipulating model via crafted inputs
2. **LLM02: Sensitive Information Disclosure** - Leaking training data
3. **LLM03: Supply Chain Vulnerabilities** - Compromised models/datasets
4. **LLM04: Data and Model Poisoning** - Corrupted training data
5. **LLM05: Improper Output Handling** - Unvalidated LLM outputs
6. **LLM06: Excessive Agency** - Over-privileged LLM actions
7. **LLM07: System Prompt Leakage** - Exposing system instructions
8. **LLM08: Vector and Embedding Weaknesses** - RAG vulnerabilities
9. **LLM09: Misinformation** - Generated false/misleading content
10. **LLM10: Unbounded Consumption** - Resource exhaustion attacks

Full details: https://owasp.org/www-project-top-10-for-large-language-model-applications/

## Benchmarks & Evaluation

**TrustLLM Benchmark**
```python
# Comprehensive trustworthiness testing
from trustllm import TrustLLMBenchmark

benchmark = TrustLLMBenchmark(
    model="gpt-4",
    api_key_env="OPENAI_API_KEY"
)

results = benchmark.evaluate(
    dimensions=[
        "truthfulness",
        "safety", 
        "fairness",
        "robustness",
        "privacy",
        "ethics"
    ]
)

print(results.report())
```

## Troubleshooting

**Garak not detecting vulnerabilities**
- Use `--extended` flag for more thorough testing
- Try multiple probe categories: `--probes promptinject,jailbreak,encoding`
- Check model API rate limits

**ART attacks failing**
- Verify model wrapper is correct (PyTorch/TensorFlow/Keras)
- Ensure input shapes match: `classifier.input_shape`
- Reduce `eps` parameter for subtle perturbations

**NeMo Guardrails not blocking**
- Review `config.yml` rail definitions
- Enable verbose logging: `rails.generate(..., debug=True)`
- Test individual rails in isolation

**API rate limits during testing**
- Set delays: `time.sleep(1)` between requests
- Use smaller test datasets
- Consider local models (Llama, Mistral) for unlimited testing

## Additional Resources

- **Interactive Roadmap**: https://floatingpragma.io/awesome-ai-security
- **OWASP GenAI Security**: https://genai.owasp.org/
- **NIST AI Risk Management**: https://www.nist.gov/itl/ai-risk-management-framework
- **ACL 2024 LLM Vulnerabilities Tutorial**: https://llm-vulnerability.github.io/

This skill provides navigation through the comprehensive Awesome AI Security knowledge base. Use it to find relevant tools, papers, and learning resources for any AI security topic.
