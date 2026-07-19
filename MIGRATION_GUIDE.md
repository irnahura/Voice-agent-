# Migration Guide: Bolna to VoiceAI

## Quick Reference

### Package Name Change
| Old | New |
|-----|-----|
| `bolna` | `voiceai` |

### Import Statements
```python
# OLD
from bolna.assistant import Assistant
from bolna.models import Transcriber, Synthesizer
from bolna.exceptions import BolnaComponentError

# NEW
from voiceai.assistant import Assistant
from voiceai.models import Transcriber, Synthesizer
from voiceai.exceptions import VoiceAIComponentError
```

### Environment Variables
```bash
# OLD
BOLNA_WS_SERVER_URL=ws://localhost:5001
BOLNA_API_KEY=your_key_here

# NEW
VOICEAI_WS_SERVER_URL=ws://localhost:5001
VOICEAI_API_KEY=your_key_here
```

### Docker Service Names
```bash
# OLD
docker compose up -d bolna-app twilio-app

# NEW
docker compose up -d voiceai-app twilio-app
```

### Exception Handling
```python
# OLD
try:
    # your code
except BolnaComponentError as e:
    print(f"Error: {e}")

# NEW
try:
    # your code
except VoiceAIComponentError as e:
    print(f"Error: {e}")
```

## Step-by-Step Migration

### 1. Update Your Code
Run a find-and-replace in your project:
- Find: `from bolna`
- Replace: `from voiceai`

- Find: `import bolna`
- Replace: `import voiceai`

- Find: `BolnaComponentError`
- Replace: `VoiceAIComponentError`

### 2. Update Environment Variables
In your `.env` file:
```bash
# Update these variable names
BOLNA_WS_SERVER_URL → VOICEAI_WS_SERVER_URL
BOLNA_API_KEY → VOICEAI_API_KEY
```

### 3. Update Docker Configuration
If you have a custom docker-compose file:
```yaml
# OLD
services:
  bolna-app:
    image: bolna-app:latest

# NEW
services:
  voiceai-app:
    image: voiceai-app:latest
```

### 4. Reinstall the Package
```bash
# Uninstall old package
pip uninstall bolna

# Install new package
pip install voiceai
# OR if installing from source
pip install -e .
```

### 5. Test Your Application
After migration, test all functionality:
- Voice agent creation
- Telephony integration
- LLM interactions
- Exception handling

## Common Issues

### Import Errors
**Problem**: `ModuleNotFoundError: No module named 'bolna'`

**Solution**: 
```bash
pip uninstall bolna
pip install voiceai
```

### Docker Service Not Found
**Problem**: `ERROR: No such service: bolna-app`

**Solution**: Update docker-compose.yml and use `voiceai-app` instead

### Environment Variable Not Working
**Problem**: Configuration not loading

**Solution**: Check that you've renamed `BOLNA_*` to `VOICEAI_*` in your `.env` file

## API Compatibility

All APIs remain the same, only the package name and imports have changed:

```python
# Both old and new have the same API
assistant = Assistant(name="demo_agent")
assistant.add_task(
    task_type="conversation",
    llm_agent=llm_agent,
    transcriber=transcriber,
    synthesizer=synthesizer
)
```

## Need Help?

If you encounter issues during migration:
1. Check this migration guide
2. Review the REBRANDING_SUMMARY.md for detailed changes
3. Check the examples/ directory for updated code samples
4. Open an issue on the project repository

## Validation Checklist

- [ ] All `from bolna` imports updated to `from voiceai`
- [ ] Environment variables renamed
- [ ] Docker configurations updated
- [ ] Exception handling updated
- [ ] Package reinstalled
- [ ] Tests passing
- [ ] Application running successfully
