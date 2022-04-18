# Design Patterns

Design Patterns are common architectural approaches. It was popularized by the `Gang of Four` book (1994).
Are universally relevant and have since being internalized in some programming languages and several libraries.

The Patterns are:
![patterns](images/patterns.png)

Design Patterns are typycally split into 3 categories and this is called `Gamma Categorization` (after Erich Gamma, one of the authors of the GoF book):
- Creational Patterns:
  - Deal with the creation (construction) of objects
  - Explicit (constructor) vs implicit (DI, reflection, etc)
  - Wholesale (single statement/call in order to initialize the object) vs Piecewise (step-by-step initialization)
- Structural Patterns:
  - Concerned with the structure (ex. class members)
  - Many patterns are wrappers that mimic the underlying class interface
  - Stress the importance of good API design
- Behavioral Patterns:
  - They are all different, no central theme
