---
name: problem-definition_v3
description: Define and sharpen a problem before any solution work begins. Use this skill when the user wants to define, frame, scope, or clarify a problem; when they describe a vague pain point, symptom, complaint, or frustration that needs sharpening into a well-posed problem; when they ask "what is the actual problem here", want to write a problem statement, or need to capture what is wrong, who is affected, and what success looks like before research or requirements. Conducts a questioning interview and produces a structured problem-definition artifact stored in SCS.
version: 0.1.0
---

# Problem Definition

To define a problem, at a minimum we must know what the problem is and who is experiencing the problem and what success looks like?

If we don't have at least those three, then the problem is ill posed.

In order for the problem to be well framed, other supporting information could be

- **Why(s)** Why is this a problem for the effected party? Five Why's is a good model for deciphering this.
- **Alternatives** What workarounds or alternatives do those experiencing the problem currently use?
- **Scope** Where does the problem begin and end? What are we explicitly not solving?
- **Severity** How critical is it that we solve this problem?
- **Constraints** What are the constraints that any solution must adhere to?
- **Priority** If there are other related problems is this more or less important?
- **Evidence** What supporting evidence is there for this problem occuring?

Keep asking the user questions to get as much of this information out of them as possible, then store it as a problem-definition artifact. Limit the problem definition to 150 lines.

Validate that the problem definition is well posed. It should be clear what 'solved' looks like.

## Storing the problem definition

Store the document as the `problem` artifact on the concept named `{feature}` (kebab-case) for a standalone problem, or `{initiative}` when the problem feeds a roadmap. Use the `artifacts_v3` skill for how to do this — concept addressing, creating the concept, reading and writing revisions, conflict handling, and the sign-in preflight.
