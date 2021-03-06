# SOLID Design Principles

Introduced by Robert C. Martin and is frequently referenced in Design Pattern literature.

It is set of 5 principles:
- Single Responsibility Principle
- Open Closed Principle
- Liskov Substitution Principle
- Interface Segregation Principle
- Dependency Inversion Principle

## Single Responsibility Principle

A class should have a SINGLE primary responsibility and as consequence should only ONE reason to change, that reason being related to that responsibility. In other words is a bad idea to add more than one responsibility to a class.

```javascript
// main responsibility: keep entries
class Journal {
    constructor() {
        this.entries = {};
    }

    addEntry(text) {
        let c = ++Journal.count;
        let entry = `${c}: ${text}`;
        this.entries[c] = entry;
        return c;
    }

    removeEntry(idx) {
        delete this.entries[idx];
    }

    toString() {
        return Object.values(this.entries).join('\n');
    }
}

const j = new Journal();
j.addEntry('I had good and bad news today')
j.addEntry('I love my dog')
console.log(j.toString());
```

If you added save to a file and then load from a file, and maybe in the future, also load from a url to the same Journal class you would be adding a second responsibility to it. You should have these persistance operation and add to a second class that could eventually be generalize and be used for other classes, not only the Journal. Ex.

```javascript
import fs from 'fs'

class PersistenceManager {
    preprocess(notes) {...}
    saveToFile(notes, filename) {
        fs.writeFileSync(filename, notes.toString)
    }
    loadFromFile(filename) {...}
    loadFromUrl(url) {...}
}

const persistance = new PersistenceManager()
persistance.saveToFile(j, './myJournal')
```

There is an **anti-pattern** called `the God object`, which is this huge massive class with lots of responsibilities and spaghetti code to do everything.

**Separation of Concerns** is what you do when you refactor. You split the code up into smaller portions (separate components) with a single responsibility.

## Open Closed Principle (OCP)

It basically says that objects are OPEN for extension (inheritance) but CLOSED for modifications (add more things into a class).

Let's imagine we have a website that allows people to search for different products based on a certain criteria.

```javascript
const COLOR = Object.freeze({
    red: 'red',
    green: 'green',
    blue: 'blue'
})
const SIZE = Object.freeze({
    small: 'small',
    medium: 'medium',
    large: 'large'
})

class Product {
    constructor(name, color, size) {
        this.name = name;
        this.color = color;
        this.size = size;
    }
}

class ProductFilter {
    filterByColor(products, color) {
        return products.filter(p => p.color === color);
    }
    // then boss ask to add the size filter -> this is a modification
    filterBySize(products, size) {
        return products.filter(p => p.size === size);
    }
    // then add filter by color and size -> another modification
    filterBySizeAndColor(products, size, color) {
        return products.filter(p => p.size === size && p.color === color);
    }
    // if this goes one we will have the State Space Explosion, where it never ends.
    // ex. 3 criteria = 7 methods
}

// Using OCP:
class Specification {
    constructor() {
        if (this.constructor.name === 'Specification')
            throw new Error('Specification is abstract!')
    }
    isSatisfied(item) {
        throw new Error('Not implemented!')
    }
}

class ColorSpecification extends Specification {
    constructor(color) {
        super();
        this.color = color;
    }
    isSatisfied(item) {
        return item.color === this.color;
    }
}

class SizeSpecification extends Specification {
    constructor(size) {
        super();
        this.size = size;
    }
    isSatisfied(item) {
        return item.size === this.size;
    }
}

class AndSpecification extends Specification {
    constructor(...specs) {
        super();
        this.specs = specs;
    }
    isSatisfied(item) {
        return this.specs.every(spec => spec.isSatisfied(item))
    }
}

class OrSpecification extends Specification {
    constructor(...specs) {
        super();
        this.specs = specs;
    }
    isSatisfied(item) {
        return this.specs.some(spec => spec.isSatisfied(item))
    }
}

class BetterFilter {
    filter(items, spec) {
        return items.filter(item => spec.isSatisfied(item))
    }
}

const apple = new Product('Apple', COLOR.red, SIZE.small);
const greenApple = new Product('Green Apple', COLOR.green, SIZE.small);
const tree = new Product('Tree', COLOR.green, SIZE.large);
const house = new Product('House', COLOR.blue, SIZE.large);
const car = new Product('Car', COLOR.blue, SIZE.medium);

const products = [apple, greenApple, tree, house, car];

const pf = new ProductFilter();
for (const p of pf.filterByColor(products, COLOR.green)) console.log(`${p.name} is green`)

const bf = new BetterFilter();
for (const p of bf.filter(products, new ColorSpecification(COLOR.green))) console.log(`${p.name} is green`)

for (const p of bf.filter(products, new SizeSpecification(SIZE.large))) console.log(`${p.name} is large`)

const andSpec = new AndSpecification(
    new SizeSpecification(SIZE.large),
    new ColorSpecification(COLOR.green)
);
for (const p of bf.filter(products, andSpec)) console.log(`${p.name} is large and green`)

const orSpec = new OrSpecification(
    new SizeSpecification(SIZE.large),
    new ColorSpecification(COLOR.green)
);
for (const p of bf.filter(products, orSpec)) console.log(`${p.name} is large or green`)
```

## Liskov Substitution Principle

The idea behind this principle is that if you have some method within a class that takes come base type, it should also equally be able to take a derived type.

```javascript
class Rectangle {
    constructor(w, h) {
        this.width = w;
        this.height = h;
    }

    get area() {
        return this.width * this.height;
    }

    toString() {
        return `${this.width} x ${this.height}`;
    }
}

class Square extends Rectangle {
    constructor(size) {
        super(size, size)
    }
    // how to impose the same size?
}
```

One thing we could do is to set getters and setters. However, as we can see from the example below this is not a good idea, because the `useIt` function will not work properly for the square. 

```javascript
class Rectangle
{
    constructor(width, height)
    {
        this._width = width;
        this._height = height;
    }

    get width() { return this._width; }
    get height() { return this._height; }

    set width(value) { this._width = value; }
    set height(value) { this._height = value; }

    get area()
    {
        return this._width * this._height;
    }

    toString()
    {
        return `${this._width}??${this._height}`;
    }
}

class Square extends Rectangle
{
    constructor(size)
    {
        super(size, size);
    }

    set width(value)
    {
        this._width = this._height = value;
    }

    set height(value)
    {
        this._width = this._height = value;
    }
}

let useIt = function(rc)
{
    let width = rc._width;
    rc.height = 10;
    console.log(
        `Expected area of ${10*width}, got ${rc.area}`
    );
};

let rc = new Rectangle(2,3);
useIt(rc);

let sq = new Square(5);
useIt(sq);
```

The `useIt` function receives a base class, so according to this principle, it should also work with derived classes, like the square. Unfortunately this way of "fixing" the code is not proper, since it goes against this principle.

We should be using factory design here. *(We will see it further along the course)*

## Interface Segregation Principle

You need to segregate/split the interface into several small interfaces, so people won't have to implement things they don't need.

Ex. We implement the machine class, where each client can create their own machine. However, because we put 3 methods within the machine class, the clients will have to implement (or throw an error) even if they didn't wanted/needed.
```javascript
class Machine
{
    constructor()
    {
        if (this.constructor.name === 'Machine')
        throw new Error('Machine is abstract!');
    }

    print(doc) {}
    fax(doc) {}
    scan(doc) {}
}

class MultiFunctionPrinter extends Machine
{
    print(doc) {
        //
    }

    fax(doc) {
        //
    }

    scan(doc) {
        //
    }
}

class NotImplementedError extends Error
{
    constructor(name)
    {
        let msg = `${name} is not implemented!`;
        super(msg);
        // maintain proper stack trace
        if (Error.captureStackTrace)
        Error.captureStackTrace(this, NotImplementedError);
        // your custom stuff here :)
    }
}

class OldFashionedPrinter extends Machine   // => only prints
{
    print(doc) {
        // it will work
    }

    // omitting fax is the same as no-op impl

    scan(doc) {
        // throw new Error('not implemented!');
        throw new NotImplementedError(
        'OldFashionedPrinter.scan')
    }
}

// solution
class Printer
{
    constructor()
    {
        if (this.constructor.name === 'Printer')
        throw new Error('Printer is abstract!');
    }

    print(){}
}

class Scanner
{
    constructor()
    {
        if (this.constructor.name === 'Scanner')
        throw new Error('Scanner is abstract!');
    }

    scan(){}
}

// we don't allow this!
// let m = new Machine();

let printer = new OldFashionedPrinter();
printer.fax(); // nothing happens
//printer.scan();
```

**Principle of Least Surprise**, when people use your api they should not be surprised, they should not be seeing some magical behavior. They should be getting predictable results.

**YAGNI** - You Ain't Going to Need It

## Dependency Inversion Principle

Defines a relationship that you should have between a low level models and high level models. High level modules should not depend on low level modules. It should be a abstract class/interface in between.

```javascript
let Relationship = Object.freeze({
    parent: 0,
    child: 1,
    sibling: 2
});

class Person
{
    constructor(name)
    {
        this.name = name;
    }
}

// LOW-LEVEL (STORAGE)
class RelationshipBrowser
{
    constructor()
    {
        if (this.constructor.name === 'RelationshipBrowser')
        throw new Error('RelationshipBrowser is abstract!');
    }

    findAllChildrenOf(name) {}
}

class Relationships extends RelationshipBrowser
{
    constructor()
    {
        super();
        this.data = [];
    }

    addParentAndChild(parent, child)
    {
        this.data.push({
        from: parent,
        type: Relationship.parent,
        to: child
        });
        this.data.push({
        from: child,
        type: Relationship.child,
        to: parent
        });
    }


    findAllChildrenOf(name) {
        return this.data.filter(r =>
        r.from.name === name &&
        r.type === Relationship.parent
        ).map(r => r.to);
    }
}

// HIGH-LEVEL (RESEARCH)
class Research
{
    // constructor(relationships)
    // {
    //   // problem: direct dependence ???????????? on storage mechanic
    //   let relations = relationships.data;
    //   for (let rel of relations.filter(r =>
    //     r.from.name === 'John' &&
    //     r.type === Relationship.parent
    //   ))
    //   {
    //     console.log(`John has a child named ${rel.to.name}`);
    //   }
    // }

    constructor(browser)
    {
        for (let p of browser.findAllChildrenOf('John'))
        {
        console.log(`John has a child named ${p.name}`);
        }
    }
}

let parent = new Person('John');
let child1 = new Person('Chris');
let child2 = new Person('Matt');

// low-level module
let rels = new Relationships();
rels.addParentAndChild(parent, child1);
rels.addParentAndChild(parent, child2);

new Research(rels);
```