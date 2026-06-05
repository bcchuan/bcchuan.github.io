# bcchuan.github.io

## How to install Go for Raspberry Pi 4

```
cd $HOME
wget https://dl.google.com/go/go1.26.2.linux-arm64.tar.gz
sudo tar -C /usr/local -xvf go1.26.2.linux-arm64.tar.gz
```
```
vi ~/.bashrc

export GOPATH=$HOME/go
export PATH=/usr/local/go/bin:$PATH:$GOPATH/bin
```
```
source ~/.bashrc
```

## A Go program to print ASCII art

ascii-art.go
```
package main

import (
    "fmt"
)

func main() {

    var stegosaurus = `         \                      .       .
          \                    / ` + "`" + `.   .' "
           \           .---.  <    > <    >  .---.
            \          |    \  \ - ~ ~ - /  /    |
          _____           ..-~             ~-..-~
         |     |   \~~~\\.'                    ` + "`" + `./~~~/
        ---------   \__/                         \__/
       .'  O    \     /               /       \  "
      (_____,    ` + "`" + `._.'               |         }  \/~~~/
       ` + "`" + `----.          /       }     |        /    \__/
             ` + "`" + `-.      |       /      |       /      ` + "`" + `. ,~~|
                 ~-.__|      /_ - ~ ^|      /- _      ` + "`" + `..-'
                      |     /        |     /     ~-.     ` + "`" + `-. _  _  _
                      |_____|        |_____|         ~ - . _ _ _ _ _>
    `


    var cow = `         \  ^__^
          \ (oo)\_______
        (__)\       )\/\
            ||----w |
            ||     ||
        `

    fmt.Println(stegosaurus)
    fmt.Println(cow)
}
```

Run the go program:
```
go run ascii-art.go
```

## Compile-time State Machine in Go

```
// i2c.go
package main

import (
	"fmt"
	"log"
)

// Enums are represented as typed integers in Go (idiomatic approach)
type BusFrequency int

const (
	Standard100kHz BusFrequency = iota
	Fast400kHz
	FastPlus1MHz
)

type AddressMode int

const (
	SevenBit AddressMode = iota
	TenBit
)

type TimingConfig struct {
	SetupTimeNs  uint32
	HoldTimeNs   uint32
	TimeoutMs    uint32
}

type I2cConfig struct {
	BusFrequency          BusFrequency
	AddressMode           AddressMode
	Timing                TimingConfig
	EnableClockStretching bool
	MasterMode            bool
}

// Builder methods return new instances (value semantics) to match Rust's `mut self` pattern
func NewI2cConfig() I2cConfig {
	return I2cConfig{
		BusFrequency:          Standard100kHz,
		AddressMode:           SevenBit,
		Timing:                TimingConfig{SetupTimeNs: 250, HoldTimeNs: 300, TimeoutMs: 1000},
		EnableClockStretching: true,
		MasterMode:            true,
	}
}

func (c I2cConfig) WithFrequency(freq BusFrequency) I2cConfig {
	c.BusFrequency = freq
	return c
}

func (c I2cConfig) WithAddressMode(mode AddressMode) I2cConfig {
	c.AddressMode = mode
	return c
}

func (c I2cConfig) WithTiming(t TimingConfig) I2cConfig {
	c.Timing = t
	return c
}

// State types: Each state is a distinct type to enforce compile-time safety.
// Go doesn't have Rust's `impl<T>` or Scala's type-level generics, so separate types
// are the idiomatic solution for strict state machines.
type I2cControllerConfigured struct { config I2cConfig }
type I2cControllerInitialized struct { config I2cConfig }
type I2cControllerInstalled   struct { config I2cConfig }

func NewI2cController(config I2cConfig) *I2cControllerConfigured {
	fmt.Println("I2C Controller configured")
	return &I2cControllerConfigured{config: config}
}

// Transitions from Configured -> Initialized state
func (c *I2cControllerConfigured) Initialize() (*I2cControllerInitialized, error) {
	fmt.Println("Initializing I2C controller...")
	if c.config.Timing.TimeoutMs == 0 {
		return nil, fmt.Errorf("timeout cannot be zero")
	}
	fmt.Println("Hardware initialized")
	return &I2cControllerInitialized{config: c.config}, nil
}

// Transitions from Initialized -> Installed state
func (c *I2cControllerInitialized) Install() *I2cControllerInstalled {
	fmt.Println("Installing I2C driver...")
	fmt.Println("Driver installed and ready")
	return &I2cControllerInstalled{config: c.config}
}

// Operations only available in Installed state
func (c *I2cControllerInstalled) Read(deviceAddr uint8, buffer []byte) error {
	fmt.Printf("Reading %d bytes from device 0x%02X\n", len(buffer), deviceAddr)
	for i := range buffer {
		// Mimics Rust's wrapping_add for u8
		buffer[i] = uint8(uint16(deviceAddr) + uint16(i))
	}
	return nil
}

func (c *I2cControllerInstalled) Write(deviceAddr uint8, data []byte) error {
	fmt.Printf("Writing %d bytes to device 0x%02X: ", len(data), deviceAddr)
	for _, b := range data {
		fmt.Printf("%02X", b)
	}
	fmt.Println()
	return nil
}

func (c *I2cControllerInstalled) GetConfig() *I2cConfig {
	return &c.config
}

func main() {
	fmt.Println("I2C Driver Compile-Time State Machine Demo")
	fmt.Println("==========================================")

	// Correct usage: Config -> Initialize -> Install -> Operations
	fmt.Println("\n1. Correct usage:")
	config := NewI2cConfig().WithFrequency(Fast400kHz).WithAddressMode(SevenBit)
	controller := NewI2cController(config)

	initialized, err := controller.Initialize()
	if err != nil {
		log.Fatalf("Failed to initialize: %v", err)
	}

	installed := initialized.Install()

	buffer := make([]byte, 4)
	if err := installed.Read(0x48, buffer); err != nil {
		log.Fatal(err)
	}
	if err := installed.Write(0x50, []byte{0xAA, 0xBB}); err != nil {
		log.Fatal(err)
	}

	// Error case: Invalid configuration
	fmt.Println("\n2. Error case - invalid config:")
	badConfig := I2cConfig{
		BusFrequency:          Standard100kHz,
		AddressMode:           SevenBit,
		Timing:                TimingConfig{SetupTimeNs: 200, HoldTimeNs: 250, TimeoutMs: 0}, // Invalid!
		EnableClockStretching: true,
		MasterMode:            true,
	}

	badController := NewI2cController(badConfig)
	if _, err := badController.Initialize(); err != nil {
		fmt.Printf("Expected error: %v\n", err)
	} else {
		fmt.Println("Unexpected success")
	}

	// The following would cause COMPILE-TIME ERRORS (uncomment to see):
	/*
		configured := NewI2cController(NewI2cConfig())
		// configured.Read(0x48, buffer)  // ERROR: *I2cControllerConfigured does not have Read method
		initialized2, _ := configured.Initialize()
		// initialized2.Write(0x50, []byte{0xAA}) // ERROR: *I2cControllerInitialized does not have Write method
	*/

	fmt.Println("\nDemo completed - state machine enforces correct sequence!")
}
```

Running the Go program produces the following output:

```
I2C Driver Compile-Time State Machine Demo
==========================================

1. Correct usage:
I2C Controller configured
Initializing I2C controller...
Hardware initialized
Installing I2C driver...
Driver installed and ready
Reading 4 bytes from device 0x48
Writing 2 bytes to device 0x50: AABB

2. Error case - invalid config:
I2C Controller configured
Initializing I2C controller...
Expected error: timeout cannot be zero

Demo completed - state machine enforces correct sequence!
```

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
