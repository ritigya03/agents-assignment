# Intelligent Interruption Handler - Assignment Solution

## The Problem

When a user says "yeah", "okay", or "hmm" while the AI agent is talking, the agent stops speaking. This is annoying because these words just mean "I'm listening" - they're not real interruptions.

**Example:**
- Agent: "Let me explain blockchain. It's a distributed ledger that..."
- User: "yeah" *(just listening)*
- Agent: ❌ **STOPS TALKING** ← This is the problem!

## My Solution

I created a smart filter that knows the difference between:
- 🟢 **Soft words** = "yeah", "okay", "hmm" (just listening sounds)
- 🔴 **Interrupt words** = "stop", "wait", "no" (real commands)

### How It Works (Simple Diagram)

```
User says something
        ↓
   Is agent talking?
        ↓
    ┌───┴───┐
   YES      NO
    ↓        ↓
Is it a     Process
soft word?  normally
    ↓
 ┌──┴──┐
YES    NO
 ↓      ↓
IGNORE  Is it "stop"?
Agent      ↓
keeps   ┌──┴──┐
talking YES   NO
        ↓     ↓
      STOP  STOP
      agent agent
```

## The Logic (4 Simple Rules)

| What User Says | Agent Status | What Happens |
|----------------|--------------|--------------|
| "yeah", "okay" | 🗣️ **Speaking** | ✅ **IGNORE** - Agent keeps talking |
| "stop", "wait" | 🗣️ **Speaking** | 🛑 **STOP** - Agent stops immediately |
| "yeah", "okay" | 🤐 **Silent** | 💬 **RESPOND** - Agent replies normally |
| "hello" | 🤐 **Silent** | 💬 **RESPOND** - Normal conversation |

## Why My Solution is Good

### 1. **Three-Layer Protection**
- **Layer 1**: VAD filter (blocks very short sounds)
- **Layer 2**: My smart word detector (checks if it's a soft word)
- **Layer 3**: Auto-resume (fixes false stops)

### 2. **No Pause or Stutter**
When user says "okay" while agent talks:
1. My code detects it's a soft word
2. Calls `session.resume()` instantly
3. Agent continues smoothly - **no pause!**

### 3. **Easy to Customize**
Just edit the `.env` file:
```bash
# Add your own soft words
SOFT_INTERRUPT_WORDS="yeah,okay,hmm,right,cool"

# Add your own stop words
INTERRUPT_KEYWORDS="wait,stop,pause"
```

### 4. **Handles Tricky Cases**
- "Okay." with punctuation → ✅ Still ignored
- "yeah but wait" → ✅ Stops (detects "wait")
- Multiple "yeah yeah yeah" → ✅ All ignored

## How to Run

1. **Install**:
   ```bash
   uv sync
   ```

2. **Setup** (copy `.env.example` to `.env` and add your API keys):
   ```bash
   LIVEKIT_URL="your-url"
   LIVEKIT_API_KEY="your-key"
   LIVEKIT_API_SECRET="your-secret"
   OPENAI_API_KEY="your-key"
   DEEPGRAM_API_KEY="your-key"
   CARTESIA_API_KEY="your-key"
   ```

3. **Run**:
   ```bash
   uv run --no-sync examples/voice_agents/interrupt_handler_agent.py dev
   ```

4. **Test**:
   - Connect via [Agents Playground](https://agents-playground.livekit.io/)
   - Ask: "Explain something long"
   - Say "okay" while agent talks → Agent continues!
   - Say "stop" → Agent stops!

## Test Scenarios

### ✅ Test 1: Long Explanation
- Agent talks for 30 seconds
- User says "yeah", "okay", "hmm"
- **Result**: Agent never stops

### ✅ Test 2: Silent Response
- Agent asks "Ready?"
- User says "yeah"
- **Result**: Agent replies "Great, let's go!"

### ✅ Test 3: Real Interrupt
- Agent is talking
- User says "stop"
- **Result**: Agent stops immediately

### ✅ Test 4: Mixed Input
- Agent is talking
- User says "yeah but wait"
- **Result**: Agent stops (detected "wait")

## Technical Details

### Key Code Parts

1. **State Tracking**:
   ```python
   agent_speaking = True/False  # Is agent talking?
   was_vad_interrupted = True/False  # Did VAD stop the agent?
   ```

2. **Word Detection**:
   ```python
   is_only_soft_words("okay")  # Returns True
   contains_interrupt_keyword("stop")  # Returns True
   ```

3. **Smart Resume**:
   ```python
   if soft_word and was_vad_interrupted:
       session.resume()  # Continue talking!
   ```

### Files Changed
- `interrupt_handler_agent.py` - Main logic
- `.env` - Configuration
- `ASSIGNMENT_README.md` - This file

## Why This Meets Requirements

| Requirement | My Solution |
|-------------|-------------|
| ✅ Ignore "yeah" while speaking | Uses `is_only_soft_words()` + `session.resume()` |
| ✅ Stop on "wait" while speaking | Uses `contains_interrupt_keyword()` + `session.interrupt()` |
| ✅ Respond to "yeah" when silent | Checks `agent_speaking == False` |
| ✅ No pause/stutter | Instant resume, no delays |
| ✅ Configurable words | Environment variables in `.env` |
| ✅ State-aware | Tracks `agent_speaking` state |
| ✅ Handles mixed input | Checks for interrupt keywords in any position |

## Summary

**Problem**: Agent stops on "yeah" even when just listening sounds.

**Solution**: Smart 3-layer filter that knows when agent is talking and ignores soft words, but stops on real commands.

**Result**: Natural conversation where agent doesn't stop for "yeah" but does stop for "stop".

---

**Author**: Ritigya Gupta  
**Branch**: `feature/interrupt-handler-ritigya`  
**Repo**: https://github.com/Dark-Sys-Jenkins/agents-assignment
**Video**: https://drive.google.com/file/d/1lRWFzSwuO0l-Y_neWqJvWRTaxjmpdoLl/view?usp=sharing