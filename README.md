# bcchuan.github.io

## Prompt-Based Reasoning: 
Instruct the model in your prompt to show its steps: "Analyze the following query, provide your reasoning, and then give the final answer: [Your Question]".

## Cosine Similarity


```python
import math

def dot_product(a, b):
    return sum(ai * bi for ai, bi in zip(a, b))

def magnitude(v):
    return math.sqrt(sum(vi ** 2 for vi in v))

def cosine_similarity(a, b):
    return dot_product(a, b) / (magnitude(a) * magnitude(b))
```

>>> cosine_similarity([1, 2, 3], [1, 2, 3])
1.0
>>> cosine_similarity([1, 2, 3], [-1, -2, -3])
-1.0
>>> cosine_similarity([1, 0, 0], [0, 1, 0])
0.0
