# Chat History Response Model

## Overview

The **Chat History Response Model** defines a standardized response format for all chat interactions between the client application and the Question Generation API.

The primary objective of this model is to provide a **consistent and extensible structure** for all generated content, regardless of whether the user requests:

- Quiz
- Concept Questions
- Coding Problems

Instead of returning multiple response collections such as `quiz`, `concept_questions`, and `coding_problems`, every assistant response returns a **single collection of generated items**.

This significantly simplifies frontend rendering while making the API easier to extend.

---

# Design Goals

The response model is designed with the following goals:

- Maintain complete chat history
- Preserve the original user prompt
- Support multiple generated content types
- Provide a single rendering model for the UI
- Minimize frontend conditional logic
- Support future content types without breaking API contracts
- Keep common fields standardized while allowing type-specific extensions

---

# Conversation Structure

A conversation consists of an ordered list of messages.

Each message belongs to one of two roles:

- **User**
- **Assistant**

```json
[
  {
    "role": "user",
    "text": "Generate 2 quiz"
  },
  {
    "role": "assistant",
    "request": {},
    "items": []
  }
]
```

---

# User Message

Represents the prompt entered by the user.

## Schema

```json
{
  "role": "user",
  "text": "Generate 2 quiz"
}
```

## Fields

| Field | Type | Description |
|--------|------|-------------|
| role | String | Always `"user"` |
| text | String | Exact prompt entered by the user |

---

# Assistant Message

Represents the generated content corresponding to the previous user prompt.

## Schema

```json
{
  "role": "assistant",
  "request": {},
  "items": []
}
```

The assistant message consists of two primary sections:

- `request`
- `items`

---

# Request Object

The `request` object stores metadata describing what the user requested.

## Schema

```json
{
  "prompt": "Generate 2 quiz",
  "type": "quiz",
  "count": 2
}
```

## Fields

| Field | Type | Description |
|--------|------|-------------|
| prompt | String | Original prompt entered by the user |
| type | String | Requested generation type |
| count | Integer | Number of requested items |

## Supported Types

Current supported values:

```text
quiz
concept_question
coding_problem
```

Future supported values may include:

```text
flashcard
assignment
summary
notes
case_study
lab_exercise
```

No changes to the response structure are required when introducing new content types.

---

# Items Collection

The assistant always returns generated content inside the `items` array.

Even when a single item is generated, the response still returns an array.

## Example

```json
{
  "items": [
    {
      ...
    }
  ]
}
```

This provides a consistent rendering experience on the frontend.

---

# Standard Item Model

Every generated item follows the same base schema.

```json
{
  "id": "",
  "type": "",
  "title": "",
  "question": "",
  "difficulty": "",
  "edited": false,
  "metadata": {}
}
```

---

# Common Item Fields

| Field | Type | Description |
|--------|------|-------------|
| id | UUID | Unique identifier for the generated item |
| type | String | Type of generated content |
| title | String / null | Optional title |
| question | String | Main question or generated content |
| difficulty | String | `basic`, `intermediate`, or `advanced` |
| edited | Boolean | Indicates whether the content has been edited |
| metadata | Object | Content-specific information |

---

# Metadata

The `metadata` object contains fields that are unique to each content type.

This separation allows the common response schema to remain stable while supporting specialized data.

---

# Quiz Item

## Schema

```json
{
  "id": "9c9be0e6-a093-46d1-80bf-bc8a58d30b54",
  "type": "quiz",
  "title": null,
  "question": "In Java's constructor chaining...",
  "difficulty": "basic",
  "edited": false,
  "metadata": {
    "quizType": "MCQ",
    "options": [],
    "correctAnswers": [],
    "explanation": ""
  }
}
```

## Quiz Metadata

| Field | Description |
|--------|-------------|
| quizType | MCQ, MSQ, TrueFalse |
| options | Available answer choices |
| correctAnswers | Correct option(s) |
| explanation | Explanation for the correct answer |

---

# Coding Problem

## Schema

```json
{
  "id": "4e58b16c-2404-472b-b745-c99d1d7b1cf5",
  "type": "coding_problem",
  "title": "Multilevel Animal Hierarchy",
  "question": "Design a Java program...",
  "difficulty": "basic",
  "edited": false,
  "metadata": {
    "instructions": "...",
    "topics": [
      "Inheritance",
      "Method Overriding"
    ]
  }
}
```

## Coding Metadata

| Field | Description |
|--------|-------------|
| instructions | Step-by-step implementation instructions |
| topics | Related notebook topics |

---

# Concept Question

## Schema

```json
{
  "id": "521fbd8d-f8c8-4c74-9515-b7435f8f406d",
  "type": "concept_question",
  "title": "IS-A vs. CAN-DO Relationships",
  "question": "Explain how the IS-A relationship...",
  "difficulty": "basic",
  "edited": false,
  "metadata": {
    "topics": [
      "Inheritance",
      "Interfaces"
    ]
  }
}
```

## Concept Metadata

| Field | Description |
|--------|-------------|
| topics | Related notebook concepts |

---

# Complete Conversation Example

```json
[
  {
    "role": "user",
    "text": "Generate 2 quiz"
  },
  {
    "role": "assistant",
    "request": {
      "prompt": "Generate 2 quiz",
      "type": "quiz",
      "count": 2
    },
    "items": [
      {
        "id": "9c9be0e6-a093-46d1-80bf-bc8a58d30b54",
        "type": "quiz",
        "title": null,
        "question": "In Java's constructor chaining...",
        "difficulty": "basic",
        "edited": false,
        "metadata": {
          "quizType": "MCQ",
          "options": [
            "...",
            "...",
            "...",
            "...",
            "..."
          ],
          "correctAnswers": [
            "..."
          ],
          "explanation": "..."
        }
      },
      {
        "id": "17f98fbb-769e-4c51-a1e8-901386229af6",
        "type": "quiz",
        "title": null,
        "question": "Which of the following statements are true about method overloading?",
        "difficulty": "basic",
        "edited": false,
        "metadata": {
          "quizType": "MSQ",
          "options": [
            "...",
            "...",
            "...",
            "...",
            "..."
          ],
          "correctAnswers": [
            "...",
            "..."
          ],
          "explanation": "..."
        }
      }
    ]
  },
  {
    "role": "user",
    "text": "Generate 1 coding question"
  },
  {
    "role": "assistant",
    "request": {
      "prompt": "Generate 1 coding question",
      "type": "coding_problem",
      "count": 1
    },
    "items": [
      {
        "id": "4e58b16c-2404-472b-b745-c99d1d7b1cf5",
        "type": "coding_problem",
        "title": "Multilevel Animal Hierarchy",
        "question": "Design a Java program...",
        "difficulty": "basic",
        "edited": false,
        "metadata": {
          "instructions": "...",
          "topics": [
            "Inheritance",
            "Method Overriding"
          ]
        }
      }
    ]
  },
  {
    "role": "user",
    "text": "Generate 1 concept question"
  },
  {
    "role": "assistant",
    "request": {
      "prompt": "Generate 1 concept question",
      "type": "concept_question",
      "count": 1
    },
    "items": [
      {
        "id": "521fbd8d-f8c8-4c74-9515-b7435f8f406d",
        "type": "concept_question",
        "title": "IS-A vs. CAN-DO Relationships",
        "question": "Explain how the IS-A relationship is used in Java.",
        "difficulty": "basic",
        "edited": false,
        "metadata": {
          "topics": [
            "Inheritance",
            "Interfaces"
          ]
        }
      }
    ]
  }
]
```

---

# UI Rendering Strategy

Since every assistant response follows the same schema, the frontend only needs to iterate over the conversation and render based on the message role.

Example (TypeScript):

```typescript
chatHistory.forEach(message => {
    if (message.role === "user") {
        renderUserMessage(message.text);
        return;
    }

    message.items.forEach(item => {
        renderTitle(item.title);
        renderQuestion(item.question);
        renderDifficulty(item.difficulty);

        switch (item.type) {
            case "quiz":
                renderQuiz(
                    item.metadata.options,
                    item.metadata.correctAnswers,
                    item.metadata.explanation
                );
                break;

            case "coding_problem":
                renderInstructions(item.metadata.instructions);
                renderTopics(item.metadata.topics);
                break;

            case "concept_question":
                renderTopics(item.metadata.topics);
                break;
        }
    });
});
```

---

# Advantages

## 1. Unified Response Structure

Every assistant response follows the same schema regardless of content type.

---

## 2. Simplified Frontend

The frontend no longer needs to inspect multiple response arrays such as:

- `quiz`
- `concept_questions`
- `coding_problems`

Instead, it always renders a single `items` collection.

---

## 3. Extensible Design

Adding a new content type only requires:

- Introducing a new `type`
- Defining the corresponding `metadata`

No changes are required to the existing response model.

---

## 4. Strong Separation of Concerns

Common fields remain at the top level.

Content-specific information is isolated within `metadata`.

This minimizes duplication and keeps the schema stable.

---

## 5. Easier Chat History Management

Each assistant response includes the `request` object, preserving:

- Original prompt
- Requested content type
- Requested item count

This makes it straightforward to:

- Replay conversations
- Restore chat history
- Debug generation requests
- Display contextual information in the UI

---

# Future Extensions

The current schema supports seamless expansion.

| Type | Metadata Example |
|------|------------------|
| flashcard | front, back, hint |
| assignment | rubric, submissionFormat, dueDate |
| summary | keyTakeaways, sections |
| notes | headings, references |
| case_study | scenario, questions |
| lab_exercise | prerequisites, expectedOutput |

Since the core structure remains unchanged, future content types can be introduced without affecting existing frontend implementations.

---

# Conclusion

The proposed Chat History Response Model establishes a stable, scalable, and frontend-friendly API contract. By standardizing all assistant 
responses around a single `items` collection and encapsulating content-specific fields within `metadata`, the model simplifies UI development, 
improves maintainability, and provides a clear path for supporting additional educational content types in the future.

### Design:
<img width="2560" height="3574" alt="Image" src="https://github.com/user-attachments/assets/05cd5d61-e83b-45da-b7a5-4f69088fe680" />

### Final history Response:

```
[
  {
    "role": "user",
    "text": "Generate 2 quiz"
  },
  {
    "role": "assistant",
    "request": {
      "prompt": "Generate 2 quiz",
      "type": "quiz",
      "count": 2
    },
    "items": [
      {
        "id": "9c9be0e6-a093-46d1-80bf-bc8a58d30b54",
        "type": "quiz",
        "title": null,
        "question": "In Java's constructor chaining, what is the sequence of constructor invocation when a subclass object is created?",
        "difficulty": "basic",
        "edited": false,
        "metadata": {
          "quizType": "MCQ",
          "options": [
            "The subclass constructor executes first, then implicitly calls the superclass constructor.",
            "The superclass constructor is always explicitly called by the subclass constructor first.",
            "The superclass constructor executes first (either explicitly or implicitly), followed by the subclass constructor.",
            "Constructors are called simultaneously for both superclass and subclass.",
            "Only the subclass constructor is invoked, unless `super()` is explicitly used."
          ],
          "correctAnswers": [
            "The superclass constructor executes first (either explicitly or implicitly), followed by the subclass constructor."
          ],
          "explanation": "The notebook states: 'When a subclass object is created, Java automatically calls the superclass constructor first (either explicitly using super(), or implicitly if no call is written) before executing the subclass constructor body.' This establishes the correct sequence of invocation."
        }
      },
      {
        "id": "17f98fbb-769e-4c51-a1e8-901386229af6",
        "type": "quiz",
        "title": null,
        "question": "Which of the following statements are true about method overloading in Java?",
        "difficulty": "basic",
        "edited": false,
        "metadata": {
          "quizType": "MSQ",
          "options": [
            "Method overloading occurs when multiple methods in the same class have different names but the same parameters.",
            "Overloaded methods must have the same return type.",
            "Method overloading is a form of compile-time polymorphism.",
            "Overloaded methods must differ in the number, type, or order of their parameters.",
            "The `@Override` annotation is mandatory for overloaded methods."
          ],
          "correctAnswers": [
            "Method overloading is a form of compile-time polymorphism.",
            "Overloaded methods must differ in the number, type, or order of their parameters."
          ],
          "explanation": "The notebook defines method overloading as occurring when multiple methods in the same class share the same name but differ in the number, type, or order of parameters."
        }
      }
    ]
  },
  {
    "role": "user",
    "text": "Generate 1 coding question"
  },
  {
    "role": "assistant",
    "request": {
      "prompt": "Generate 1 coding question",
      "type": "coding_problem",
      "count": 1
    },
    "items": [
      {
        "id": "4e58b16c-2404-472b-b745-c99d1d7b1cf5",
        "type": "coding_problem",
        "title": "Multilevel Animal Hierarchy with Constructor Chaining and Method Overriding",
        "question": "Design a Java program to model a multilevel animal hierarchy, demonstrating constructor chaining and method overriding across different levels.",
        "difficulty": "basic",
        "edited": false,
        "metadata": {
          "instructions": "1. Define a superclass Animal with a constructor that takes a String name and prints 'Animal: [name] created'. 2. Create a subclass Mammal that extends Animal. 3. Create a subclass Dog that extends Mammal. 4. Instantiate Dog in Main. 5. Call speak(). 6. Verify constructor chaining and method overriding.",
          "topics": [
            "Inheritance",
            "Multilevel Inheritance",
            "Constructor Chaining",
            "super keyword",
            "Method Overriding"
          ]
        }
      }
    ]
  },
  {
    "role": "user",
    "text": "Generate 1 concept question"
  },
  {
    "role": "assistant",
    "request": {
      "prompt": "Generate 1 concept question",
      "type": "concept_question",
      "count": 1
    },
    "items": [
      {
        "id": "521fbd8d-f8c8-4c74-9515-b7435f8f406d",
        "type": "concept_question",
        "title": "IS-A vs. CAN-DO Relationships in Design",
        "question": "Explain how the 'IS-A' and 'CAN-DO' relationships serve as guiding principles for structuring a Java application, particularly when deciding whether to use class extension (extends) or interface implementation (implements). Provide an example of a class that demonstrates both types of relationships.",
        "difficulty": "basic",
        "edited": false,
        "metadata": {
          "topics": [
            "IS-A relationship",
            "CAN-DO relationship",
            "Inheritance",
            "Interfaces",
            "Design"
          ]
        }
      }
    ]
  }
]

```
