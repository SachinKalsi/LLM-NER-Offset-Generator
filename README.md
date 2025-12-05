# LLM-NER-Offset-Generator
Generate accurate NER training datasets with `start_offset`, `end_offset` with the help of LLM tool functions.

# 🚀 Overview

**LLM-NER-Offset-Generator** is a lightweight, deterministic toolkit designed to produce **high-precision NER training data**. It automates the extraction of:

✔ start_offset

✔ end_offset

✔ support for multiple entities

✔ contextual information for disambiguation

✔ LLM-friendly tool function for automated annotation workflows

# 🔥 What Problem It Solves?

If you need to extract entities from **millions of documents**, you need your own custom NER system and building an accurate one requires **large-scale, high-quality training data.**
NER datasets must include precise character offsets, which is often the most challenging and error-prone part of dataset creation.

> **LLM-NER-Offset-Generator automates this process, producing high-precision entity offsets at scale.**

# 🔍 What This Notebook Does

This project addresses one of the core challenges in NER dataset creation:

> **How do we reliably generate accurate start/end offsets for entities and do it at scale?**

This notebook provides a deterministic, scalable solution that ensures:

 ✔ exact `start_offset` & `end_offset`
 
 ✔ Correct handling of multiple occurrences of the same entity
 
 ✔ Clear context windows to disambiguate repeated mentions 
 
 ✔ A clean LLM tool function format for agent workflows

 # 🧩 How It Works

 1. LLM receives the prompt which has `input text`, `text-id`, `list of entities` to be identified from text.
 2. The LLM identifies the specific entity values in the text.
 3. The LLM calls the tool `locate_entity_positions_in_text` by passing `text-id` & list of `(entity_value, entity_name)` pairs. (e.g: [[Apple, Organisation]]
 4. The tool performs deterministic matching using the raw text:
 
    5. Finds all occurrences of each entity value
    
    6. Computes `start_offset` and `end_offset`
    
    7. Extracts the surrounding context
    
 8. The tool returns structured offsets back to the LLM.
 9. The LLM produces clean JSON output (by using tool results) with zero hallucination in offsets.

This hybrid approach gives you the best of both worlds: `LLM intelligence` to find entity values + deterministic logic to compute exact offsets.

# Sample example
```
text_id = "1234"
input_text = """Steve Jobs founded Apple in 1976. And he love eating Apple ans so he named the company Apple."""
entities = "ORGANISATION, FRUIT"
```
LLM response
<img width="1256" height="475" alt="image" src="https://github.com/user-attachments/assets/160c25b4-8ae5-43c5-af7c-7ad0814c0303" />

<details>
<summary><strong>Click to view full JSON output</strong></summary>

```json
[
  {
    'role': 'user',
    'content': 'You are a named entity recognition expert who specializes in identifying and extracting specific entities from text with high precision.\n\nHere is the text from which you need to extract entities:\n\n<text>\ntext-id: 1234\nSteve Jobs founded Apple in 1976. And he love eating Apple ans so he named the company Apple.\n</text>\n\nHere is the list of entities you need to extract:\n\n<entities>\nORGANISATION, FRUIT\n</entities>\n\nYour task is to identify the exact values of each entity from the given text and return them in a structured format.\n\nFollow these guidelines:\n\n1. **Exact Matching**: Extract the entity values exactly as they appear in the text, preserving capitalization, spacing, and punctuation.\n\n2. **Offset Calculation**: \n   - start_offset: The character position where the entity begins (0-indexed, counting from the start of the text)\n   - end_offset: The character position where the entity ends (the position after the last character of the entity)\n\n3. **Entity Identification**:\n   - Only extract entities that are explicitly mentioned in the text\n   - If an entity appears multiple times, extract all occurrences\n   - If an entity is not found in the text, do not include it in the output\n\n4. **Output Format**: Return a list of JSON objects, where each object has these keys:\n   - entity_name: The name of the entity type (from the entities list)\n   - entity_value: The exact text extracted from the input\n   - start_offset: Starting character position\n   - end_offset: Ending character position\n\n\nAfter your analysis, provide your final answer as a valid JSON array. Your output should contain ONLY the JSON array with the extracted entities, with no additional explanation or text outside the array.\n'
  },
  {
    'role': 'assistant',
    'content': [
      {
        'type': 'tool_use',
        'id': 'toolu_019W2PdniMGr41eiQge7pwDh',
        'name': 'locate_entity_positions_in_text',
        'input': {
          'text_id': '1234',
          'entity_value_pairs': [
            {
              'label': 'ORGANISATION',
              'value': 'Apple'
            },
            {
              'label': 'FRUIT',
              'value': 'Apple'
            }
          ],
          'context_chars': 20
        }
      }
    ]
  },
  {
    'role': 'user',
    'content': [
      {
        'type': 'tool_result',
        'tool_use_id': 'toolu_019W2PdniMGr41eiQge7pwDh',
        'content': '{"found_contexts": [[{"label": "ORGANISATION", "value": "Apple", "start_offset": 19, "end_offset": 24, "surround_context": "Steve Jobs founded Apple in 1976. And he lov"}, {"label": "ORGANISATION", "value": "Apple", "start_offset": 53, "end_offset": 58, "surround_context": " And he love eating Apple ans so he named the"}, {"label": "ORGANISATION", "value": "Apple", "start_offset": 87, "end_offset": 92, "surround_context": "e named the company Apple."}], [{"label": "FRUIT", "value": "Apple", "start_offset": 19, "end_offset": 24, "surround_context": "Steve Jobs founded Apple in 1976. And he lov"}, {"label": "FRUIT", "value": "Apple", "start_offset": 53, "end_offset": 58, "surround_context": " And he love eating Apple ans so he named the"}, {"label": "FRUIT", "value": "Apple", "start_offset": 87, "end_offset": 92, "surround_context": "e named the company Apple."}]], "not_found": []}'
      }
    ]
  }
]
</details> ```
 
