# Reactive Forms vs. Signal Forms in Angular 22 — A Complete, Beginner‑Friendly Guide

> A step‑by‑step learning guide. We build the **same sign‑up form** twice — once with classic **Reactive Forms**, once with the new **Signal Forms** — and then learn how to **convert** one into the other.

---

## Table of Contents

- [Before We Start: What Changed in Angular 22?](#before-we-start-what-changed-in-angular-22)
- [The Form We Are Building](#the-form-we-are-building)
- [Two Big Ideas (Read This First)](#two-big-ideas-read-this-first)
- [Part 1 — The Reactive Form](#part-1--the-reactive-form)
  - [1.1 The TypeScript](#11-the-typescript)
  - [1.2 The HTML Template](#12-the-html-template)
  - [1.3 What Hurts Here](#13-what-hurts-here)
- [Part 2 — The Signal Form](#part-2--the-signal-form)
  - [2.1 The Model](#21-the-model)
  - [2.2 The Form and Its Validation Schema](#22-the-form-and-its-validation-schema)
  - [2.3 Async Email Validation](#23-async-email-validation)
  - [2.4 Cross‑Field: Confirmation Email Must Match](#24-cross-field-confirmation-email-must-match)
  - [2.5 Address: Both or Neither](#25-address-both-or-neither)
  - [2.6 Skills: A Dynamic Array](#26-skills-a-dynamic-array)
  - [2.7 The Full TypeScript](#27-the-full-typescript)
  - [2.8 The Full HTML Template](#28-the-full-html-template)
  - [2.9 Handling Submit](#29-handling-submit)
- [Part 3 — Converting a Reactive Form to a Signal Form](#part-3--converting-a-reactive-form-to-a-signal-form)
  - [3.1 The Five‑Step Recipe](#31-the-five-step-recipe)
  - [3.2 The Big Conversion Table](#32-the-big-conversion-table)
  - [3.3 Migrating Gradually (For Big Projects)](#33-migrating-gradually-for-big-projects)
- [Cheat Sheet](#cheat-sheet)
- [Final Thoughts](#final-thoughts)

---

## Before We Start: What Changed in Angular 22?

For years, Angular gave us two ways to build forms: **Template‑Driven Forms** (simple, but limited) and **Reactive Forms** (powerful, but verbose). Reactive Forms have been the standard choice for serious apps for a long time.

In **Angular 22 (released around June 2026)**, a third option became **stable and ready for production: Signal Forms.** They were experimental in v21 and matured into a fully supported feature in v22, alongside other "signal‑first" changes like the stable Resource API and `OnPush` becoming the default change detection strategy.

The key idea behind Signal Forms is simple:

> In Reactive Forms, your data lives in **two places**: your real application data on one side, and a separate `FormGroup` object on the other. You constantly keep them in sync.
>
> In Signal Forms, there is **one place**: your data *is* the form. You describe your data as a signal, and Angular builds the form directly from its shape.

Think of it like this:

- **Reactive Forms** are like a **photocopy** of an office document — you keep editing the copy and have to remember to update the original.
- **Signal Forms** are like a **live Google Doc** — whatever anyone types is instantly saved to the one true document. One source of truth, not two.

Everything in Signal Forms lives in the `@angular/forms/signals` package.

---

## The Form We Are Building

We will build a **sign‑up form** with these fields:

| Field | Rule |
| --- | --- |
| **Name** | Required |
| **Email** | Required, valid email, **and must be unique** (checked on the server — *async*) |
| **Confirmation Email** | Required, **must match** the Email field (*cross‑field*) |
| **Address → City** | City and Street must be **both filled or both empty** (*cross‑field*) |
| **Address → Street** | (same rule as City) |
| **Skills** | A **dynamic list** — zero, one, or many skills; each one must not be empty |

This single form touches every hard part of form building: basic validation, async validation, cross‑field validation, nested objects, and dynamic arrays. If you understand this form, you understand forms.

---

## Two Big Ideas (Read This First)

These two sentences explain almost everything that follows. Keep them in mind:

**Idea 1 — The shape of your data is the shape of your form.**
If your data has a nested object (`address`), the form automatically has a nested group. If your data has an array (`skills`), the form automatically has an array. You never build `FormGroup` or `FormArray` by hand in Signal Forms — you just write a normal JavaScript object, and Angular walks through it to build the form.

**Idea 2 — Everything is a signal, so you read it by calling it.**
In Signal Forms, a field is a function. To read its value or state, you *call* it: `form.email()` gives you the field state, and then `form.email().valid()`, `form.email().touched()`, and `form.email().errors()` give you live, reactive information. The UI updates by itself when these change.

---

## Part 1 — The Reactive Form

Let's build the form the classic way first. This is the version most Angular developers already know.

### 1.1 The TypeScript

```typescript
import { Component, inject } from '@angular/core';
import {
  FormBuilder,
  Validators,
  ReactiveFormsModule,
  AbstractControl,
  ValidationErrors,
  AsyncValidatorFn,
  FormArray,
} from '@angular/forms';
import { of, timer } from 'rxjs';
import { map, switchMap, catchError } from 'rxjs/operators';
import { AccountService } from './account.service';

// --- A group-level validator: confirmation email must match email ---
function emailsMatch(group: AbstractControl): ValidationErrors | null {
  const email = group.get('email')?.value;
  const confirm = group.get('confirmEmail')?.value;
  // Don't complain before the user has typed the confirmation
  if (!confirm) return null;
  return email === confirm ? null : { emailMismatch: true };
}

// --- A group-level validator: city + street must be both or neither ---
function addressComplete(group: AbstractControl): ValidationErrors | null {
  const city = group.get('city')?.value;
  const street = group.get('street')?.value;
  const onlyOneFilled = !!city !== !!street; // true if exactly one is filled
  return onlyOneFilled ? { incompleteAddress: true } : null;
}

// --- An async validator: email must be unique on the server ---
function uniqueEmail(api: AccountService): AsyncValidatorFn {
  return (control: AbstractControl) => {
    if (!control.value) return of(null);
    // We debounce by waiting 400ms before calling the server
    return timer(400).pipe(
      switchMap(() => api.isEmailTaken(control.value)),
      map((taken) => (taken ? { emailTaken: true } : null)),
      catchError(() => of(null)), // if the server fails, don't block the user
    );
  };
}

@Component({
  selector: 'app-signup',
  imports: [ReactiveFormsModule],
  templateUrl: './signup.html',
})
export class Signup {
  private fb = inject(FormBuilder);
  private api = inject(AccountService);

  signupForm = this.fb.group(
    {
      name: ['', Validators.required],
      email: [
        '',
        [Validators.required, Validators.email],
        [uniqueEmail(this.api)], // async validators go in the third slot
      ],
      confirmEmail: ['', Validators.required],
      address: this.fb.group(
        {
          city: [''],
          street: [''],
        },
        { validators: addressComplete }, // group-level validator
      ),
      skills: this.fb.array<string>([]), // starts empty
    },
    { validators: emailsMatch }, // top-level group validator
  );

  // We need a getter just to access the array in a typed way
  get skills(): FormArray {
    return this.signupForm.get('skills') as FormArray;
  }

  addSkill(): void {
    this.skills.push(this.fb.control('', Validators.required));
  }

  removeSkill(index: number): void {
    this.skills.removeAt(index);
  }

  onSubmit(): void {
    if (this.signupForm.valid) {
      console.log(this.signupForm.value);
    } else {
      this.signupForm.markAllAsTouched();
    }
  }
}
```

### 1.2 The HTML Template

```html
<form [formGroup]="signupForm" (ngSubmit)="onSubmit()">
  <!-- Name -->
  <label>
    Name
    <input formControlName="name" />
  </label>
  @if (signupForm.get('name')?.touched && signupForm.get('name')?.invalid) {
    <small class="error">Name is required.</small>
  }

  <!-- Email -->
  <label>
    Email
    <input type="email" formControlName="email" />
  </label>
  @if (signupForm.get('email')?.touched) {
    @if (signupForm.get('email')?.hasError('required')) {
      <small class="error">Email is required.</small>
    }
    @if (signupForm.get('email')?.hasError('email')) {
      <small class="error">Please enter a valid email.</small>
    }
    @if (signupForm.get('email')?.pending) {
      <small class="info">Checking availability…</small>
    }
    @if (signupForm.get('email')?.hasError('emailTaken')) {
      <small class="error">This email is already registered.</small>
    }
  }

  <!-- Confirmation Email -->
  <label>
    Confirm Email
    <input type="email" formControlName="confirmEmail" />
  </label>
  @if (signupForm.touched && signupForm.hasError('emailMismatch')) {
    <small class="error">The two emails do not match.</small>
  }

  <!-- Address (nested group) -->
  <fieldset formGroupName="address">
    <legend>Address</legend>
    <label>
      City
      <input formControlName="city" />
    </label>
    <label>
      Street
      <input formControlName="street" />
    </label>
    @if (signupForm.get('address')?.hasError('incompleteAddress')) {
      <small class="error">Please fill both City and Street, or leave both empty.</small>
    }
  </fieldset>

  <!-- Skills (dynamic array) -->
  <fieldset formArrayName="skills">
    <legend>Skills</legend>
    @for (skill of skills.controls; track $index) {
      <div>
        <input [formControlName]="$index" />
        <button type="button" (click)="removeSkill($index)">Remove</button>
      </div>
    }
    <button type="button" (click)="addSkill()">+ Add skill</button>
  </fieldset>

  <button type="submit">Sign Up</button>
</form>
```

### 1.3 What Hurts Here

Reactive Forms work, and millions of apps run on them. But notice the friction:

1. **Two models to keep in sync.** Your real data and the `FormGroup` are different things.
2. **Cross‑field validators live far from the fields.** The "emails must match" rule sits on the whole group, away from the `confirmEmail` field it really belongs to. Finding the error later (`signupForm.hasError('emailMismatch')`) is awkward.
3. **Arrays are heavy.** You need a `FormArray`, a `getter`, type casting (`as FormArray`), `push`, `removeAt`, plus `formArrayName` and `[formControlName]="$index"` in the template.
4. **Async validation is fiddly.** You manage debouncing, RxJS operators, and pending state yourself.
5. **Reading state is verbose.** `signupForm.get('email')?.hasError('required')` everywhere.

Now let's see how Signal Forms makes each of these simpler.

---

## Part 2 — The Signal Form

We'll build the exact same form, piece by piece, then put it all together.

### 2.1 The Model

In Signal Forms, you start by describing your data as a plain signal. The form's structure comes straight from this shape — nested object for the address, array for the skills.

```typescript
import { signal } from '@angular/core';

interface SignupData {
  name: string;
  email: string;
  confirmEmail: string;
  address: {
    city: string;
    street: string;
  };
  skills: string[];
}

signupModel = signal<SignupData>({
  name: '',
  email: '',
  confirmEmail: '',
  address: {
    city: '',
    street: '',
  },
  skills: [], // can be empty
});
```

That's it. No `FormGroup`, no `FormArray`. Just data.

### 2.2 The Form and Its Validation Schema

You create the form by passing the model to `form()`. The **second argument is a schema function** where all your validation rules live. The schema receives a `path` object that points to each field.

```typescript
import { form, required, email, minLength } from '@angular/forms/signals';

signupForm = form(this.signupModel, (path) => {
  required(path.name, { message: 'Name is required' });

  required(path.email, { message: 'Email is required' });
  email(path.email, { message: 'Please enter a valid email' });

  required(path.confirmEmail, { message: 'Please confirm your email' });

  // (async, cross-field, address, and skills rules added below)
});
```

The built‑in validators are exactly what you'd expect: `required()`, `email()`, `min()`, `max()`, `minLength()`, `maxLength()`, and `pattern()`. Every validator accepts an options object so you can set a custom `message`.

### 2.3 Async Email Validation

To check that the email is unique on the server, Signal Forms gives us `validateHttp()`. It wraps an HTTP request and turns the response into a validation error. You only provide three things: how to build the request, what to do on success, and what to do on error.

```typescript
import { validateHttp } from '@angular/forms/signals';

// inside the schema function:
validateHttp(path.email, {
  // Runs automatically when the email value changes
  request: (ctx) => `/api/users/check-email?email=${ctx.value()}`,
  onSuccess: (response: { isTaken: boolean }) =>
    response.isTaken
      ? { kind: 'emailTaken', message: 'This email is already registered' }
      : undefined,
  onError: () => ({
    kind: 'emailCheckFailed',
    message: 'Could not verify the email. Please try again.',
  }),
});
```

Two things worth knowing:

- **Async validation only runs after the synchronous rules pass.** So Angular won't waste a server call on an empty or malformed email — `required()` and `email()` run first.
- **While the request is in flight, the field's `pending()` signal is `true`,** so you can show a "Checking…" message with no extra work.

For anything beyond a simple HTTP call (WebSockets, IndexedDB, custom caching), there is a lower‑level `validateAsync()`. For most apps, `validateHttp()` is all you need.

### 2.4 Cross‑Field: Confirmation Email Must Match

This is where Signal Forms really shines compared to Reactive Forms. We use `validate()` and place the rule **right on the field that should show the error** — `confirmEmail`. Inside, `value()` is the current field, and `valueOf(...)` reads any other field.

```typescript
import { validate } from '@angular/forms/signals';

// inside the schema function:
validate(path.confirmEmail, ({ value, valueOf, stateOf }) => {
  // Don't show a mismatch error until the user has touched the email field
  if (!stateOf(path.email).touched()) {
    return null;
  }
  if (value() !== valueOf(path.email)) {
    return { kind: 'emailMismatch', message: 'The two emails do not match' };
  }
  return null; // null means valid
});
```

- `valueOf(path.email)` reads another field's **value**.
- `stateOf(path.email)` reads another field's **state** — like `touched()`, `dirty()`, or `invalid()`. We use it here so we don't flag a mismatch before the user has even started typing the email.

> ⚠️ **Watch out for infinite loops.** Inside a validator, never read the *validity* of a parent field that depends on this validator. A parent's validity depends on its children's validity — and your validator is one of those children — so reading it creates an endless cycle. Reading another field's **value** is always safe; reading the **validity of a field above you** is the dangerous part.

### 2.5 Address: Both or Neither

We want City and Street to be filled together or left empty together. The rule looks at two fields inside the `address` group, but we want the error to appear on a specific field. For that we use `validateTree()`, which works like `validate()` but lets you aim the error at a chosen sub‑field.

```typescript
import { validateTree } from '@angular/forms/signals';

// inside the schema function:
validateTree(path.address, ({ value, fieldTreeOf }) => {
  const { city, street } = value();
  const onlyOneFilled = !!city !== !!street; // exactly one is filled

  return onlyOneFilled
    ? {
        fieldTree: fieldTreeOf(path.address.street), // put the error on the street field
        kind: 'incompleteAddress',
        message: 'Please fill both City and Street, or leave both empty',
      }
    : undefined;
});
```

The difference in one line:

- `validate(field, …)` → the error lands on **that** field.
- `validateTree(group, …)` → you look at a whole group and **aim** the error wherever you want with `fieldTree: fieldTreeOf(path.x)`.

### 2.6 Skills: A Dynamic Array

Remember Idea 1: an array in your data is automatically an array in your form. So `skills` is just a normal JavaScript array of strings.

To validate **every** item the same way, use `applyEach()`. To enforce a rule on the **whole** array (for example, "at least one skill"), use `validate()` on the array itself.

```typescript
import { applyEach, validate, required } from '@angular/forms/signals';

// inside the schema function:

// Each skill must not be empty
applyEach(path.skills, (skillPath) => {
  required(skillPath, { message: 'A skill cannot be empty' });
});

// (Optional) require at least one skill
validate(path.skills, ({ value }) =>
  value().length === 0
    ? { kind: 'noSkills', message: 'Please add at least one skill' }
    : null,
);
```

> ⚠️ **The `applyEach` callback takes only ONE argument** (the item's path), not an index. Passing a second `index` argument is an error. If you truly need the index, read it inside a `validate()` using the `index` helper from the field context.

Adding and removing items is just normal array work on the signal — no special form methods:

```typescript
addSkill(): void {
  this.signupModel.update((model) => ({
    ...model,
    skills: [...model.skills, ''], // add an empty string
  }));
}

removeSkill(index: number): void {
  this.signupModel.update((model) => ({
    ...model,
    skills: model.skills.filter((_, i) => i !== index), // remove by index
  }));
}
```

To remove an item, you simply remove it from the data array. There is no `removeAt()` — you work with the data, and the form follows.

### 2.7 The Full TypeScript

Here is everything from Part 2 assembled into one component.

```typescript
import { Component, signal } from '@angular/core';
import {
  form,
  FormField,
  submit,
  required,
  email,
  validate,
  validateTree,
  validateHttp,
  applyEach,
} from '@angular/forms/signals';
import { AccountService } from './account.service';
import { inject } from '@angular/core';

interface SignupData {
  name: string;
  email: string;
  confirmEmail: string;
  address: { city: string; street: string };
  skills: string[];
}

@Component({
  selector: 'app-signup',
  imports: [FormField], // note: FormField, not ReactiveFormsModule
  templateUrl: './signup.html',
})
export class Signup {
  private api = inject(AccountService);

  // 1. The data model
  signupModel = signal<SignupData>({
    name: '',
    email: '',
    confirmEmail: '',
    address: { city: '', street: '' },
    skills: [],
  });

  // 2. The form + all validation in one place
  signupForm = form(this.signupModel, (path) => {
    required(path.name, { message: 'Name is required' });

    required(path.email, { message: 'Email is required' });
    email(path.email, { message: 'Please enter a valid email' });

    // Async: must be unique
    validateHttp(path.email, {
      request: (ctx) => `/api/users/check-email?email=${ctx.value()}`,
      onSuccess: (response: { isTaken: boolean }) =>
        response.isTaken
          ? { kind: 'emailTaken', message: 'This email is already registered' }
          : undefined,
      onError: () => ({
        kind: 'emailCheckFailed',
        message: 'Could not verify the email. Please try again.',
      }),
    });

    required(path.confirmEmail, { message: 'Please confirm your email' });

    // Cross-field: confirmation must match
    validate(path.confirmEmail, ({ value, valueOf, stateOf }) => {
      if (!stateOf(path.email).touched()) return null;
      return value() !== valueOf(path.email)
        ? { kind: 'emailMismatch', message: 'The two emails do not match' }
        : null;
    });

    // Cross-field: address both or neither
    validateTree(path.address, ({ value, fieldTreeOf }) => {
      const { city, street } = value();
      const onlyOneFilled = !!city !== !!street;
      return onlyOneFilled
        ? {
            fieldTree: fieldTreeOf(path.address.street),
            kind: 'incompleteAddress',
            message: 'Please fill both City and Street, or leave both empty',
          }
        : undefined;
    });

    // Dynamic array: each skill required
    applyEach(path.skills, (skillPath) => {
      required(skillPath, { message: 'A skill cannot be empty' });
    });
  });

  addSkill(): void {
    this.signupModel.update((m) => ({ ...m, skills: [...m.skills, ''] }));
  }

  removeSkill(index: number): void {
    this.signupModel.update((m) => ({
      ...m,
      skills: m.skills.filter((_, i) => i !== index),
    }));
  }

  onSubmit(): void {
    submit(this.signupForm, async () => {
      const data = this.signupModel(); // final, valid data
      await this.api.createAccount(data);
      return null; // null = success; or return server-side field errors
    });
  }
}
```

### 2.8 The Full HTML Template

```html
<form (ngSubmit)="onSubmit()">
  <!-- Name -->
  <label>
    Name
    <input [formField]="signupForm.name" />
  </label>
  @if (signupForm.name().touched() && signupForm.name().invalid()) {
    @for (e of signupForm.name().errors(); track e.kind) {
      <small class="error">{{ e.message }}</small>
    }
  }

  <!-- Email -->
  <label>
    Email
    <input type="email" [formField]="signupForm.email" />
  </label>
  @if (signupForm.email().pending()) {
    <small class="info">Checking availability…</small>
  }
  @if (signupForm.email().touched()) {
    @for (e of signupForm.email().errors(); track e.kind) {
      <small class="error">{{ e.message }}</small>
    }
  }

  <!-- Confirmation Email -->
  <label>
    Confirm Email
    <input type="email" [formField]="signupForm.confirmEmail" />
  </label>
  @if (signupForm.confirmEmail().touched()) {
    @for (e of signupForm.confirmEmail().errors(); track e.kind) {
      <small class="error">{{ e.message }}</small>
    }
  }

  <!-- Address (nested object — no special directive needed) -->
  <fieldset>
    <legend>Address</legend>
    <label>
      City
      <input [formField]="signupForm.address.city" />
    </label>
    <label>
      Street
      <input [formField]="signupForm.address.street" />
    </label>
    @for (e of signupForm.address.street().errors(); track e.kind) {
      <small class="error">{{ e.message }}</small>
    }
  </fieldset>

  <!-- Skills (dynamic array — just a @for loop) -->
  <fieldset>
    <legend>Skills</legend>
    @for (skill of signupForm.skills; track $index) {
      <div>
        <input [formField]="signupForm.skills[$index]" />
        <button type="button" (click)="removeSkill($index)">Remove</button>
        @for (e of signupForm.skills[$index]().errors(); track e.kind) {
          <small class="error">{{ e.message }}</small>
        }
      </div>
    }
    <button type="button" (click)="addSkill()">+ Add skill</button>
  </fieldset>

  <button type="submit" [disabled]="signupForm().invalid()">Sign Up</button>
</form>
```

Notice what disappeared compared to Part 1:

- No `[formGroup]`, no `formGroupName`, no `formArrayName`.
- Nested fields are reached with a simple dot: `signupForm.address.city`.
- Array items are reached by index: `signupForm.skills[$index]`.
- The submit button can disable itself reactively with `signupForm().invalid()`.

### 2.9 Handling Submit

The `submit()` helper runs your action **only when the form is valid**, and it can even take server‑side errors you return and attach them to the right fields automatically.

```typescript
onSubmit(): void {
  submit(this.signupForm, async () => {
    const data = this.signupModel();
    const result = await this.api.createAccount(data);
    // Return null on success.
    // Or return errors to display them on specific fields.
    return result.ok ? null : result.fieldErrors;
  });
}
```

---

## Part 3 — Converting a Reactive Form to a Signal Form

Now the practical part: you already have a Reactive Form, and you want to move it to Signal Forms. Here is a repeatable recipe.

### 3.1 The Five‑Step Recipe

**Step 1 — Turn the `FormGroup` into a model signal.**
Stop listing controls. Instead, write the *shape* of your data with starting values. Nested groups become nested objects; `FormArray`s become plain arrays.

```typescript
// Before
this.fb.group({ name: [''], address: this.fb.group({ city: [''] }) })

// After
signal({ name: '', address: { city: '' } })
```

**Step 2 — Move validators out of the controls and into the schema function.**
Each `Validators.x` becomes a schema rule:

| Reactive | Signal Forms |
| --- | --- |
| `Validators.required` | `required(path.field)` |
| `Validators.email` | `email(path.field)` |
| `Validators.minLength(3)` | `minLength(path.field, 3)` |
| `Validators.min(18)` | `min(path.field, 18)` |
| `Validators.pattern(re)` | `pattern(path.field, re)` |

**Step 3 — Rewrite cross‑field validators.**
Group‑level validators (like "emails match" or "address complete") move into `validate()` or `validateTree()`, placed near the field that should display the error. Use `valueOf()` to read other fields instead of `group.get('...')`.

**Step 4 — Rewrite async validators.**
Replace your RxJS `AsyncValidatorFn` with `validateHttp()` (or `validateAsync()` for non‑HTTP cases). You no longer manage debouncing or pending state by hand.

**Step 5 — Update the template.**
Swap the directives using the conversion table below, and remember to *call* signals with `()` when reading state.

### 3.2 The Big Conversion Table

| Concept | Reactive Form | Signal Form |
| --- | --- | --- |
| Define the form | `fb.group({ ... })` | `form(model, (path) => { ... })` |
| Define the data | controls inside the group | a `signal({ ... })` object |
| Nested group | `fb.group({ ... })` inside | a nested object in the model |
| Bind a field (HTML) | `formControlName="name"` | `[formField]="form.name"` |
| Bind a nested field (HTML) | `formGroupName="address"` wrapper | `[formField]="form.address.city"` |
| Dynamic list | `FormArray` + getter + casting | a plain JS array in the model |
| Bind a list (HTML) | `formArrayName` + `[formControlName]="$index"` | `@for` + `[formField]="form.skills[$index]"` |
| Add to list | `array.push(fb.control(''))` | `model.update(m => ({ ...m, skills: [...m.skills, ''] }))` |
| Remove from list | `array.removeAt(i)` | `skills.filter((_, idx) => idx !== i)` |
| List item validation | per control | `applyEach(path.skills, ...)` |
| Cross‑field rule | group‑level validator | `validate()` / `validateTree()` |
| Read another field | `group.get('email')?.value` | `valueOf(path.email)` |
| Async validation | `AsyncValidatorFn` + RxJS | `validateHttp()` / `validateAsync()` |
| Read a value | `form.get('email')?.value` | `model().email` |
| Is the field valid? | `form.get('email')?.valid` | `form.email().valid()` |
| Get errors | `form.get('email')?.errors` | `form.email().errors()` |
| Was it touched? | `form.get('email')?.touched` | `form.email().touched()` |
| Pending (async)? | `form.get('email')?.pending` | `form.email().pending()` |
| Whole form valid? | `form.valid` | `form().valid()` |
| Submit | `if (form.valid) { ... }` | `submit(form, async () => { ... })` |
| Module to import | `ReactiveFormsModule` | `FormField` (and helpers) |

### 3.3 Migrating Gradually (For Big Projects)

You do **not** have to convert everything at once. A practical strategy for a large codebase:

1. **First, get to a supported Angular version.** Separate the *version upgrade* from the *pattern modernization*. Reach Angular 22 on a working build before changing how forms work.
2. **Then migrate forms one at a time.** Angular provides interop so Reactive Forms and Signal Forms can live side by side during the transition. Start with a small, low‑risk form to learn the patterns, then move to bigger ones.
3. **Reuse schemas.** As you go, extract shared rules (like an `addressSchema`) with `schema()` and apply them with `apply()`. This pays off quickly across many forms.

---

## Cheat Sheet

Keep this near you while you build.

**Imports** (from `@angular/forms/signals`):
`form`, `FormField`, `submit`, `required`, `email`, `min`, `max`, `minLength`, `maxLength`, `pattern`, `validate`, `validateTree`, `validateHttp`, `validateAsync`, `applyEach`, `apply`, `schema`, `disabled`, `hidden`, `readonly`, `debounce`.

**The core pattern:**

```typescript
model = signal({ /* your data shape */ });

myForm = form(this.model, (path) => {
  required(path.someField);
  validate(path.otherField, ({ value, valueOf }) => /* return error or null */);
});
```

**Reading state (always call the signal):**

```text
myForm.field()            → the field state
myForm.field().value()    → current value
myForm.field().valid()    → boolean
myForm.field().touched()  → boolean
myForm.field().pending()  → boolean (async in progress)
myForm.field().errors()   → array of { kind, message }
myForm().valid()          → whole form validity
```

**Three validation tools:**

- `required`, `email`, `minLength`, … → simple, single‑field rules.
- `validate(field, ctx => …)` → custom rule on one field; can read others via `valueOf`.
- `validateTree(group, ctx => …)` → rule across a group; aim the error with `fieldTree: fieldTreeOf(path.x)`.
- `validateHttp(field, { request, onSuccess, onError })` → server‑backed async rule.

**Two array tools:**

- `applyEach(path.list, itemPath => …)` → same rule on every item (callback takes **one** argument).
- `validate(path.list, ctx => …)` → a rule on the whole array (e.g., "at least one").

---

## Final Thoughts

The whole story of Signal Forms fits in one sentence: **you work with your data, and the form keeps itself in sync.** Once you internalize that, the rest is just learning the small set of helper functions in the cheat sheet above.

- Reactive Forms keep your data and your form as two separate things you must connect.
- Signal Forms make your data *be* the form, expose everything as signals, and let validation live right next to the fields it protects — including async checks, cross‑field rules, nested objects, and dynamic arrays.

If you learned forms with Reactive Forms, you already understand the concepts (validators, dirty/touched, valid/invalid). Signal Forms just give you a cleaner, more modern way to express them. Build the sign‑up form above once by hand, and the pattern will stick.

Happy building. 🚀
