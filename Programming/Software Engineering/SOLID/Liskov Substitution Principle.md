---
tags:
  - software-engineering
  - solid
  - lsp
created: 2026-01-02
status: 🔴
---
# 🔄 Liskov Substitution Principle (LSP)

> *"Objects of a superclass should be replaceable with objects of its subclasses without breaking the application."* — Barbara Liskov

## 🎯 The Principle

Si `S` es un subtipo de `T`, entonces objetos de tipo `T` pueden ser reemplazados por objetos de tipo `S` sin alterar las propiedades del programa.

**En simple:** Las clases hijas deben poder usarse donde se usa la clase padre sin romper nada.

---

## ❌ Classic Violation: Rectangle-Square

```typescript
// ❌ BAD: Square "is-a" Rectangle mathematically, but not behaviorally

class Rectangle {
  protected _width: number;
  protected _height: number;

  setWidth(width: number): void {
    this._width = width;
  }

  setHeight(height: number): void {
    this._height = height;
  }

  getArea(): number {
    return this._width * this._height;
  }
}

class Square extends Rectangle {
  // Square overrides to maintain square invariant
  setWidth(width: number): void {
    this._width = width;
    this._height = width; // Forces height = width
  }

  setHeight(height: number): void {
    this._width = height; // Forces width = height
    this._height = height;
  }
}

// This function expects Rectangle behavior
function testRectangle(rect: Rectangle) {
  rect.setWidth(5);
  rect.setHeight(4);
  
  // For a rectangle: 5 * 4 = 20
  console.log(rect.getArea()); // Expected: 20
}

const rectangle = new Rectangle();
testRectangle(rectangle); // ✅ Output: 20

const square = new Square();
testRectangle(square); // ❌ Output: 16 (4 * 4) - BROKEN!
```

**The Problem:** Square changes the expected behavior of setWidth/setHeight.

---

## ✅ Fixing LSP Violation

```typescript
// ✅ GOOD: Use a common interface instead of inheritance

interface Shape {
  getArea(): number;
}

class Rectangle implements Shape {
  constructor(
    private width: number,
    private height: number
  ) {}

  getArea(): number {
    return this.width * this.height;
  }
}

class Square implements Shape {
  constructor(private side: number) {}

  getArea(): number {
    return this.side * this.side;
  }
}

// Now they're siblings, not parent-child
function printArea(shape: Shape) {
  console.log(shape.getArea());
}

printArea(new Rectangle(5, 4)); // 20
printArea(new Square(4));        // 16 - Both work correctly!
```

---

## ❌ Another Violation: Bird Example

```typescript
// ❌ BAD: Not all birds can fly!

class Bird {
  fly(): void {
    console.log('Flying...');
  }
}

class Sparrow extends Bird {
  fly(): void {
    console.log('Sparrow flying high!');
  }
}

class Penguin extends Bird {
  fly(): void {
    // Penguins can't fly! What do we do?
    throw new Error("Penguins can't fly!"); // 💥 Breaks LSP
  }
}

function makeBirdFly(bird: Bird) {
  bird.fly(); // Works for Sparrow, crashes for Penguin
}

makeBirdFly(new Sparrow()); // ✅ Works
makeBirdFly(new Penguin()); // ❌ Throws error!
```

---

## ✅ Fixing Bird Example

```typescript
// ✅ GOOD: Separate capabilities into interfaces

interface Bird {
  eat(): void;
  sleep(): void;
}

interface FlyingBird extends Bird {
  fly(): void;
}

interface SwimmingBird extends Bird {
  swim(): void;
}

class Sparrow implements FlyingBird {
  eat(): void { console.log('Eating seeds'); }
  sleep(): void { console.log('Sleeping in nest'); }
  fly(): void { console.log('Flying high!'); }
}

class Penguin implements SwimmingBird {
  eat(): void { console.log('Eating fish'); }
  sleep(): void { console.log('Sleeping standing'); }
  swim(): void { console.log('Swimming fast!'); }
}

// Type-safe functions
function makeFly(bird: FlyingBird) {
  bird.fly();
}

function makeSwim(bird: SwimmingBird) {
  bird.swim();
}

makeFly(new Sparrow()); // ✅ Works
// makeFly(new Penguin()); // ❌ Compile error - Penguin is not FlyingBird
makeSwim(new Penguin()); // ✅ Works
```

---

## 🔄 Real-World Example: File Storage

### ❌ Violating LSP
```typescript
class FileStorage {
  save(path: string, data: string): void {
    fs.writeFileSync(path, data);
  }
  
  read(path: string): string {
    return fs.readFileSync(path, 'utf-8');
  }
  
  delete(path: string): void {
    fs.unlinkSync(path);
  }
}

class ReadOnlyStorage extends FileStorage {
  save(path: string, data: string): void {
    throw new Error('Cannot save to read-only storage'); // 💥 LSP violation
  }
  
  delete(path: string): void {
    throw new Error('Cannot delete from read-only storage'); // 💥 LSP violation
  }
}

function backupData(storage: FileStorage, data: string) {
  storage.save('/backup/data.txt', data); // Crashes with ReadOnlyStorage!
}
```

### ✅ Following LSP
```typescript
interface ReadableStorage {
  read(path: string): string;
}

interface WritableStorage extends ReadableStorage {
  save(path: string, data: string): void;
}

interface DeletableStorage extends ReadableStorage {
  delete(path: string): void;
}

interface FullStorage extends WritableStorage, DeletableStorage {}

class FileStorage implements FullStorage {
  read(path: string): string { return fs.readFileSync(path, 'utf-8'); }
  save(path: string, data: string): void { fs.writeFileSync(path, data); }
  delete(path: string): void { fs.unlinkSync(path); }
}

class ReadOnlyFileStorage implements ReadableStorage {
  read(path: string): string { return fs.readFileSync(path, 'utf-8'); }
  // No save or delete - not part of interface!
}

// Type-safe
function backupData(storage: WritableStorage, data: string) {
  storage.save('/backup/data.txt', data);
}

backupData(new FileStorage(), 'data'); // ✅ Works
// backupData(new ReadOnlyFileStorage(), 'data'); // ❌ Compile error
```

---

## 📋 LSP Rules

### Preconditions
Subclass cannot strengthen preconditions
```typescript
// Base: accepts any number
class Calculator {
  divide(a: number, b: number): number {
    if (b === 0) throw new Error('Cannot divide by zero');
    return a / b;
  }
}

// ❌ BAD: Subclass adds MORE restrictions
class StrictCalculator extends Calculator {
  divide(a: number, b: number): number {
    if (b === 0) throw new Error('Cannot divide by zero');
    if (a < 0 || b < 0) throw new Error('Only positive numbers'); // New restriction!
    return a / b;
  }
}
```

### Postconditions
Subclass cannot weaken postconditions
```typescript
// Base: guarantees positive result
class AbsoluteCalculator {
  calculate(n: number): number {
    return Math.abs(n); // Always returns positive
  }
}

// ❌ BAD: Subclass might return negative
class BrokenCalculator extends AbsoluteCalculator {
  calculate(n: number): number {
    return n; // Can return negative - weaker guarantee!
  }
}
```

---

## 📋 LSP Checklist

> [!check] Signs You're Following LSP
> - [ ] Subclasses don't throw unexpected exceptions
> - [ ] Subclasses don't return unexpected null/undefined
> - [ ] Subclasses maintain parent's invariants
> - [ ] Code using parent class works with any subclass

> [!warning] Signs You're Violating LSP
> - Subclass methods throw "Not implemented" exceptions
> - Using `instanceof` checks before calling methods
> - Subclass methods do nothing (empty implementations)
> - Unexpected behavior when using polymorphism

---

← [[Programming/Software Engineering/SOLID/_Index|Back to SOLID]]
