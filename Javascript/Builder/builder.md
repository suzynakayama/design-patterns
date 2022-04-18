# BUILDER Design Principle

Some object are simple and ca be created in a single initializer call. However some objects require a lot of steps to be created. So having an object with 10 initializer arguments is not productive. Instead we opt for a piecewise construction.

The `Builder` design pattern provides an API for constructing an object step-by-step.

When piecewise object construction is complicated, the `Builder` provide an API for doing it succinctly.

A builder is a separate component for building an object. You can either give it an initializer or return it wia a static function.
To make the builder fluent, return self.
Different facets of an object can be built with different builders working in tandem via a base class.

Example. We want an html list of words.
```javascript
const words = ['hello', 'world']
html = []
html.push('<ul>\n')
for (let word of words) 
    html.push(`  <li>${word}</li>\n`)
html.push('</ul>')
console.log(html.join(''))
```

Better approach:
```javascript
class Tag {
    constructor(name='', text='') {
        this.name = name;
        this.text = text;
        this.children = [];
    }

    static get indentSize() { return 2; }

    toStringImplementation(indent) {
        let html = [];
        let i = ' '.repeat(indent * Tag.indentSize);
        html.push(`${i}<${this.name}>\n`);
        if (this.text.length > 0) {
            html.push(' '.repeat(Tag.indentSize * (indent + 1)));
            html.push(this.text);
            html.push('\n');
        }

        for (let child of this.children)
            html.push(child.toStringImplementation(indent + 1));

        html.push(`${i}</${this.name}>\n`);
        return html.join('');
    }

    toString() { return this.toStringImplementation(0); }

    static create(name) { return new HtmlBuilder(name); }
}

class HtmlBuilder {
    constructor(rootName) {
        this.root = new Tag(rootName);
        this.rootName = rootName;
    }

    // non-fluent
    addChild(childName, childText) {
        let child = new Tag(childName, childText);
        this.root.children.push(child);
    }

    // fluent - very common when dealing with interfaces
    addChildFluent(childName, childText) {
        let child = new Tag(childName, childText);
        this.root.children.push(child);
        return this;
    }

    toString() { return this.root.toString(); }

    build() { return this.root; }

    clear() { this.root = new Tag(this.rootName); }
}

const builder = new HtmlBuilder('ul');
for (let word of words) builder.addChild('li', word);
console.log(builder.build().toString());

// initializing through the TAG object - this introduces coupling, so not a good way to do it
const builder1 = Tag.create('ul');
for (let word of words) builder.addChild('li', word);
console.log(builder.build().toString());

builder.clear();
builder.addChildFluent('li', 'foo').addChildFluent('li', 'bar').addChildFluent('li', 'baz')
console.log(builder.toString())
```

Sometimes a single builder is not enough to build the object. So you can introduce several builders within a bigger builder in order to build the proper interface.

```javascript
class Person {
    constructor() { 
        // address
        this.streetAddress = this.postcode = this.city = '';

        // employment
        this.companyName = this.position = '';
        this.annualIncome = 0;
    }

    toString() { 
        return `Person lives at ${this.streetAddress}, ${this.city}, ${this.postcode}\n`
        + `and works at ${this.companyName} as a ${this.position} earning ${this.annualIncome}`;
    }
}

class PersonBuilder {
    constructor(person = new Person()) {    // we create the One single object and store it inside the PersonBuilder
        this.person = person;
    }

    get lives() { 
        return new PersonAddressBuilder(this.person);
    }

    get works() { 
        return new PersonJobBuilder(this.person);
    }

    build() { return this.person; }
}

class PersonAddressBuilder extends PersonBuilder {
    constructor(person) {
        super(person);
    }

    at(streetAddress) {
        this.person.streetAddress = streetAddress;
        return this;
    }

    withPostcode(postcode) {
        this.person.postcode = postcode;
        return this;
    }

    in(city) {
        this.person.city = city;
        return this;
    }
}

class PersonJobBuilder extends PersonBuilder {
    constructor(person) {
        super(person);
    }

    at(companyName) {
        this.person.companyName = companyName;
        return this;
    }

    asA(position) {
        this.person.position = position;
        return this;
    }

    earning(annualIncome) {
        this.person.annualIncome = annualIncome;
        return this;
    }
}

let pb = new PersonBuilder();
let person = pb.lives.at('123 London Road').in('London').withPostcode('SW12BC').works.at('Fabrikam').asA('Engineer').earning(123000).build();
console.log(person.toString());
```