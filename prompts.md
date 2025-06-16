## General Prompt Template
```
You are [ role or expert type ].  
Your task is to [ objective ].  
Input: [your data]  
Constraints: [ format, length, style, etc. ]  
Output should be: [ bullets, table, JSON, etc. ]  
```

## Song Emotion Analysis Prompt Template
```
You are a music psychologist and lyrical analyst.  
Your task is to articulate why a particular song evokes a specific emotional response in me by analyzing  its musical elements, lyrical content, and personal associations.  
Input:  
- Song: [ song title and artist ]  
- Emotion: [ description of the feeling it evokes-e.g., nostalgic, empowered, melancholic, etc. ]  
- Optional context: [ any personal memories, settings, or associations linked to the song ]  
- Constraints:  
 - Length: 3-5 short paragraphs  
 - Style: Reflective and emotionally intelligent, blending analytical insight with relatable language.  
- Output should be:  
 - A brief narrative explanation of why the song might cause this emotional response.  
 - References to specific musical and lyrical elements.  
 - A short bullet list summarizing key triggers for the emotion.  
```

## Capture Styling of an Image 1
```
Write me a paragraph description of the artistic styling of this image. Don't include anything that is in the picture. Only the styling that is used.
```

## Capture Styling of an Image 2
```
You are a professional prompt engineer specializing in image generation.  
Your task is to analyze a reference image and extract its visual styling into a structured template that can be reused to generate new images in the same aesthetic.  
Be descriptive, precise, and avoid referencing specific objects or content from the original image — focus only on the style.

---

Image Style Description Framework

- Medium & Era Reference:  
  What traditional or digital medium does the style resemble? Any particular art era (e.g., 1950s gouache, 1990s animation, etc.)?

- Color Palette & Texture:  
  Describe the dominant tones, saturation level, warmth/coolness, and whether the texture is smooth, grainy, or tactile.

- Mood & Emotional Effect:  
  What atmosphere does the style evoke? (e.g., nostalgic, surreal, whimsical, serene)

- Structure & Composition:  
  Comment on framing, balance, perspective, and visual hierarchy. Is it cluttered or minimalist? Geometric or organic?

- Comparative Touchstones:  
  Name any known illustrators, painters, or animation studios whose work this resembles.

- Brush & Texture Details (if applicable):  
  Describe visible brushstrokes, canvas/paper texture, or any digital replication techniques.

- Character vs. Background Styling:  
  Note differences between character design and background. Are characters more stylized or expressive? Does lighting or detail favor one over the other?
```

## Example Styles
### Soft Chronicle
Both visual softness and a storytelling mood.
```
A traditional painterly technique with a soft impressionistic touch, evoking the feel of mid-20th century gouache or acrylic illustrations. Brushwork is visible but controlled, lending a tactile texture to surfaces without becoming overly abstract. The color palette is muted yet warm, with a slight sepia or sun-washed filter that gives the piece a nostalgic, timeless quality. Lighting is naturalistic but diffused, emphasizing atmosphere over high contrast. Line work is subtly implied rather than overtly drawn, and the figures and forms maintain a stylized simplicity that borders on the cartoonish, while still respecting proportion and volume. The overall composition is balanced and narrative-driven, inviting emotional resonance through its softness and clarity.
```

### Symmetric Minimal Futurism
Balanced structure meets editorial clarity in a tech-forward world.
```
A clean, digital illustration style influenced by mid-century modern design and contemporary editorial graphics. The composition is highly symmetrical, with characters and objects mirrored across a central axis to emphasize duality and contrast. Flat color fills and soft gradients replace texture and brushwork, lending a crisp, vector-like appearance that feels both polished and deliberate. The muted color palette leans warm and neutral, favoring subtle harmony over bold contrast. Lighting is soft and evenly distributed, with minimal shadowing used to suggest depth without realism. Characters are rendered in a stylized, geometric manner, more detailed than the understated background but still simplified and approachable. The overall tone is contemplative and quiet, evoking a gentle reflection on human-technology coexistence through a minimalist, structured lens.
```

### Storybook Realism
Evokes the feeling of an interesting and fun child-like story that blends realism and stylized figures.
```
Storybook Realism combines a softly textured, semi-realistic painting style for the background with cartoon-style characters in the foreground. The backgrounds should feature naturalistic brushwork—evocative of traditional landscape paintings—with soft lighting, subtle gradients, and atmospheric depth.

Characters are rendered with simple, rounded, and expressive features, using minimal linework. Their color shading is smooth and mostly flat, with subtle use of light and shadow to evoke real-world lighting and give a sense of dimensionality. This soft shading creates a lively, animated feel—like a still from a moving picture—while keeping the character style visually distinct from the more textured, painterly background. The eyes are large with rounded white sclera and solid black pupils, designed without irises, eyelids, lashes, or extra detail lines, giving the face an open and approachable appearance.

The overall effect should feel like a modern picture book illustration: grounded yet playful, serene yet full of life.
``` 

# ADHD-Friendly Prompt Library

A collection of AI prompts designed to help externalize executive function, reduce overwhelm, and support the development of a coherent narrative self.

---

### Get Started When You're Stuck
Use when inertia is high and task initiation feels impossible.
```
Give me a 2-minute version of this task that builds momentum.
```
```
What’s a warm-up task that helps my brain ease into ...
```

### Make a Plan Without Overwhelm
Use when you’re spiraling with too many to-dos and no structure.
```
I have these 5 things to do. Help me prioritize them based on urgency and energy.
```
```
Make me a plan with 3 focus blocks, each 25 minutes, and breaks in between.
```
```
Break this big task into 3 small chunks I can do today, even with low motivation.
```

### Manage Working Memory and Keep Your Train of Thought
Use to stay on track and not lose ideas mid-task.
```
Summarize what I just told you so I can remember it better.
```

### Reduce Decision Fatigue
Use when everything feels equally urgent or unimportant.
```
I’m overwhelmed by choices. What should I focus on if I only have 30 minutes?
```
```
Give me 3 categories for sorting this messy to-do list.
```
```
Help me pick one thing to do right now based on low energy and high impact.
```

### Reflect and Reinforce the Narrative Self
Use to connect your actions with your values and identity.
```
What did I do this week that aligned with the kind of person I want to be?
```
```
Summarize my strengths based on the last 3 things I accomplished.
```
```
Can you help me write a short story of who I am when I’m at my best? At my worst?
```

### Build an External Brain (Persistent Context Prompts)
Use to make AI your prosthetic prefrontal cortex.
```
You are my executive function assistant. Your job is to help me stay focused, prioritize, and reflect. I’ll talk to you throughout the day.
```
```
Each morning, help me review what’s most important to me and how I want to show up.
```

### Daily Check-In Prompt Template
Use this as a consistent daily journaling structure.
**Daily Check-In**
```
What’s one thing I’m proud of from yesterday?  
What’s one priority for today?  
What’s one way I can be kind to myself?  
What’s one distraction I want to minimize?  
What’s one thing that will make me feel accomplished by bedtime?  
```
# Coding
Pillars for evaluating a codebase
1. Performance 
  - Does the code perform well under expected load? Are there obvious performance bottlenecks?
  - Runtime
  - Memory 
  - Resource usage
2. Security
  - Are inputs validated and outputs sanitized?
  - Are secrets stored safely?
  - Are external dependencies up to date?
3. Modularity
  - Can modules be used in other contexts?
  - Are dependencies between modules **minimal and intentional**?
  - Would a new teammate understand the boundaries between components?
4. Functionality
  - Does the code do what it's supposed to?
  - Are edge cases and error states handled well?
  - Are there unexpected side effects or silent falilures?
5. Testing
  - What types of tests are present? (Unit, integration, end-to-end?)
  - Are the tests fast, reliable, and easy to understand?
  - Does the test coverage match the most critical paths in the code?
6. Readability
  - Would someone new to the repo understand what this function does?
  - Are names (functions, variables, files) **descriptive without being verbose**? ⭐
  - Is there **consistent formatting** and commenting where needed?

```
CODEBASE EVALUATION RUBRIC
1. Efficiency
  - Guiding questions:
    - Does the code perform well under expected load?
    - Are there obvious performance bottlenecks?
  - Scoring (1-4):
    1. Slow or wasteful; no profiling or optimization
    2. Some slow areas; inefficient patterns used
    3. Generally performant; optimized for common use
    4. Fast and scalable; known hot paths are optimized
2. Security
  - Guiding questions:
    - Are inputs validated?
    - Are secrets protected?
    - Are common vulnerabilities accounted for?
  - Scoring (1-4):
    1. No security considerations; vulnerable code present
    2. Some secure practices, but gaps exist
    3. Follows basic security hygiene; minimal risk
    4. Secure by design; proactively prevents misuse or attack
3. Modularity
  - Guiding questions:
    - Are responsibilities well-separated?
    - Can code be reused or maintained easily?
  - Scoring (1-4):
    1. Large, monolithic chunks; tight coupling
    2. Some separation, but code reuse is hard
    3. Clear modules with good boundaries
    4. High cohesion and loose coupling; excellent reuse potential
4. Functionality
  - Guiding questions:
    - Does the code meet its requirements?
    - Are edge cases handled?
  - Scoring (1-4):
    1. Fails in core functionality or obvious edge cases
    2. Works for main cases; some bugs or gaps
    3. Works reliably with decent coverage
    4. Robust and comprehensive; handles edge cases gracefully
5. Testing
  - Guiding questions:
    - Are there meaningful, reliable tests?
    - What kinds of tests are present?
  - Scoring (1-4):
    1. Few or no tests; hard to change code safely
    2. Basic tests exist; some are flaky or outdated
    3. Good coverage with unit and integration tests
    4. Fast, reliable, comprehensive suit with coverage for critical paths
6. Readability
  - Guiding questions:
    - Is the code easy to understand?
    - Are names, structure, and formatting clear?
  - Scoring:
    1. Confusing structure; poor naming and formatting
    2. Understandable with effort; inconsistent style
    3. Clear, consistent, and well-structured
    4. Self-explanatory; minimal need for comments; welcoming to new devs
  
SCORING TEMPLATE
  - Efficiency: [1-4]
  - Security: [1-4]
  - Modularity: [1-4]
  - Functionality: [1-4]
  - Testing: [1-4]
  - Readability: [1-4]
  Total Score: [Sum of above, max = 24]  

INTERPRETATION GUIDE
  - 20-24: Excellent — high-quality and production-ready.
  - 15-19: Solid — good structure with some improvement areas.
  - 10-14: Needs work — risk of tech debt or fragility.
  - <10: Warning — revisit before scaling or extending.
```

