### Index Signature

- A Feature in Typescript that lets you define `types` for objects whose property names are not known in advance.
- Called so because it derives from the broader idea of indexing into a data structure.

```ts
type Scores = {
  [studentName: string]: number;
};
//A data structure that conforms to Scores type can have any number of string-named properties with their values being a number type

const scores: Scores = {
  Alice: 95,
  Bob: 88,
};

scores.Charlie = 91;
```

### Compile Time vs Runtime Typescript world

- Things like `let` and `const` create objects that actually exist in memory at runtime.
- Things like `type` do not create objects that exist in memory. They purely serve a description used by the compiler

```ts
type User = {
  name: string;
};

const user: User = {
  name: "Manpreet",
};
//`user` exists when the code runs. `User` only helps TypeScript check the code before it runs.
```

### Indexed Access Type

A feature of typescript allows you to access `type` of elements nested in other `types` by indexing into them using `types`.

```ts
type Person = {
  name: string;
  age: number;
  skills: string[];
};

type PersonName = Person["name"]; //returns string
type PersonAge = Person["age"]; // returns number because Typescript treats age as a string literal type. 
type Skill = Person["skills"][number]; // returns string because If we access the `skills` type from `Person`, we get `string[]`. Then, if we index into that array type using `number`, we get the element type: `string`.



