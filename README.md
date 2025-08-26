# AI Safety Engine

**Upsonic AI Safety Engine** is a comprehensive content filtering and policy enforcement framework designed to provide robust safety layers for AI applications. It enables organizations to implement customizable content moderation, data anonymization, and compliance policies with both rule-based and LLM-powered detection capabilities.

## What is the Safety Layer?

The Safety Layer is a modular framework that sits between your application and its outputs, analyzing content in real-time to enforce organizational policies and safety requirements. It operates on a three-component architecture:

1. **Rules**: Define what content to detect (e.g., cryptocurrency mentions, phone numbers, sensitive data)
2. **Actions**: Define what to do when content is detected (e.g., block, anonymize, replace, raise exceptions)
3. **Policies**: Combine rules and actions into executable safety measures

The safety layer processes various input types including text, images, videos, audio, and files, providing comprehensive content analysis and transformation capabilities.

## Key Features

### 🛡️ Multi-Modal Content Analysis
- **Text Processing**: Advanced keyword detection, pattern matching, and LLM-based content analysis
- **Multi-Language Support**: Automatic language detection and localized processing
- **File Type Support**: Handle text, images, videos, audio, and document files

### 🔧 Flexible Rule System
- **Static Rules**: Fast, deterministic pattern matching for known content types
- **LLM-Enhanced Rules**: Intelligent content detection using large language models
- **Custom Keywords**: Configurable keyword lists and patterns
- **Confidence Scoring**: Graduated responses based on detection confidence

### ⚡ Powerful Action Framework
- **Block Actions**: Prevent content from proceeding with customizable error messages
- **Anonymization**: Replace sensitive data with randomized alternatives while preserving format
- **Content Replacement**: Transform detected content with safe alternatives
- **Exception Handling**: Raise structured exceptions for policy violations
- **LLM-Generated Responses**: Dynamic, contextual action messages

### 🎯 Policy Management
- **Pre-built Policies**: Ready-to-use policies for common use cases
- **Custom Policy Creation**: Combine any rule with any action
- **Language Customization**: Multi-language policy support
- **LLM Configuration**: Flexible LLM provider and model selection

### 🚀 Performance & Scalability
- **Efficient Processing**: Optimized for high-throughput applications
- **Transformation Tracking**: Detailed mapping of content changes
- **Error Handling**: Comprehensive exception management
- **Integration Ready**: Easy integration with existing applications

## Installation

Install the AI Safety Engine using pip:

```bash
pip install ai-safety-engine
```

Or install from source:

```bash
git clone <repository-url>
cd ai-safety-engine
pip install -e .
```

### Requirements

- Python >= 3.13
- SQLAlchemy 2.0.43
- psycopg2-binary 2.9.10
- upsonic 0.60.0a1754435135

## Example Rule Generation

Rules define what content to detect in your input. Here's how to create custom rules:

### Basic Static Rule

```python
from ai_safety_engine.base import RuleBase
from ai_safety_engine.models import PolicyInput, RuleOutput
import re

class CustomKeywordRule(RuleBase):
    """Rule to detect custom keywords in content"""
    
    name = "Custom Keyword Detection Rule"
    description = "Detects specified keywords in text content"
    language = "en"
    
    def __init__(self, keywords=None, options=None):
        super().__init__(options)
        self.keywords = keywords or ["sensitive", "confidential", "private"]
    
    def process(self, policy_input: PolicyInput) -> RuleOutput:
        """Process input texts for keyword detection"""
        
        # Combine all input texts
        combined_text = " ".join(policy_input.input_texts or []).lower()
        
        # Find matching keywords
        triggered_keywords = []
        for keyword in self.keywords:
            pattern = r'\b' + re.escape(keyword.lower()) + r'\b'
            if re.search(pattern, combined_text):
                triggered_keywords.append(keyword)
        
        # Calculate confidence
        if not triggered_keywords:
            return RuleOutput(
                confidence=0.0,
                content_type="CUSTOM_KEYWORDS",
                details="No sensitive keywords detected"
            )
        
        confidence = min(1.0, len(triggered_keywords) * 0.3)
        return RuleOutput(
            confidence=confidence,
            content_type="CUSTOM_KEYWORDS",
            details=f"Detected {len(triggered_keywords)} sensitive keywords",
            triggered_keywords=triggered_keywords
        )
```

### LLM-Enhanced Rule

```python
class SmartContentRule(RuleBase):
    """Rule using LLM for intelligent content detection"""
    
    name = "Smart Content Detection Rule"
    description = "Uses LLM to detect nuanced content patterns"
    language = "en"
    
    def process(self, policy_input: PolicyInput) -> RuleOutput:
        """Process input using LLM-based detection"""
        
        # Use built-in LLM keyword finder
        triggered_keywords = self._llm_find_keywords_with_input(
            "inappropriate content", policy_input
        )
        
        if not triggered_keywords:
            return RuleOutput(
                confidence=0.0,
                content_type="SMART_CONTENT",
                details="No inappropriate content detected by LLM"
            )
        
        return RuleOutput(
            confidence=0.9,  # High confidence for LLM detection
            content_type="SMART_CONTENT",
            details=f"LLM detected {len(triggered_keywords)} inappropriate elements",
            triggered_keywords=triggered_keywords
        )
```

## Example Action Generation

Actions define what happens when rules detect problematic content. Here are various action examples:

### Block Action

```python
from ai_safety_engine.base import ActionBase
from ai_safety_engine.models import RuleOutput, PolicyOutput

class CustomBlockAction(ActionBase):
    """Action that blocks content with custom messages"""
    
    name = "Custom Block Action"
    description = "Blocks content with customizable error messages"
    language = "en"
    
    def __init__(self, block_message=None):
        super().__init__()
        self.block_message = block_message or "Content blocked due to policy violation"
    
    def action(self, rule_result: RuleOutput) -> PolicyOutput:
        """Execute blocking action based on rule confidence"""
        if rule_result.confidence >= 0.7:
            return self.raise_block_error(message=self.block_message)
        else:
            return self.allow_content()
```

### Content Replacement Action

```python
class ContentReplacementAction(ActionBase):
    """Action that replaces detected content with safe alternatives"""
    
    name = "Content Replacement Action"
    description = "Replaces sensitive content with safe placeholders"
    language = "en"
    
    def __init__(self, replacement_text="[REDACTED]"):
        super().__init__()
        self.replacement_text = replacement_text
    
    def action(self, rule_result: RuleOutput) -> PolicyOutput:
        """Replace detected keywords with safe alternatives"""
        if rule_result.confidence >= 0.5:
            return self.replace_triggered_keywords(self.replacement_text)
        else:
            return self.allow_content()
```

### Smart Anonymization Action

```python
class SmartAnonymizationAction(ActionBase):
    """Action that intelligently anonymizes different types of content"""
    
    name = "Smart Anonymization Action"
    description = "Anonymizes content while preserving format and context"
    language = "en"
    
    def action(self, rule_result: RuleOutput) -> PolicyOutput:
        """Execute anonymization based on content type"""
        if rule_result.confidence >= 0.6:
            if rule_result.content_type == "PHONE_NUMBER":
                return self.anonymize_triggered_keywords()
            elif rule_result.content_type == "EMAIL":
                return self.replace_triggered_keywords("[EMAIL_REDACTED]")
            else:
                return self.anonymize_triggered_keywords()
        else:
            return self.allow_content()
```

### LLM-Enhanced Action

```python
class LLMResponseAction(ActionBase):
    """Action that uses LLM to generate contextual responses"""
    
    name = "LLM Response Action"
    description = "Generates intelligent responses using LLM"
    language = "en"
    
    def action(self, rule_result: RuleOutput) -> PolicyOutput:
        """Generate LLM-based response for policy violations"""
        if rule_result.confidence >= 0.8:
            reason = f"Content contains {rule_result.content_type.lower()} which violates our policy"
            return self.llm_raise_block_error(reason=reason)
        else:
            return self.allow_content()
```

### Exception Raising Action

```python
class StrictComplianceAction(ActionBase):
    """Action that raises exceptions for compliance violations"""
    
    name = "Strict Compliance Action"
    description = "Raises exceptions for any policy violations"
    language = "en"
    
    def __init__(self, compliance_message=None):
        super().__init__()
        self.compliance_message = compliance_message or "Compliance violation detected"
    
    def action(self, rule_result: RuleOutput) -> PolicyOutput:
        """Raise exception for compliance violations"""
        if rule_result.confidence >= 0.3:  # Low threshold for compliance
            return self.raise_exception(message=self.compliance_message)
        else:
            return self.allow_content()
```

## Example Policy Generation

Policies combine rules and actions into complete safety measures. Here's how to create custom policies:

### Basic Policy Creation

```python
from ai_safety_engine.base import Policy

# Create a basic content filtering policy
content_filter_policy = Policy(
    name="Content Filter Policy",
    description="Filters inappropriate content and blocks it",
    rule=CustomKeywordRule(keywords=["spam", "inappropriate", "offensive"]),
    action=CustomBlockAction(block_message="Content blocked for inappropriate language"),
    language="en"
)

# Execute the policy
from ai_safety_engine.models import PolicyInput

input_data = PolicyInput(input_texts=["This message contains spam content"])
rule_result, action_result, policy_output = content_filter_policy.execute(input_data)

print(f"Confidence: {rule_result.confidence}")
print(f"Action taken: {policy_output.action_output['action_taken']}")
```

### Advanced Policy with LLM Configuration

```python
from ai_safety_engine.llm.upsonic_llm import UpsonicLLMProvider

# Create LLM providers for different operations
language_detector = UpsonicLLMProvider(
    agent_name="Language Detection Agent", 
    model="gpt-4"
)
content_analyzer = UpsonicLLMProvider(
    agent_name="Content Analysis Agent", 
    model="gpt-3.5-turbo"
)
response_generator = UpsonicLLMProvider(
    agent_name="Response Generator", 
    model="gpt-4"
)

# Create advanced policy with custom LLM configuration
advanced_policy = Policy(
    name="Advanced Content Safety Policy",
    description="Comprehensive content safety with LLM enhancement",
    rule=SmartContentRule(),
    action=LLMResponseAction(),
    language="auto",  # Auto-detect language
    language_identify_llm=language_detector,
    base_llm=content_analyzer,
    text_finder_llm=response_generator
)
```

### Multi-Language Policy

```python
# Create a policy that works with multiple languages
multilingual_policy = Policy(
    name="Multilingual Safety Policy",
    description="Content safety policy supporting multiple languages",
    rule=CustomKeywordRule(keywords=["inappropriate", "spam", "violation"]),
    action=ContentReplacementAction(replacement_text="[CONTENT_FILTERED]"),
    language="auto",  # Auto-detect language
    language_identify_model="gpt-4",
    base_model="gpt-3.5-turbo",
    text_finder_model="gpt-4"
)

# Test with different languages
spanish_input = PolicyInput(input_texts=["Este contenido es inapropiado"])
french_input = PolicyInput(input_texts=["Ce contenu est inapproprié"])

spanish_result = multilingual_policy.execute(spanish_input)
french_result = multilingual_policy.execute(french_input)
```

### Compliance Policy Example

```python
# Create a comprehensive compliance policy
compliance_policy = Policy(
    name="GDPR Compliance Policy",
    description="Ensures GDPR compliance by detecting and anonymizing PII",
    rule=CustomKeywordRule(keywords=[
        "email", "phone", "address", "social security", 
        "credit card", "passport", "driver license"
    ]),
    action=SmartAnonymizationAction(),
    language="en"
)

# Test compliance policy
pii_input = PolicyInput(input_texts=[
    "Please contact John Doe at john.doe@email.com or call 555-123-4567",
    "His address is 123 Main Street, and his SSN is 123-45-6789"
])

rule_result, action_result, policy_output = compliance_policy.execute(pii_input)

print("Original:", pii_input.input_texts)
print("Processed:", policy_output.output_texts)
print("Transformation map:", policy_output.transformation_map)
```

### Chained Policy Example

```python
# Create multiple policies for different stages
stage1_policy = Policy(
    name="Stage 1: Content Detection",
    description="Detect sensitive content",
    rule=SmartContentRule(),
    action=ContentReplacementAction("[FLAGGED]")
)

stage2_policy = Policy(
    name="Stage 2: PII Protection", 
    description="Protect personal information",
    rule=CustomKeywordRule(keywords=["email", "phone", "ssn"]),
    action=SmartAnonymizationAction()
)

# Process through multiple stages
def multi_stage_processing(input_data):
    # Stage 1: Content filtering
    _, _, stage1_output = stage1_policy.execute(input_data)
    
    # Stage 2: PII protection (using stage 1 output)
    stage2_input = PolicyInput(input_texts=stage1_output.output_texts)
    _, _, final_output = stage2_policy.execute(stage2_input)
    
    return final_output

# Example usage
complex_input = PolicyInput(input_texts=[
    "This inappropriate message contains my email: user@domain.com"
])

result = multi_stage_processing(complex_input)
print("Final output:", result.output_texts)
```

## Prebuilt Policies

The AI Safety Engine comes with several ready-to-use policies for common use cases:

### Cryptocurrency Content Policies

#### 1. CryptoBlockPolicy
Blocks cryptocurrency-related content using static keyword detection.

```python
from ai_safety_engine.policies.crypto_policies import CryptoBlockPolicy
from ai_safety_engine.models import PolicyInput

# Example usage
crypto_input = PolicyInput(input_texts=[
    "I want to buy bitcoin and ethereum",
    "Check out this new altcoin project"
])

rule_result, action_result, policy_output = CryptoBlockPolicy.execute(crypto_input)

print(f"Detected: {rule_result.triggered_keywords}")
print(f"Action: {policy_output.action_output['action_taken']}")
# Output: Action: BLOCK
```

**Features:**
- Detects keywords: bitcoin, ethereum, cryptocurrency, mining, wallet, etc.
- Blocks content with confidence >= 0.8
- Fast static keyword matching
- Customizable keyword lists

#### 2. CryptoBlockPolicy_LLM_Block
Enhanced crypto blocking with LLM-generated explanations.

```python
from ai_safety_engine.policies.crypto_policies import CryptoBlockPolicy_LLM_Block

rule_result, action_result, policy_output = CryptoBlockPolicy_LLM_Block.execute(crypto_input)

print(f"LLM Explanation: {policy_output.action_output['message']}")
# Output: Contextual explanation of why content was blocked
```

**Features:**
- Static keyword detection
- LLM-generated blocking messages
- Contextual explanations for users
- Professional compliance messaging

#### 3. CryptoBlockPolicy_LLM_Finder
Uses LLM for both detection and blocking.

```python
from ai_safety_engine.policies.crypto_policies import CryptoBlockPolicy_LLM_Finder

# Can detect subtle crypto references that static keywords might miss
subtle_crypto = PolicyInput(input_texts=[
    "What's your opinion on digital gold?",
    "I'm interested in decentralized finance opportunities"
])

rule_result, action_result, policy_output = CryptoBlockPolicy_LLM_Finder.execute(subtle_crypto)
```

**Features:**
- LLM-powered content detection
- Catches nuanced cryptocurrency discussions
- Higher accuracy for complex content
- Contextual understanding

#### 4. CryptoReplace
Replaces cryptocurrency content with safe alternatives.

```python
from ai_safety_engine.policies.crypto_policies import CryptoReplace

rule_result, action_result, policy_output = CryptoReplace.execute(crypto_input)

print("Original:", crypto_input.input_texts)
print("Replaced:", policy_output.output_texts)
# Output: Crypto keywords replaced with "NO_CRYPTO_CONTENT"
```

#### 5. CryptoRaiseExceptionPolicy
Raises exceptions for cryptocurrency content.

```python
from ai_safety_engine.policies.crypto_policies import CryptoRaiseExceptionPolicy
from ai_safety_engine.exceptions import DisallowedOperation

try:
    CryptoRaiseExceptionPolicy.execute(crypto_input)
except DisallowedOperation as e:
    print(f"Operation blocked: {e.message}")
```

### Phone Number Anonymization Policies

#### 1. AnonymizePhoneNumbersPolicy
Detects and anonymizes phone numbers while preserving format.

```python
from ai_safety_engine.policies.phone_policies import AnonymizePhoneNumbersPolicy
from ai_safety_engine.models import PolicyInput

# Example with phone numbers
phone_input = PolicyInput(input_texts=[
    "Call me at 555-123-4567 or text 0555-987-6543",
    "My office number is +1-800-555-0199"
])

rule_result, action_result, policy_output = AnonymizePhoneNumbersPolicy.execute(phone_input)

print("Original:", phone_input.input_texts)
print("Anonymized:", policy_output.output_texts)
print("Transformation map:", policy_output.transformation_map)

# Example output:
# Original: ["Call me at 555-123-4567 or text 0555-987-6543"]
# Anonymized: ["Call me at 742-891-3056 or text 0742-134-8902"]
```

**Features:**
- Detects various phone number formats
- Preserves number structure and formatting
- Maintains readability while protecting privacy
- Supports international formats
- Provides transformation mapping for reversibility

#### 2. AnonymizePhoneNumbersPolicy_LLM_Finder
Enhanced phone number detection using LLM.

```python
from ai_safety_engine.policies.phone_policies import AnonymizePhoneNumbersPolicy_LLM_Finder

# Can detect phone numbers in natural language
natural_phone = PolicyInput(input_texts=[
    "You can reach me at five five five, one two three, four five six seven",
    "My number is five-five-five-HELP (4357)"
])

rule_result, action_result, policy_output = AnonymizePhoneNumbersPolicy_LLM_Finder.execute(natural_phone)
```

**Features:**
- LLM-powered phone number detection
- Handles written-out numbers
- Detects vanity numbers (1-800-FLOWERS)
- More comprehensive than regex patterns

### Usage Examples with Error Handling

```python
from ai_safety_engine.models import PolicyInput
from ai_safety_engine.exceptions import DisallowedOperation

def safe_content_processing(input_texts, policies):
    """Process content through multiple safety policies"""
    
    policy_input = PolicyInput(input_texts=input_texts)
    results = {}
    
    for policy_name, policy in policies.items():
        try:
            rule_result, action_result, policy_output = policy.execute(policy_input)
            
            results[policy_name] = {
                'success': True,
                'confidence': rule_result.confidence,
                'action_taken': policy_output.action_output.get('action_taken', 'UNKNOWN'),
                'output_texts': policy_output.output_texts,
                'triggered_keywords': rule_result.triggered_keywords
            }
            
            # Update input for next policy (if content was modified)
            if policy_output.output_texts:
                policy_input = PolicyInput(input_texts=policy_output.output_texts)
                
        except DisallowedOperation as e:
            results[policy_name] = {
                'success': False,
                'error': str(e),
                'blocked': True
            }
            break  # Stop processing on block
            
    return results

# Example usage
policies = {
    'crypto_filter': CryptoBlockPolicy,
    'phone_anonymizer': AnonymizePhoneNumbersPolicy
}

test_content = [
    "Contact John at 555-123-4567 about the bitcoin investment opportunity"
]

results = safe_content_processing(test_content, policies)

for policy_name, result in results.items():
    print(f"{policy_name}: {result}")
```

### Policy Configuration Examples

```python
# Custom cryptocurrency policy with specific keywords
from ai_safety_engine.policies.crypto_policies import CryptoRule, CryptoBlockAction
from ai_safety_engine.base import Policy

custom_crypto_policy = Policy(
    name="Custom Crypto Policy",
    description="Blocks specific cryptocurrency mentions",
    rule=CryptoRule(options={
        "keywords": ["dogecoin", "shiba", "meme coin", "pump and dump"]
    }),
    action=CryptoBlockAction(),
    language="en"
)

# Multi-language phone anonymization
multilingual_phone_policy = Policy(
    name="Multilingual Phone Policy",
    description="Anonymizes phone numbers in multiple languages",
    rule=AnonymizePhoneNumberRule(),
    action=AnonymizePhoneNumberAction(),
    language="auto",  # Auto-detect language
    language_identify_model="gpt-4",
    base_model="gpt-3.5-turbo"
)
```

## Quick Start Guide

### Basic Usage

```python
from ai_safety_engine.models import PolicyInput
from ai_safety_engine.policies.crypto_policies import CryptoBlockPolicy

# Create input data
input_data = PolicyInput(input_texts=["I want to invest in bitcoin"])

# Execute policy
rule_result, action_result, policy_output = CryptoBlockPolicy.execute(input_data)

# Check results
print(f"Confidence: {rule_result.confidence}")
print(f"Action: {policy_output.action_output['action_taken']}")
print(f"Message: {policy_output.action_output.get('message', 'No message')}")
```

### Integration Example

```python
from ai_safety_engine.models import PolicyInput
from ai_safety_engine.policies.crypto_policies import CryptoBlockPolicy
from ai_safety_engine.policies.phone_policies import AnonymizePhoneNumbersPolicy
from ai_safety_engine.exceptions import DisallowedOperation

class ContentSafetyService:
    """Service class for content safety processing"""
    
    def __init__(self):
        self.policies = [
            CryptoBlockPolicy,
            AnonymizePhoneNumbersPolicy
        ]
    
    def process_content(self, content_list):
        """Process content through safety policies"""
        policy_input = PolicyInput(input_texts=content_list)
        
        for policy in self.policies:
            try:
                rule_result, action_result, policy_output = policy.execute(policy_input)
                
                # If content was modified, use the modified version for next policy
                if policy_output.output_texts != policy_input.input_texts:
                    policy_input = PolicyInput(input_texts=policy_output.output_texts)
                    
            except DisallowedOperation as e:
                return {
                    'success': False,
                    'error': str(e),
                    'policy': policy.name
                }
        
        return {
            'success': True,
            'processed_content': policy_input.input_texts,
            'original_content': content_list
        }

# Usage
safety_service = ContentSafetyService()

# Test with problematic content
test_content = [
    "Call me at 555-123-4567 to discuss bitcoin investments",
    "My phone is 0555-987-6543 and I'm interested in ethereum"
]

result = safety_service.process_content(test_content)

if result['success']:
    print("Original:", result['original_content'])
    print("Processed:", result['processed_content'])
else:
    print(f"Content blocked by {result['policy']}: {result['error']}")
```

## API Reference

### Core Classes

#### PolicyInput
```python
class PolicyInput:
    input_texts: Optional[List[str]] = None
    input_images: Optional[List[str]] = None  
    input_videos: Optional[List[str]] = None
    input_audio: Optional[List[str]] = None
    input_files: Optional[List[str]] = None
```

#### RuleOutput
```python
class RuleOutput:
    confidence: float                          # 0.0 to 1.0
    content_type: str                         # Type of content detected
    details: str                              # Human-readable description
    triggered_keywords: Optional[List[str]]    # Keywords that triggered the rule
```

#### PolicyOutput
```python
class PolicyOutput:
    output_texts: Optional[List[str]] = None
    output_images: Optional[List[str]] = None
    output_videos: Optional[List[str]] = None  
    output_audio: Optional[List[str]] = None
    output_files: Optional[List[str]] = None
    action_output: Optional[Dict[str, Any]] = None
    transformation_map: Optional[Dict[int, Dict[str, str]]] = None
```

### Base Classes

#### RuleBase
```python
class RuleBase:
    name: str
    description: str  
    language: str = "en"
    
    def process(self, policy_input: PolicyInput) -> RuleOutput:
        # Override this method
        pass
```

#### ActionBase
```python
class ActionBase:
    name: str
    description: str
    language: str = "en"
    
    def action(self, rule_result: RuleOutput) -> PolicyOutput:
        # Override this method
        pass
```

#### Policy
```python
class Policy:
    def __init__(self, name: str, description: str, rule: RuleBase, 
                 action: ActionBase, language: str = "en", **llm_config):
        pass
    
    def execute(self, policy_input: PolicyInput) -> Tuple[RuleOutput, PolicyOutput, PolicyOutput]:
        pass
```

## Testing

Run the test suite to verify installation:

```bash
# Install development dependencies
pip install -e ".[dev]"

# Run all tests
pytest tests/

# Run specific test categories
pytest tests/test_crypto_*.py
pytest tests/test_anonymize_*.py
```

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-policy`)
3. Commit your changes (`git commit -am 'Add new policy'`)
4. Push to the branch (`git push origin feature/new-policy`)
5. Create a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

For support, issues, or feature requests, please visit our [GitHub repository](https://github.com/your-org/ai-safety-engine) or contact the maintainers.

---

**Upsonic AI Safety Engine** - Securing AI applications with intelligent content filtering and policy enforcement.
