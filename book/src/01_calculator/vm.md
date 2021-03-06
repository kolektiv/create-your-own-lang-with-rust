## Virtual Machine

Recall from the [crash course](../crash_course.md#virtual-machine-vm) that a (process) VM abstracts away hardware specific instructions so that its Bytecodes (abstract instructions) can be executed in any environment that has the Bytecode runtime support. So to create our VM and its runtime, we need to define our

* Opcodes (new encoding atoms)
* Bytecode representation and
* **Runtime model**  as [Stack Machine](../crash_course.md#stack-machine)

### Opcode

Since our expression based `Calc` language is made up of

* *constant* integers
* *unary* (plus, minus sign) operators and
* *binary* (addition, subtraction) operators

we can define our new opcode encodings like

```rust,ignore
{{#include ../../../calculator/src/compiler/vm/opcode.rs:vm_opcode}}
```
<span class="filename">Filename: calculator/src/compiler/vm/opcode.rs</span>

We choose the simplest form of encoding i.e. encoding the ops as bytes `u8` (in hex format). That is,

```rust,ignore
{{#include ../../../calculator/src/compiler/vm/opcode.rs:vm_make_op}}
```

For easy of access, we store constant `Node::Int(i32)` nodes in a separate memory and `OpConstant(arg)` tracks these values.
In a unary expression like `"1"`, we encode `Node::Int(1)` as the opcode `[1, 0, 0]` as the first constant. (**0x01** in decimal is **1**).

### Bytecode

So we define

```rust,ignore
{{#include ../../../calculator/src/compiler/vm/bytecode.rs:bytecode}}
```
<span class="filename">Filename: calculator/src/compiler/vm/bytecode.rs</span>

and as an example, `"1 + 2"` AST to Bytecode pictorially is

<p align="center">
</br>
    <a href><img alt="ast bytecode" src="../img/ast_bytecode.svg"> </a>
</p>

and in Rust

```rust, ignore
ByteCode {
    instructions: [1, 0, 0, 1, 0, 1, 3, 2],
    constants: [Int(1), Int(2)]
}
```

Now, we can implement our Bytecode interpreter
```rust,ignore
{{#include ../../../calculator/src/compiler/vm/bytecode.rs:bytecode_interpreter}}
```
<span class="filename">Filename: calculator/src/compiler/vm/bytecode.rs</span>

### Runtime

From previous example, our interpreter goes through the bytecode instructions and executes them.

Continuing our `"1 + 2"` Bytecode example,

```text
instructions: [1, 0, 0, 1, 0, 1,   3,   2],
              -------- --------    -    -
                 |         |       |    |
constants: [  Int(1),   Int(2)]  OpAdd  OpPop

[1, 0, 0] points to the first element in constants table i.e. Int(1)
[1, 0, 1] points to Int(2)
[3] (or [0x03]) corresponding to the Opcode *OpAdd*, performs the addition operation Int(1 + 2)
[2] (or [0x02]) corresponding to the Opcode *OpPop* pops out the computed Bytecodes
```

and since we want to model our runtime as a Stack Machine so we define our VM as struct with Bytecode, stack memory (in Stack Machine) and a stack pointer to the next free space

```rust,ignore
{{#include ../../../calculator/src/compiler/vm/vm.rs:vm}}
```

and with the help of *instruction pointer (IP)*, we execute the Bytecodes as follows

```rust,ignore
{{#include ../../../calculator/src/compiler/vm/vm.rs:vm_interpreter}}
```
<span class="filename">Filename: calculator/src/compiler/vm/vm.rs</span>
