---
title: CS143 Compiler: Basic concepts
date: 2023-05-27 16:12:38
tags: compiler
---

### 1. Lexical Analysis

> This term comes from "lexicon". In linguistics, a lexicon refers to the vocabulary of a language, including words and phrases. 

In a compiler, the task of lexical analysis is to break down the source code into a series of "tokens", which can be considered the vocabulary of the programming language. 

#### Why we don't call this phase cutting or chopping?
This process is not simply "cutting", as it involves identifying and classifying various elements in the source code, such as keywords, identifiers, operators, etc.

### 2. Parsing
> This term comes from the linguistic term "parse", which means to analyze the grammatical structure of a sentence. 

In a compiler, the task of parsing is to construct the grammatical structure of the source code based on a series of tokens, usually represented as a parse tree or abstract syntax tree. 

#### Why we don't call it assembling?
This process is not simply "assembling", as it involves determining the relationships between tokens based on the grammatical rules of the programming language.

### 3. Semantic Analysis

After understanding the sentence structure, the next step is to try to understand the meaning of what has been written.
This is hard and compilers can only do very limited kinds of semantic analysis. They generally try to catch inconsistencies in the program.

> Jack said Jerry left his assignment at home

Do you know whose assignment Jack was talking about?
Yes, in English or any other language, we experience ambiguity. But programming languages do not have that much tolerance for ambiguity. So we need to do semantic analysis to ensure that the meaning of each statement in the program is clear and unambiguous. 

#### Example: Type check
This involves checking for type mismatches, ensuring that variables are declared before they are used, checking that functions are called with the correct number of arguments, and so on.

> int x = "hello";

This is a type mismatch and the compiler will throw an error. The compiler, during semantic analysis, will catch this inconsistency, because an integer variable cannot be assigned a string value.

### 4. Optimization

The fourth compiler phase, optimization, is a bit like editing. The goal is to modify the program so that it uses less of some resource, such as time, space, power, etc.

For example, the text uses the phrase "but a little bit like ending" and its shorter version "but akin to editing" as an example of optimization in English. 

In the context of a compiler, an example rule that says `X = Y * 0`, is the same as `X = 0` is used to illustrate the kind of optimizations a compiler might do.

### 5. Code Generation

The last compiler phase is code generation, often referred to as Code Gen. It involves translating the high-level program into some other language, such as assembly code.

Using human languages as an analogy, a human translator might translate English into French, a compiler will translate a high-level program into assembly code.
