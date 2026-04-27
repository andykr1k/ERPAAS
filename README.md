# Efficient Robotics Planning in Atomic Action Space

A framework for enabling embodied agents to interact with humans and the world through natural language, physical clarification, and general tool use.

---

## Overview

Current embodied agents interact with humans in limited and unintuitive ways. Vision-Language-Action (VLA) models attempt to bridge this gap by enabling natural language interaction, but fail to address fundamental limitations:

- Humans frequently **underspecify tasks**, requiring back-and-forth dialog before execution
- VLAs are **not general tool-use models** — they cannot leverage the same "movement" ability to, for example, search the internet or retrieve relevant memories via RAG
- Existing systems miss the unique advantage of embodied agents: using **physicality to interact in more natural ways** (e.g., pointing to disambiguate)

This project addresses all three by combining a **physical intention clarification model**, a **tool embedding space**, and a **robot interop layer** into a unified agentic system.

---

## Architecture

### Components

| Component | Description |
|---|---|
| **Clarification Model** | Fine-tuned LLM that asks clarifying questions and generates disambiguating physical actions |
| **Tool Embedding Space** | Coordinator agent that maps natural language intentions to structured tool calls |
| **Robot Interop Layer** | Async Python functions bridging tool calls to physical robot execution |
| **Simulator** | Simulated arm environment providing RGB + Depth image outputs |

---

## How It Works

### Tool Call Flow

```
Language model: "Point to the green block"
       ↓
Tool Embedding Space: Embed("Point to the green block")
       ↓
Tool Schema:
  {
    "Tool": moveToLocation,
    "Params": {
      "pixel_location": (x, y),
      "offset": (x_meters, y_meters, z_meters)
    }
  }
       ↓
Call: moveToLocation(pixel_location, offset)
       ↓
Returns: { "success": true }
       ↓
Passed back to language model
```

### Clarification Model as a Tool

When a scene is ambiguous (e.g., two identical blocks on a table):

```
Tool Embedding Space: Embed("Clarify which block user wants.")
       ↓
Tool Schema:
  {
    "Tool": clarify,
    "Params": {
      "visual_input": <RGB Image>,
      "to_clarify": "Please pass me the block"
    }
  }
       ↓
Call: clarify(image, "Please pass me the block")
       ↓
Returns:
  {
    "clarifying_question": "Do you mean the block on the left?",
    "movement_intention": "Point to the block on the left"
  }
       ↓
Tool Embedding Space: Embed("Point to the block on the left")
       ↓
Call: moveToLocation((500, 300), (0, 0, 0.1))
       ↓
Returns: { "success": true }
```

The agent simultaneously **asks a question** and **physically points** to resolve ambiguity — combining verbal and physical communication naturally.

---

## Key Features

### Physical Clarification
The clarification model generalizes the concept of "clarifying questions" to include **physical actions**. When asked to "pick up the block" with two identical blocks present, the agent can:
- Ask: *"Do you mean the block on the left?"*
- Simultaneously point its arm at the candidate block to resolve perspective ambiguity between agent and human

### General Tool Use
The tool embedding space is not limited to physical actions. Any tool can be registered — including web search, RAG-based memory retrieval, or the clarification model itself.

### Flexible Movement Primitives
A single low-level tool (e.g., `moveToLocation` with pixel coordinates + 3D offset) can express many high-level intentions:
- Picking up an object
- Pointing to an object
- Placing an object in a bin

---

## Project Structure

---

## Roadmap

- [x] Base clarification model fine-tuned to ask clarifying questions
- [x] Base tool embedding space and coordinator agent
- [ ] Physical clarification dataset (modeled after ClearVQA): tuples of `(ambiguous query, disambiguating question, disambiguating movement intention)`
- [ ] Fine-tune clarification model on physical clarification dataset
- [ ] Add physical action tools to the embedding space
- [ ] Program robot interop layer (async Python)
- [ ] Simulator setup with RGB + Depth image extraction
- [ ] Define evaluation scenarios (e.g., *"Pass me my favorite drink"* with orange juice and grape juice on the table)

---

## Dataset Format

Training data for the clarification model follows this tuple structure (inspired by ClearVQA):

```
(ambiguous_query, disambiguating_question, disambiguating_movement_intention)
```

**Note:** `movement_intention` is generalized — it includes any action that conveys information to the human, such as pointing at an object *or* drawing a bounding box on a screen.

---

## Example Evaluation Scenario

**Setup:** Orange juice and grape juice placed on a table.  
**User command:** *"Pass me my favorite drink."*  
**Expected behavior:** Agent recognizes ambiguity → calls clarify tool → asks which drink + points to each option → executes after confirmation.

---

## Contributing

This project is jointly developed across two teams. Before generating datasets or implementing tools, coordinate on the **canonical tool and intention definitions** — both the clarification model training and the tool embedding space depend on a shared schema.