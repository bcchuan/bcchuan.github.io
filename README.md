# bcchuan.github.io

## Setup a Data Science Project with uv run jupyter lab

```bash
uv init weather_analysis
cd weather_analysis
uv add pandas matplotlib
uv add --dev jupyter
uv run jupyter lab
```

## Introduction to TypeScript

### Variable Declarations
```typescript
let name: string = "John";
let age: number = 30;
let isActive: boolean = true;

let hobbies = ["coding", "biking"]; // seen as an array of strings
```

### Functions
```typescript
function getName(id: number): string {
  //...
  return name; 
}

const log = (message: string): void => {
  console.log(message);
}
```

### Classes
```typescript
class Person {

  // Fields
  name: string;

  // Constructor 
  constructor(name: string) {
    this.name = name;
  }
  
  // Methods
  greet() {
    console.log(`Hello, ${this.name}!")
  }

}

const person = new Person("Maria");
person.greet();
```

### Generics
```typescript
function identity<T>(arg: T): T {
  return arg; 
}

let output = identity<string>("myString");


class Cache<T> {
  store: T;
  constructor(value: T) {
    this.store = value;
  }
}

let stringCache = new Cache<string>("cached string");
```

### Enumerations
```typescript
enum Direction {
  Up = 1, 
  Down = 2,
  Left = 3,
  Right = 4  
}

let current = Direction.Up;
```

### Decorators
```typescript
function Log(target: any, propertyKey: string) {
  let value = target[propertyKey];

  const getter = () => {
    console.log(`Getting ${propertyKey}`);
    return value;
  };

  const setter = (newVal) => {
    console.log(`Setting ${propertyKey}`);
    value = newVal;
  }

  Object.defineProperty(target, propertyKey, {
    get: getter,
    set: setter
  });
}

class Person {
  @Log
  name: string;
  
  constructor(name: string) {
    this.name = name;
  }
}

let person = new Person("John");
person.name = "Sara"; 
```

## Algebraic Data Type (ADT) in TypeScript
```typescript
interface Circle {
  kind: "circle"; // Discriminant
  radius: number;
}

interface Square {
  kind: "square"; // Discriminant
  sideLength: number;
}

type Shape = Circle | Square; // The ADT (Sum of Products)

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2; // TypeScript knows this is a Circle
    case "square":
      return shape.sideLength ** 2; // TypeScript knows this is a Square
  }
}

```

## Prompt-Based Reasoning 
Instruct the model in your prompt to show its steps: "Analyze the following query, provide your reasoning, and then give the final answer: [Your Question]".

## Cosine Similarity in Python
```python
import math

def dot_product(a, b):
    return sum(ai * bi for ai, bi in zip(a, b))

def magnitude(v):
    return math.sqrt(sum(vi ** 2 for vi in v))

def cosine_similarity(a, b):
    return dot_product(a, b) / (magnitude(a) * magnitude(b))

>>> cosine_similarity([1, 2, 3], [1, 2, 3])
1.0
>>> cosine_similarity([1, 2, 3], [-1, -2, -3])
-1.0
>>> cosine_similarity([1, 0, 0], [0, 1, 0])
0.0
```

## DecoupledIO in Python

```python
import pyrtl


class DecoupledIO:
    """Groups valid/ready/bits signals for a ready-valid handshake interface.

    flipped=False (default): producer side — valid and bits are Outputs, ready is Input.
    flipped=True: consumer side — valid and bits are Inputs, ready is Output.
    Mirrors Chisel's DecoupledIO and Flipped(DecoupledIO(...)).
    """

    def __init__(self, name: str, bitwidth: int, flipped: bool = False):
        if flipped:
            self.valid = pyrtl.Input(1, f'{name}_valid')
            self.bits = pyrtl.Input(bitwidth, f'{name}_bits')
            self.ready = pyrtl.Output(1, f'{name}_ready')
        else:
            self.valid = pyrtl.Output(1, f'{name}_valid')
            self.bits = pyrtl.Output(bitwidth, f'{name}_bits')
            self.ready = pyrtl.Input(1, f'{name}_ready')

```
```python
import pyrtl

from mypyrtl import DecoupledIO


in_io = DecoupledIO('in', 8, flipped=True)
out_io = DecoupledIO('out', 8)

# Input: in_io.valid
# Input: in_io.bits  
# Output: in_io.ready

# Output: out_io.valid
# Output: out_io.bits
# Input: out_io.ready

```
