# 📘 Section 4: Design Patterns (Core of LLD Interviews)

> **Must know intent, structure, and usage for each pattern**

Design patterns are reusable solutions to common software design problems. They're the vocabulary of experienced developers and essential for LLD interviews.

---

## 📑 Table of Contents

1. [Overview](#overview)
2. [Creational Patterns](./creational/README.md)
3. [Structural Patterns](./structural/README.md)
4. [Behavioral Patterns](./behavioral/README.md)
5. [Pattern Selection Guide](#pattern-selection-guide)

---

## Overview

### What Are Design Patterns?

Design patterns are **proven solutions** to recurring design problems. They provide:
- Common vocabulary for developers
- Best practices distilled from experience
- Templates for solving common problems

### Categories

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                            DESIGN PATTERN CATEGORIES                                │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   CREATIONAL                 STRUCTURAL                  BEHAVIORAL                 │
│   (Object Creation)          (Object Composition)        (Object Interaction)       │
│   ─────────────────          ──────────────────          ──────────────────         │
│   • Singleton                • Adapter                   • Strategy                 │
│   • Factory                  • Decorator                 • Observer                 │
│   • Abstract Factory         • Facade                    • Command                  │
│   • Builder                  • Composite                 • State                    │
│   • Prototype                • Proxy                     • Template Method          │
│                                                          • Chain of Responsibility  │
│                                                          • Iterator                 │
│                                                                                     │
│   WHEN TO USE:               WHEN TO USE:                WHEN TO USE:               │
│   Complex object creation    Assembling objects          Communication between      │
│   Control instantiation      Wrapping/adapting           objects                    │
│   Hide creation logic        Simplifying interfaces      Distributing behavior      │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Interview Priority

| Priority | Pattern | Must Know |
|----------|---------|-----------|
| ⭐⭐⭐ | Singleton | Thread-safe implementation |
| ⭐⭐⭐ | Factory | When to use vs Abstract Factory |
| ⭐⭐⭐ | Strategy | Classic example: payment, sorting |
| ⭐⭐⭐ | Observer | Publisher-subscriber model |
| ⭐⭐⭐ | Decorator | Dynamic behavior addition |
| ⭐⭐⭐ | State | State machine implementation |
| ⭐⭐ | Builder | Complex object construction |
| ⭐⭐ | Adapter | Interface compatibility |
| ⭐⭐ | Command | Undo/redo, queuing |
| ⭐⭐ | Facade | Simplified interface |
| ⭐ | Abstract Factory | Related object families |
| ⭐ | Composite | Tree structures |
| ⭐ | Proxy | Access control, caching |

---

## Pattern Selection Guide

### Decision Tree

```
Start Here:
│
├─ Need to CREATE objects?
│  │
│  ├─ Single instance globally? ──────────────> SINGLETON
│  │
│  ├─ Choose type at runtime? ────────────────> FACTORY
│  │
│  ├─ Create families of related objects? ────> ABSTRACT FACTORY
│  │
│  ├─ Step-by-step complex construction? ─────> BUILDER
│  │
│  └─ Clone existing objects? ────────────────> PROTOTYPE
│
├─ Need to STRUCTURE objects?
│  │
│  ├─ Convert interface to another? ──────────> ADAPTER
│  │
│  ├─ Add behavior dynamically? ──────────────> DECORATOR
│  │
│  ├─ Simplify complex subsystem? ────────────> FACADE
│  │
│  ├─ Tree/hierarchy structure? ──────────────> COMPOSITE
│  │
│  └─ Control access to object? ──────────────> PROXY
│
└─ Need to define BEHAVIOR?
   │
   ├─ Switch algorithms at runtime? ──────────> STRATEGY
   │
   ├─ Notify multiple objects of change? ─────> OBSERVER
   │
   ├─ Encapsulate request as object? ─────────> COMMAND
   │
   ├─ Object behavior depends on state? ──────> STATE
   │
   ├─ Define algorithm skeleton? ─────────────> TEMPLATE METHOD
   │
   ├─ Pass request through chain? ────────────> CHAIN OF RESPONSIBILITY
   │
   └─ Traverse collection without exposing? ──> ITERATOR
```

### Common Combinations

```
┌─────────────────────────────────────────────────────────────┐
│              PATTERN COMBINATIONS                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Factory + Singleton                                       │
│   └─ Single factory instance creating objects               │
│                                                             │
│   Strategy + Factory                                        │
│   └─ Factory creates appropriate strategy                   │
│                                                             │
│   Observer + Command                                        │
│   └─ Commands notify observers of execution                 │
│                                                             │
│   Decorator + Factory                                       │
│   └─ Factory creates decorated objects                      │
│                                                             │
│   State + Singleton                                         │
│   └─ State objects as singletons                            │
│                                                             │
│   Composite + Iterator                                      │
│   └─ Traverse tree structure                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Quick Reference

### Creational Patterns Summary

| Pattern | Intent | Example |
|---------|--------|---------|
| **Singleton** | Ensure one instance | Logger, Config |
| **Factory** | Create without exposing logic | Document types |
| **Abstract Factory** | Create families | UI themes |
| **Builder** | Step-by-step construction | SQL query builder |
| **Prototype** | Clone objects | Document templates |

### Structural Patterns Summary

| Pattern | Intent | Example |
|---------|--------|---------|
| **Adapter** | Convert interface | Legacy integration |
| **Decorator** | Add behavior | Coffee toppings |
| **Facade** | Simplify interface | Home theater |
| **Composite** | Tree structure | File system |
| **Proxy** | Control access | Lazy loading |

### Behavioral Patterns Summary

| Pattern | Intent | Example |
|---------|--------|---------|
| **Strategy** | Interchangeable algorithms | Payment methods |
| **Observer** | Notify on changes | Event systems |
| **Command** | Encapsulate request | Undo/redo |
| **State** | Behavior by state | Order lifecycle |
| **Template Method** | Algorithm skeleton | Data parsers |
| **Chain of Responsibility** | Pass request | Request handlers |
| **Iterator** | Traverse collection | Collections |

---

## Detailed Pattern Guides

Navigate to detailed guides for each category:

- **[Creational Patterns](./creational/README.md)**: Object creation mechanisms
- **[Structural Patterns](./structural/README.md)**: Object composition
- **[Behavioral Patterns](./behavioral/README.md)**: Object interaction

---

**Start with: [Creational Patterns →](./creational/README.md)**
