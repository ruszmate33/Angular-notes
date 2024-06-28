# Angular complete guide notes

## Create a project

npm start project-name-like-this

## Install the dependencies

npm install

## VSCode extensions

Angular Essentials
Angular Language Service

### This is how the app gets rendered

```html
<body>
  <app-root></app-root>
</body>
```

- in main.ts where the magic happens

```ts
import { bootstrapApplication } from "@angular/platform-browser";
import { appConfig } from "./app/app.config";
import { AppComponent } from "./app/app.component";

bootstrapApplication(AppComponent, appConfig).catch((err) =>
  console.error(err)
);
```

- the `AppComponent`

```ts
import { Component } from "@angular/core";
import { RouterOutlet } from "@angular/router";

@Component({
  selector: "app-root",
  standalone: true,
  imports: [RouterOutlet],
  templateUrl: "./app.component.html",
  styleUrl: "./app.component.css",
})
export class AppComponent {
  title = "first-angular-app";
}
```

- uses the `@Component` decorator to attach some metadata
- `selector: 'app-root'` this defines that it comes here in the html `<app-root></app-root>` with the markup
- the markup is stored at `templateUrl` at `'./app.component.html'`

## Create a new `header` component

- src/app/header.component.ts
- where {name}{type}.{extension}

- start with exporting the class

```ts
import { Component } from "@angular/core";

@Component({
  selector: "app-header",
  template: "<h1>Hello, World</h1>",
})
export class HeaderComponent {}
```

export class {identfier}{jobOfTheClass}

- When using Angular, you'll often define classes which are NEVER instantiated by you!
- You never call `new SomeComponent()` anywhere in your code.

- for `selector`, the convention is to define tags with at least two words to avoid clash
- use either `template` or if bigger, the `templateUrl`

```ts
template: "<h1>Hello, World</h1>";
OR;
templateUrl: "./header.component.html";
```

## Template (markup)

- `header.component.html` (name.type.html)

```html
<header>
  <h1>Easy task</h1>
</header>
```

## Standalone components

- set `standalone` to true
- it used to be all module based components
- the new way is standalone

## Use the new component

- you only call `bootstrapApplication` once with the main `AppComponent` and for all child components you call them with your main `AppComponent`
- you will need to register the new component on the component tht is gonna use it, `AppComponent`

```ts
import { HeaderComponent } from './header.component';

@Component({
  ...
  imports: [RouterOutlet, HeaderComponent],
 ...
})
export class AppComponent {
  title = 'first-angular-app';
}
```

- put the tags at `app.component.html`

```html
<app-header></app-header>
<h1>Hello, {{ title }}</h1>
```

## Style

- create `header.component.css`
- link the style at the component

```ts
@Component({
  ...
  styleUrl: './header.component.css'
})
export class HeaderComponent {}
```

- alternatively with `styleUrls: ['./header.component.css]`

## Assets

- add an `assets` folder in src/app
- configure it at `angular.json`

```json
not
"assets": [
              {
                "glob": "**/*",
                "input": "public"
              }
            ],
```

but add `"src/assets"` to it
also `"src/favicon.ico"`

```json
"assets": [
          "src/favicon.ico",
          "src/assets",
              {
                "glob": "**/*",
                "input": "public"
              }
            ],
```

## Create components on the CLI

- subfolders like `app / header`
- `ng generate component {nameOfCompont} {path}` or `ng g c`
- `ng g c`

## Not nested components

```html
<app-header></app-header> <app-user></app-user>
```

is equivalent to

```html
<app-header /> <app-user />
```

## Put data into the component

```ts
export class UserComponent {
  selectedUser = DUMMY_USERS[randomIndex];
}
```

- this makes `selectedUser` available on the template

```html
<span> {{ selectedUser.name }} </span>
```

### Property binding syntax

- you could do this:

```html
<img src="{{ selectedUser.avatar }}" />
```

- but rather this

```html
<img [src]="selectedUser.avatar" />
```

- for the correct path

```html
<img [src]="'assets/users/' + selectedUser.avatar" [alt]="selectedUser.name" />
```

### Using getters for computed values

```ts
export class UserComponent {
  selectedUser = DUMMY_USERS[randomIndex];

  get imagePath() {
    return "assets/users/" + this.selectedUser.avatar;
  }
}
```

- use it in the template, as a PROPERTY, dont call it

```html
<img [src]="imagePath" [alt]="selectedUser.name" />
```

### Listening to events with event binding

```html
<button (click)="onSelectUser()">...</button>
```

```ts
export class UserComponent {
  ...
  onSelectUser() {
    consolo.log('Clicked')
  }
}
```

### Managing state and changing data

```ts
export class UserComponent {
 ...
  onSelectUser() {
    console.log('Clicked')
    const randomIndex = Math.floor(Math.random() * DUMMY_USERS.length);
    this.selectedUser = DUMMY_USERS[randomIndex]
  }
}
```

### Signals

- this notifies Ng about the change and it update everywhere with

```ts
// import
import { Component, signal } from "@angular/core";

const randomIndex = Math.floor(Math.random() * DUMMY_USERS.length);

export class UserComponent {
  // initialize the signal
  selectedUser = signal(DUMMY_USERS[randomIndex]);

  onSelectUser() {
    ...
    // update the signal
    this.selectedUser.set(DUMMY_USERS[randomIndex])
  }
}
```

- in the template we access the signal value
- not like this

```html
<span> {{ selectedUser.name }} </span>
```

- but call the signal value as a function, to make the sig

```html
<div>
  <button (click)="onSelectUser()">
    <img [src]="imagePath" [alt]="selectedUser().name" />
    <span> {{ selectedUser().name }} </span>
  </button>
</div>
```

- this then executes the signals read function, that gives you the value stored in it
- Ng sets up a subscription behind the scene (tracking it)
- very much different to `zone` mechanism
- with Signals it does not need to check every possible component for changes
- signals is a bit more verbose

#### Computed value with signals

- previously with a getter

```ts
export class UserComponent {
  ...
  get imagePath() {
    return 'assets/users/' + this.selectedUser.avatar;
  }
```

- now with a `computed`

```ts
export class UserComponent {
  ...
  imagePath = computed(() => 'assets/users/' + this.selectedUser().avatar)
```

- also sets up a subscription, only recomputes on demand
- computed also returns a signal
- need to do it in the template as well: `[src]="imagePath"` -> `[src]="imagePath()"`

```html
<button (click)="onSelectUser()">
  <img [src]="imagePath()" [alt]="selectedUser().name" />
</button>
```

## Reusing mulitple components

- we could repeat the `<li><app-user /></li>` in the template

```html
<app-header></app-header>

<main>
  <ul id="users">
    <li><app-user /></li>
    <li><app-user /></li>
    <li><app-user /></li>
  </ul>
</main>
```

### Defining component inputs

- eg. mark "avatar" as a set-table property from outside

- user.component.ts

```ts
import { Input } from '@angular/core'

@Component({
  ...
})
export class UserComponent {
  @Input() avatar!: string;
}
```

- with `avatar!: string` we declare that it will definately be set from the outside

- app.component.ts

```ts
import { DUMMY_USERS } from "./dummy-users";
@Component({})
export class AppComponent {
  users = DUMMY_USERS;
}
```

- app.component.html

```html
<li><app-user [avatar]="users[0].avatar" /></li>
<li><app-user [avatar]="users[1].avatar" /></li>
```

### Required and optional inputs

```ts
@Input({ required: true }) avatar!: string;
```

### User input signals
- this is for properties: `@Inputs({ required: true }) avatar!: string`, a decorator
- this is for accepting signals as inputs: `avatar = input<string>();`, a function
- you can pass an empty value for default: `avatar = input('')`
- or just tell the generic type `avatar = input<string>();`
- mark it as required with `avatar = input.required<string>()`
  - if required, you can't set a default value: `avatar = input.required('')` errors



```ts
import { input } from '@angular/core'

export class UserComponent {
  avatar = input<string>();
  name = input<string>();
}
```
#### Setting input signals
- outside the class you still set the component with the same syntax at app.component.html
- the value does not have to be a signal, also works with signals
```ts
<li><app-user [avatar]="users[0].avatar" [name]="users[0].name"/></li>
```
- the users is a normal array, not wrapped in a signal
```ts
export class AppComponent {
  ...
  users = DUMMY_USERS;
}
```
#### Use input signals
- you need to use the input signals as signals in the template
- user.component.html `[alt]="name()"` and `<span>{{ name() }}</span>` and `[src]="imagePath()"`
- instead of the getter in `user.component.ts` use a computed function
```ts
get imagePath() {
    return 'assets/users/' + this.avatar;
  }
```
to
```ts
imagePath computed(() => {
  return 'assets/users/' + this.avatar();
});
```
- this only recomputes `imagePath` is the `avatar` signal has changed

#### Can not change the input signal from inside the class
- they are read only, can't be changed from inside the class where they are registered
- only from outside
```ts
export class UserComponent {
  ...
  onSelectUser() {
    this.avatar.set(...) // this errors
  }
}
```

## Working with Outputs and emitting data
- we need to create a custom event that UserComponent can emit when it was clicked
- eventemitter emits the custom event to any parent component that is interested

```ts
import { Output, EventEmitter } from '@angular/core'
export class UserComponent {
  ...
  @Input({ required: true }) id!: string;
  // add a descriptive name to it
  @Output() select = new EventEmitter();

  onSelectUser() {
    this.select.emit(this.id);
  }
}
```
- add `id` in app.component.html
- add event binding for `select`
- use `$event` to access the emitted value by the event it is listening for
```html
<li>
  <app-user 
    [id]="users[0].id" 
    [avatar]="users[0].avatar" 
    [name]="users[0].name"
    (select)="onSelectUser($event)"
    />
  </li>
```
- add a method to app.component.ts
```ts
export class AppComponent {
  ...
  onSelectUser(id: string) {
    console.log('Selected user with id' + id);
  }
}
```