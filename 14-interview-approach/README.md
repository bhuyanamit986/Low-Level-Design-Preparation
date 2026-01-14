# 📘 Section 14: How to Approach LLD Interviews

> **The process is as important as the solution**

---

## 📑 Table of Contents

1. [The Interview Process](#1-the-interview-process)
2. [Step-by-Step Approach](#2-step-by-step-approach)
3. [Communication Tips](#3-communication-tips)
4. [Common Mistakes to Avoid](#4-common-mistakes-to-avoid)
5. [Sample Interview Walkthrough](#5-sample-interview-walkthrough)

---

## 1. The Interview Process

### What Interviewers Evaluate

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                         EVALUATION CRITERIA                                         │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   ┌────────────────────────────────────────────────────────────────────────────┐    │
│   │  1. PROBLEM UNDERSTANDING (10%)                                            │    │
│   │     - Do you ask the right clarifying questions?                           │    │
│   │     - Do you understand the requirements correctly?                        │    │
│   └────────────────────────────────────────────────────────────────────────────┘    │
│                                                                                     │
│   ┌────────────────────────────────────────────────────────────────────────────┐    │
│   │  2. DESIGN SKILLS (40%)                                                    │    │
│   │     - Correct identification of entities and relationships                 │    │
│   │     - Appropriate use of design patterns                                   │    │
│   │     - Clean class hierarchy and responsibilities                           │    │
│   └────────────────────────────────────────────────────────────────────────────┘    │
│                                                                                     │
│   ┌────────────────────────────────────────────────────────────────────────────┐    │
│   │  3. CODE QUALITY (30%)                                                     │    │
│   │     - Clean, readable code                                                 │    │
│   │     - Proper naming conventions                                            │    │
│   │     - SOLID principles applied                                             │    │
│   └────────────────────────────────────────────────────────────────────────────┘    │
│                                                                                     │
│   ┌────────────────────────────────────────────────────────────────────────────┐    │
│   │  4. COMMUNICATION (20%)                                                    │    │
│   │     - Clear explanation of thought process                                 │    │
│   │     - Responsive to hints and feedback                                     │    │
│   │     - Discussion of trade-offs                                             │    │
│   └────────────────────────────────────────────────────────────────────────────┘    │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Time Management (45-60 min interview)

| Phase | Time | Activities |
|-------|------|------------|
| **Requirements** | 5-10 min | Clarify, scope, assumptions |
| **High-Level Design** | 10-15 min | Entities, relationships, class diagram |
| **Detailed Design** | 20-25 min | Code key classes, methods |
| **Discussion** | 5-10 min | Trade-offs, extensibility, edge cases |

---

## 2. Step-by-Step Approach

### Step 1: Clarify Requirements (5-10 minutes)

**Questions to Ask:**

```
FUNCTIONAL REQUIREMENTS:
- What are the main use cases?
- Who are the actors/users?
- What operations should the system support?
- What are the inputs and outputs?

SCOPE:
- Are we designing the full system or a specific component?
- Should I consider persistence/database?
- Do we need to handle concurrency?

CONSTRAINTS:
- Expected scale (users, data)?
- Any specific technologies to use?
- Real-time requirements?

EDGE CASES:
- What happens when X fails?
- How to handle invalid inputs?
- Any rate limits or quotas?
```

**Example for Parking Lot:**

> "Let me clarify a few things:
> - How many floors and spots per floor?
> - What types of vehicles do we support?
> - Do we need different pricing for different vehicle types?
> - Should we handle multiple entry/exit points?
> - Do we need real-time availability display?"

### Step 2: Identify Core Entities (5 minutes)

**Process:**
1. Extract **nouns** from requirements → Potential entities
2. Extract **verbs** → Potential methods
3. Group related concepts
4. Identify relationships

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                      ENTITY IDENTIFICATION PROCESS                                  │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   Requirements: "Users can book seats for movie shows at theaters"                  │
│                                                                                     │
│   Nouns (Entities):          Verbs (Methods):                                       │
│   ┌────────────────┐         ┌────────────────┐                                     │
│   │ User           │         │ book()         │                                     │
│   │ Seat           │         │ search()       │                                     │
│   │ Movie          │         │ cancel()       │                                     │
│   │ Show           │         │ pay()          │                                     │
│   │ Theater        │         │ notify()       │                                     │
│   │ Booking        │         └────────────────┘                                     │
│   └────────────────┘                                                                │
│                                                                                     │
│   Relationships:                                                                    │
│   - Theater HAS-MANY Shows                                                          │
│   - Show HAS-MANY Seats                                                             │
│   - User HAS-MANY Bookings                                                          │
│   - Booking BELONGS-TO Show                                                         │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Step 3: Define Responsibilities (5 minutes)

**For each entity, define:**
- What data does it hold? (Attributes)
- What operations does it perform? (Methods)
- Who does it interact with? (Relationships)

```java
// Example: Quick sketch of responsibilities
/*
ParkingLot:
  - Manages floors and spots
  - Handles vehicle entry/exit
  - Tracks availability
  
ParkingSpot:
  - Knows its location and size
  - Can park/unpark a vehicle
  - Tracks if occupied

Vehicle:
  - Has license plate and type
  - Knows what spot size it needs

ParkingTicket:
  - Links vehicle to spot
  - Tracks entry/exit time
  - Calculates fee
*/
```

### Step 4: Design Class Structure (10 minutes)

**Draw a simple class diagram showing:**
- Key classes
- Important attributes (not all)
- Key methods
- Relationships

**Then code the core classes:**

```java
// Start with the most important entity
public class ParkingLot {
    private final List<ParkingFloor> floors;
    private final PricingStrategy pricingStrategy;
    
    public ParkingTicket parkVehicle(Vehicle vehicle) {
        ParkingSpot spot = findAvailableSpot(vehicle)
            .orElseThrow(() -> new ParkingFullException());
        spot.park(vehicle);
        return new ParkingTicket(vehicle, spot);
    }
    
    public double unparkVehicle(String ticketId) {
        // Implementation
    }
}
```

### Step 5: Handle Edge Cases (5 minutes)

**Common edge cases to discuss:**
- What if the system is at capacity?
- What if payment fails?
- How to handle concurrent requests?
- What about invalid inputs?
- Recovery from failures?

### Step 6: Discuss Trade-offs (5 minutes)

**Areas to discuss:**
- Why you chose certain patterns
- Alternative approaches and their trade-offs
- Extensibility considerations
- Performance implications

---

## 3. Communication Tips

### Do's ✓

```
✓ Think out loud - share your thought process
✓ Ask clarifying questions before designing
✓ Start simple, then add complexity
✓ Acknowledge trade-offs
✓ Be receptive to hints
✓ Explain your design choices
✓ Use proper terminology
```

### Don'ts ✗

```
✗ Don't jump into coding immediately
✗ Don't over-engineer
✗ Don't ignore interviewer's hints
✗ Don't stay silent while thinking
✗ Don't argue when given feedback
✗ Don't panic if you make mistakes
```

### Useful Phrases

```
"Let me clarify the requirements first..."
"I'm thinking of using [pattern] here because..."
"One trade-off of this approach is..."
"I could also solve this by... but I chose this because..."
"Let me know if you'd like me to dive deeper into..."
"I'm making an assumption that... is that correct?"
```

---

## 4. Common Mistakes to Avoid

### Mistake 1: No Clarification

```
❌ BAD: "Ok, let me start coding a parking lot..."
✓ GOOD: "Before I begin, let me understand the requirements. 
         How many floors? What vehicle types? Any real-time requirements?"
```

### Mistake 2: Over-Engineering

```
❌ BAD: Designing for millions of users when not required
✓ GOOD: Start simple, mention "we can optimize this later if needed"
```

### Mistake 3: Ignoring SOLID

```
❌ BAD: One god class doing everything
✓ GOOD: Small, focused classes with clear responsibilities
```

### Mistake 4: Not Discussing Trade-offs

```
❌ BAD: "This is the only way to do it"
✓ GOOD: "I chose this approach because X. 
         Alternatively, we could do Y, but it would mean Z"
```

### Mistake 5: Silent Thinking

```
❌ BAD: *long silence while thinking*
✓ GOOD: "I'm thinking about how to handle the case when... 
         Let me consider a few options..."
```

---

## 5. Sample Interview Walkthrough

### Problem: Design a Notification System

**Phase 1: Requirements (5 min)**

> **You:** "Let me clarify the requirements. What types of notifications do we need to support?"
> 
> **Interviewer:** "Email, SMS, and Push notifications."
> 
> **You:** "Should all users receive all types, or can users configure their preferences?"
> 
> **Interviewer:** "Users can set preferences."
> 
> **You:** "Do we need to handle notification templates, or just raw messages?"
> 
> **Interviewer:** "Templates would be good."
> 
> **You:** "Should I consider retry logic for failed deliveries?"
> 
> **Interviewer:** "Yes, that's important."

**Phase 2: Entities (3 min)**

> **You:** "Based on the requirements, I'm identifying these core entities:
> - **Notification**: The message to be sent
> - **User**: Who receives notifications with their preferences
> - **NotificationChannel**: Email, SMS, Push - the delivery mechanism
> - **NotificationTemplate**: Pre-defined message formats
> - **DeliveryStatus**: Track success/failure"

**Phase 3: Design (15 min)**

> **You:** "I'll use the Strategy pattern for different notification channels, making it easy to add new channels. Here's my approach..."

```java
// Strategy interface
public interface NotificationChannel {
    void send(Notification notification, User user);
    String getChannelType();
}

// Concrete strategies
public class EmailChannel implements NotificationChannel {
    private final EmailClient emailClient;
    
    @Override
    public void send(Notification notification, User user) {
        String content = notification.getContent();
        emailClient.send(user.getEmail(), notification.getSubject(), content);
    }
    
    @Override
    public String getChannelType() { return "EMAIL"; }
}

// Notification Service
public class NotificationService {
    private final Map<String, NotificationChannel> channels;
    private final UserPreferenceService preferenceService;
    private final RetryTemplate retryTemplate;
    
    public void send(Notification notification, String userId) {
        User user = userService.getUser(userId);
        Set<String> preferredChannels = preferenceService.getPreferences(userId);
        
        for (String channelType : preferredChannels) {
            NotificationChannel channel = channels.get(channelType);
            
            retryTemplate.execute(() -> {
                channel.send(notification, user);
                logDelivery(notification, user, channelType, DeliveryStatus.SUCCESS);
            }, exception -> {
                logDelivery(notification, user, channelType, DeliveryStatus.FAILED);
            });
        }
    }
}
```

**Phase 4: Discussion (5 min)**

> **You:** "For extensibility, if we need to add a new channel like WhatsApp, we just implement the NotificationChannel interface. No changes to existing code.
> 
> For scalability, we could make this asynchronous using a message queue. Each notification would be published to a queue, and channel-specific consumers would process them.
> 
> One trade-off: I'm sending to all preferred channels in parallel. If we need ordering guarantees, we'd need to adjust this.
> 
> For the retry mechanism, I'm using exponential backoff to avoid overwhelming a failing service."

---

## Checklist Before Interview Ends

```
□ Requirements clarified
□ Core entities identified
□ Class diagram discussed
□ Key code implemented
□ Design patterns explained
□ Edge cases addressed
□ Trade-offs discussed
□ Extensibility considered
□ Questions answered
```

---

## Final Tips

1. **Practice regularly** - Solve 2-3 problems per week
2. **Time yourself** - Get comfortable with 45-60 min sessions
3. **Mock interviews** - Practice with a friend
4. **Review feedback** - Learn from each attempt
5. **Stay calm** - It's okay to not know everything
6. **Be collaborative** - Treat it as a discussion, not an exam

---

**Good luck with your interviews! 🎯**

---

← [Back to Main README](../README.md)
