---
tags:
  - software-engineering
  - solid
  - isp
created: 2026-01-02
status: 🔴
---
# 🔌 Interface Segregation Principle (ISP)

> *"Clients should not be forced to depend on interfaces they do not use."* — Robert C. Martin

## 🎯 The Principle

No obligues a las clases a implementar métodos que no necesitan. Es mejor tener **muchas interfaces pequeñas y específicas** que una interfaz grande y general.

---

## ❌ Violating ISP: Fat Interface

```typescript
// ❌ BAD: One fat interface for all workers

interface Worker {
  work(): void;
  eat(): void;
  sleep(): void;
  attendMeeting(): void;
  writeReport(): void;
  manageTeam(): void;
}

class Developer implements Worker {
  work(): void { console.log('Writing code'); }
  eat(): void { console.log('Eating lunch'); }
  sleep(): void { console.log('Sleeping'); }
  attendMeeting(): void { console.log('In meeting'); }
  writeReport(): void { console.log('Writing report'); }
  
  // Developer doesn't manage team!
  manageTeam(): void {
    throw new Error('Not a manager'); // 💥 ISP violation
  }
}

class Robot implements Worker {
  work(): void { console.log('Assembling parts'); }
  
  // Robots don't eat or sleep!
  eat(): void { /* do nothing */ } // 💥 ISP violation
  sleep(): void { /* do nothing */ } // 💥 ISP violation
  attendMeeting(): void { /* do nothing */ }
  writeReport(): void { /* do nothing */ }
  manageTeam(): void { /* do nothing */ }
}
```

**Problems:**
- Classes forced to implement irrelevant methods
- Empty implementations or exceptions
- Changes to interface affect all implementers

---

## ✅ Following ISP: Segregated Interfaces

```typescript
// ✅ GOOD: Small, focused interfaces

interface Workable {
  work(): void;
}

interface Eatable {
  eat(): void;
}

interface Sleepable {
  sleep(): void;
}

interface Manageable {
  attendMeeting(): void;
  writeReport(): void;
}

interface TeamLeader {
  manageTeam(): void;
}

// Developer implements only what it needs
class Developer implements Workable, Eatable, Sleepable, Manageable {
  work(): void { console.log('Writing code'); }
  eat(): void { console.log('Eating lunch'); }
  sleep(): void { console.log('Sleeping'); }
  attendMeeting(): void { console.log('In meeting'); }
  writeReport(): void { console.log('Writing report'); }
}

// Manager adds team leadership
class Manager implements Workable, Eatable, Sleepable, Manageable, TeamLeader {
  work(): void { console.log('Planning'); }
  eat(): void { console.log('Business lunch'); }
  sleep(): void { console.log('Sleeping'); }
  attendMeeting(): void { console.log('Leading meeting'); }
  writeReport(): void { console.log('Executive report'); }
  manageTeam(): void { console.log('Managing team'); }
}

// Robot only implements what makes sense
class Robot implements Workable {
  work(): void { console.log('Assembling parts'); }
}
```

---

## 🔄 Real-World Example: Printer

### ❌ Fat Interface
```typescript
// ❌ BAD: All-in-one printer interface

interface Printer {
  print(document: Document): void;
  scan(document: Document): void;
  fax(document: Document): void;
  staple(document: Document): void;
  photocopy(document: Document): void;
}

// Basic printer can only print!
class BasicPrinter implements Printer {
  print(document: Document): void {
    console.log('Printing...');
  }
  
  // Forced to implement these! 💥
  scan(document: Document): void {
    throw new Error('Scan not supported');
  }
  
  fax(document: Document): void {
    throw new Error('Fax not supported');
  }
  
  staple(document: Document): void {
    throw new Error('Staple not supported');
  }
  
  photocopy(document: Document): void {
    throw new Error('Photocopy not supported');
  }
}
```

### ✅ Segregated Interfaces
```typescript
// ✅ GOOD: Separate interfaces for each capability

interface Printable {
  print(document: Document): void;
}

interface Scannable {
  scan(): Document;
}

interface Faxable {
  fax(document: Document, number: string): void;
}

interface Stapleable {
  staple(document: Document): void;
}

// Basic printer - only printing
class BasicPrinter implements Printable {
  print(document: Document): void {
    console.log('Printing...');
  }
}

// Scanner only
class Scanner implements Scannable {
  scan(): Document {
    return new Document('Scanned content');
  }
}

// Multi-function printer - combines interfaces
class MultiFunctionPrinter implements Printable, Scannable, Faxable {
  print(document: Document): void {
    console.log('Printing...');
  }
  
  scan(): Document {
    return new Document('Scanned content');
  }
  
  fax(document: Document, number: string): void {
    console.log(`Faxing to ${number}...`);
  }
}

// Enterprise printer - full featured
class EnterprisePrinter implements Printable, Scannable, Faxable, Stapleable {
  print(document: Document): void { /* ... */ }
  scan(): Document { /* ... */ }
  fax(document: Document, number: string): void { /* ... */ }
  staple(document: Document): void { /* ... */ }
}
```

---

## 🔄 Real-World Example: User Repository

### ❌ Fat Interface
```typescript
// ❌ BAD: One interface for all data access

interface UserRepository {
  findById(id: string): User;
  findByEmail(email: string): User;
  findAll(): User[];
  save(user: User): void;
  update(user: User): void;
  delete(id: string): void;
  findByRole(role: string): User[];
  findActive(): User[];
  count(): number;
  existsByEmail(email: string): boolean;
  bulkInsert(users: User[]): void;
  bulkDelete(ids: string[]): void;
}

// Read-only service doesn't need write methods!
class UserReportService {
  constructor(private repo: UserRepository) {}
  
  generateReport() {
    // Only uses findAll and count
    // But has access to delete, save, etc. - dangerous!
  }
}
```

### ✅ Segregated Interfaces
```typescript
// ✅ GOOD: Separate read and write concerns

interface UserReader {
  findById(id: string): User | null;
  findByEmail(email: string): User | null;
  findAll(): User[];
}

interface UserWriter {
  save(user: User): void;
  update(user: User): void;
  delete(id: string): void;
}

interface UserQueryable extends UserReader {
  findByRole(role: string): User[];
  findActive(): User[];
  count(): number;
  existsByEmail(email: string): boolean;
}

interface UserBulkOperations {
  bulkInsert(users: User[]): void;
  bulkDelete(ids: string[]): void;
}

// Full repository implements all
class UserRepository implements UserReader, UserWriter, UserQueryable, UserBulkOperations {
  // ... all implementations
}

// Report service only has read access - safe!
class UserReportService {
  constructor(private repo: UserQueryable) {}
  
  generateReport() {
    const users = this.repo.findAll();
    const count = this.repo.count();
    // Cannot call save, delete, etc. - compile error!
  }
}

// Admin service has full access
class UserAdminService {
  constructor(private repo: UserWriter & UserBulkOperations) {}
  
  purgeInactiveUsers(ids: string[]) {
    this.repo.bulkDelete(ids);
  }
}
```

---

## 🎯 Interface Composition

```typescript
// Small, focused interfaces
interface Identifiable {
  id: string;
}

interface Timestamped {
  createdAt: Date;
  updatedAt: Date;
}

interface Auditable {
  createdBy: string;
  updatedBy: string;
}

interface SoftDeletable {
  deletedAt?: Date;
  isDeleted: boolean;
}

// Compose for specific needs
interface User extends Identifiable, Timestamped, Auditable {
  email: string;
  name: string;
}

interface AuditLog extends Identifiable, Timestamped {
  action: string;
  details: string;
}

interface Post extends Identifiable, Timestamped, Auditable, SoftDeletable {
  title: string;
  content: string;
}
```

---

## 📋 ISP Checklist

> [!check] Signs You're Following ISP
> - [ ] Interfaces are small and focused (3-5 methods max)
> - [ ] Classes implement only methods they use
> - [ ] No empty implementations or exceptions for "not supported"
> - [ ] Interfaces are named by capability (Readable, Writable, etc.)

> [!warning] Signs You're Violating ISP
> - Large interfaces with 10+ methods
> - Methods that throw "Not implemented"
> - Empty method implementations
> - Classes implementing interfaces they only partially use
> - Interface changes causing many unrelated classes to change

---

← [[Programming/Software Engineering/SOLID/SOLID Principles Complete Guide|Back to SOLID]]
