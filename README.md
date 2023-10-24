# SCTSC Final Project

## Important notice

As a part of this project, **you won't** be given explicit code or rules that your library must follow, as long as it fills in the provided requirements and passes the acceptance criteria. The important thing is that library **needs** work, with as little bugs as possible. Every code example is just an example, you can, but don't have to use it (you won't get any extra points if you do :P). Make your code clean, understandable and maintainable.

## Assignment

Almost every web app today needs some kind of a form. Almost every framework has either integrated support or a third-party library for handling forms, form validations, submissions, etc. But what if don't want to use a framework but vanilla TypeScript?

As a part of this project, your job will be to create a small reactive form library called **Simple Forms**. Think of all the libraries you used over the time - what you liked, what you didn't like, etc. The idea is that Simple Forms should have a simple and intuitive API so that it's beginner friendly, but powerful enough for a more complex use cases. They should also be fully typed and documented.

### Getting started

To get started, create a main library folder called `simple-forms` (inside `src` folder). Inside, create `index.ts` file. Everything that library exposes (e.g. functions, classes, types, etc.) should be exported from that `index.ts` file. All library related code, like utilities, helpers, types, etc. should be within `simple-forms` folder. The idea behind this is that your library can be used simply by importing from one folder (file):

```ts
import { Form } from 'simple-forms';
```

### Requirements

Simple Forms should be an abstraction level of native HTML inputs. They should support _at least_ the following:

- text inputs (text, email, password, etc.)
- number inputs
- checkbox inputs
- radio inputs
- select inputs

Library should expose two levels of abstraction - form and control. **Control** should (two way) bind to a single input element, and should used for communication and/or manipulation of the bound element. **Form** should (two way) bind to multiple elements in an object structure, i.e. handle multiple Controls within itself.

#### Control

Control accepts one to three arguments. First and second arguments are mandatory. First argument is the selector or the element reference that Control should bind to. Second argument is the initial value of the input.

```ts
import { Control } from 'simple-forms';

const nameInput = document.getElementById('name-input');

const nameControl = Control(nameInput, ''); // <-- passed in an element reference
const ageControl = Control('#age-input', 18); // <-- passed in a selector string
```

> Note: Control can bind **only to one** element when using selector string.

Third, optional, argument is an array of validator functions. Each function passed in checks the validity of the current input. Validator function accepts a (current) control parameter and returns either a boolean `true` on success or an error object on failure. Error object should have a unique name of the validator for the key and any additional info for the value. If no additional info needed, simply use `true`. These return values will be later used by validity method on the control itself.

```ts
import { Control } from 'simple-forms';

const specialCharacterValidator = (control) => {
  return /[^a-zA-Z0-9\s]/.test(value) || {
    specialChar: true,
  };
}

const minLengthValidator = (min) => {
  return (control) => {
    if(control.value.length >= min) {
      return true;
    }

    return {
      minLength: {
        mustBe: min,
        currentLength: control.value.length,
      }
    };
  }
}

const passwordControl = Control('.js-password-input', '', [
  (control) => { 
    return !!control.value || {
      required: true,
    };
  }, // <-- inline validator function
  minLengthValidator(8), // <-- factory validator, i.e. accepts a param and returns a validator function
  specialCharValidator, // <-- validator function reference
]);
```

Value is valid only if it passes all the validators, i.e. if every validator returns `true`. Validators should trigger every time the value changes.

#### Control instance

> Note: You can use either functions or classes to create Control instances. It's up to you. In the examples, we'll be using functions.

When Control instance is created, e.g. by calling

```ts
const control = Control(...);
```

it should have ceratin properties on itself. Main property is `value`, which gets/sets current value of the control.

```ts
const control = Control('#name', 'John');

console.log(control.value) // John

control.value = 'Jimmy';

// now, the HTML element `#name` should show the value "Jimmy" as well

console.log(control.value); // Jimmy
```

Another property is `valid`, which should return if the the current value of the control is valid. This field goes in correlation with `errors` field. If field is valid, errors will be `null`. If it's invalid, errors will be the list of errors returned by the validators. If no validators are given to the control, `errors` will always be `null`.

```ts
import { Control } from 'simple-forms';

const specialCharacterValidator = (control) => {
  return /[^a-zA-Z0-9\s]/.test(value) || {
    specialChar: true,
  };
}

const minLengthValidator = (min) => {
  return (control) => {
    if(control.value.length >= min) {
      return true;
    }

    return {
      minLength: {
        mustBe: min,
        currentLength: control.value.length,
      }
    };
  }
}

const passwordControl = Control('.js-password-input', '', [
  (control) => { 
    return !!control.value || {
      required: true,
    };
  }, // <-- inline validator function
  minLengthValidator(8), // <-- factory validator, i.e. accepts a param and returns a validator function
  specialCharValidator, // <-- validator function reference
]);

console.log(passwordControl.valid); // false
console.log(passwordControl.errors); // order of errors is in the order of validators
/*
[
  { required: true },
  { minLength: { mustBe: 8, currentLength: 0 }},
  { specialChar: true },
]
*/

passwordControl.value = 'Password!23';

console.log(passwordControl.valid); // true;
console.log(passwordControl.errors) // null;
```

Control should also have a `reset()` method which would reset the control value to it's initial value.

```ts
const control = Control('#name', 'John');

console.log(control.value) // John

control.value = 'Jimmy';

console.log(control.value); // Jimmy

control.reset();

console.log(control.value) // John
```

In order to know when the value is changed, Control should have `onValueChange` callback field. Each callback should receive an object inside with the previous value and current value. **`onValueChange` should not trigger when control instance is created, i.e. when initial value is assigned to it.**

```ts
const control = Control('#name', 'John');

control.onValueChange(({ prev, value }) => {
  console.log('1st listener:', prev, value);
});

control.onValueChange(({ prev, value }) => {
  console.log('2nd listener:', prev, value);
});

control.value = 'WOW!';

/*
1st listener: John WOW!
2nd listener: John WOW!
*/
```

Last but not least, one of the most important features is that whenever the value of the input that Control is bound to changes, Control should reflect that value change, updates it's validity and notify listeners.

#### Form

Form is used as a syntactic sugar to make it easier to create form groups. For example:

```ts
import { Form } from 'simple-forms';

const form = Form({
  username: ['#username', ''],
  password: ['#password', '', [/* lsit of validators */]],
});
```

Form accepts one mandatory object argument. Keys are the names of each field and value is a tuple of up to three params. Params are one-to-one mapping of Control arguments, i.e. first is selector/element, second is initial value and the third is list of validators. Under the hood, Form should use Controls to track and update input state.

Also, instead of Control definition tuple, Form can accept Control directly. The logic stays the same.

```ts
import { Form, Control } from 'simple-forms';

const nameControl = Control('#name', 'John');

const form = Form({
  name: nameControl,
  username: ['#username', ''],
  password: ['#password', '', [/* lsit of validators */]],
});
```

#### Form instance

Form instance has similar properties to the Control instance. `value`, `valid` and `onValueChange` are the same. What Form has, but Control doesn't is the `controls` property. Controls property is the object ref to all the controls that Form has. Form is valid only if every field inside is valid.

```ts
const form = Form({
  username: ['#username', 'John'],
  password: ['#password', '', [/* lsit of validators */]],
});

console.log(form.value) // { username: 'John', password: '' }

form.onValueChange(({ prev, value }) => {
  console.log('Form changed:', prev, value);
});

form.controls.username.value = 'Mike';
/* 
The above will trigger the listener:

Form changed: { username: 'John', password: '' } { username: 'Mike', password: '' }
*/
```

### Testing out the library

Once you're done with the library, create a simple (or complex :P) form in the HTML to show how the library works, e.g. real time form validation, prepopulating form fields, etc.