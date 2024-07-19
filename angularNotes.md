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
import { input } from "@angular/core";

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

### Modern approach to create Outputs

```ts
import { output } from "@angular/core";

// old way
// @Output() select = new EventEmitter();

// new way
select = output<string>();
```

- this would be more consistent with the input function (that creates signals, but this does not)

```ts
id = input.required<string>();
select = output<string>();
```

### Adding extra type information to EventEmitter

```ts
@Output() select = new EventEmitter<string>()  // defined that a string will be emitted
```

#### Exercise:

- create a component for the task of the user
- you click a usercomponent and it open a "Tasks" component
- right now will receive and output the name of the selected user

```ts
DUMMY_USERS.find((user) => user.id === selectedUserId);
```

- we get the information, which user is selected in the appcomponent
- it should be added in the appcomponent (template) eg. `<app-tasks name=...>`

```sh
ng generate component tasks
```

- standalone

- Component: put data into it

```ts
export class TasksComponent {
  @Input({ required: true }) selected_user_id!: string; // a set-table property from outside
  selectedUser = DUMMY_USERS.find((user) => user.id === selectedUserId);
}
```

- Template: put data to it

```html
<span> {{ selectedUser.name }}<span></span></span>
```

- set it in the AppComponent template

#### Solution

- `ng g c tasks --skipt-tests`

- tasks.component.html

```html
<h2>
  {{ name }}
  <h2></h2>
</h2>
```

- tasks.component.ts

```ts
@Input({ required: true }) name!: string
```

- app.component.html

  - pass data

- import TasksComponent in the AppComponent

```ts
imports: [... TasksComponent],
```

- app.component.ts - store the information which user was selected

```ts
export class AppComponent {
  ...
  selectedUserId = 'u1'  // set an initial value

  // use a computed value for a user that you can pass for property binding
  get selectedUser() {
    return this.users.find(user => user.id === this.selectedUserId)!
  }
  // the exlammation mark help to satisfy TS, although it can happen that
  // it does not find anything and returns undefined
 onSelectUser(id: string) {
  this.selectedUserId = id;
 }
}
```

- app.component.html

```html
<app-tasks [name]="selectedUser.name"></app-tasks>
```

- now upon selecting a user it output its name into the tasks

## TypeScript: undefined, union values, union types

```ts
@Input({ required: true }) id!: string;
```

- you say that undefined wont happen, technically still possible
- since `required`, ts already complains if not set in the code

- this however we cant say with certainty, it might return undefined

```ts
get selectedUser() {
    return this.users.find(user => user.id === this.selectedUserId)!
  }
```

- rather make the receiver more flexible
  - not `required` and a `?`
  - meand that this might not be set and I am aware of that

```ts
export class TasksComponent {
  @Input() name?: string;
}
```

- this "not required" and "?" does not interfere with (can be undefined)

```html
<h1>{{ name }}</h1>
```

-it does interfere with

```html
<app-tasks [name]="selectedUser.name" />
```

- since `selectedUser` might get undefined, of which has no `name`
- only access name if it is NOT undefined (plain JS-way)
- otherwise undefined as a fallback

```html
<app-tasks [name]="selectedUser?.name" />
```

- or define custom fallback value with ternary operator

```html
<app-tasks [name]="selectedUser ? selectedUser.name : 'fallback value'" />
```

### Union type

- alternatively you can just declare that `name` can be also undefined

```ts
export class TasksComponent {
  @Input() name: string | undefined;
}
```

### Type Alias: Define custom types with object types

- go from

```ts
export class UserComponent {
  @Input({ required: true }) id!: string;
  @Input({ required: true }) avatar!: string;
  @Input({ required: true }) name!: string;
```

- this

```ts
type  User = {
  id: string;
  avatar: string;
  name: string;
}
export class UserComponent {
  @Input({ required: true }) user!: User;
  ...
}
```

- adjust also everywhere, to access these through a user object: user.name, this.user.id, this.user.avatar

### Interface

- often either an `interface` or an alias `type` can be used
- no `=` in syntax definition

```ts
interface User {
  id: string;
  avatar: string;
  name: string;
}
```

## Outputting list content: Iteratig with angular

- from

```ts
<ul id="users">
    <li>
      <app-user
        [user]="users[0]"
        (select)="onSelectUser($event)"
      />
    </li>

    <li>
      <app-user
        [user]="users[1]"
        (select)="onSelectUser($event)"
      />
    </li>
```

- to

```ts
<ul id="users">
    @for (user of users; track user.id) {
      <li>
        <app-user
          [user]="user"
          (select)="onSelectUser($event)"
        />
      </li>
    }
  </ul>
```

- the `track` is for Angular update mechanism: to check if the data changes what it needs to re-render and what can keep

## Outputting conditional content

- eg. make `selectedUserId` optional
- from `selectedUserId = 'u1'`
- to

```ts
export class AppComponent {
  selectedUserId?: string;
}
```

- in the template from
  `<app-tasks [name]="selectedUser?.name"></app-tasks>`
- to

```html
@if (selectedUser) {
<app-tasks [name]="selectedUser.name"></app-tasks>
} @else {
<p id="Fallback">Select a user to see their tasks!</p>
}
```

### Legacy Angular: ngFor & ngIf

```html
<ul id="users">
  <!-- @for (user of users; track user.id) { -->
  <li *ngFor="let user of users">
    <app-user [user]="user" (select)="onSelectUser($event)" />
  </li>
  <!-- } -->
</ul>
```

- import `ngFor`, `ngIf` at the compoent too `app.comopnent`, so that you can use it in the template
  - `import { NgFor, NgIf } from '@angular/common';`
  - imports: [...NgFor, NgIf]

```html
<!-- @if (selectedUser) { -->
<app-tasks *ngIf="selectedUser; else fallback" [name]="selectedUser.name!" />
<!-- } @else { -->
<ng-template #fallback>
  <p id="Fallback">Select a user to see their tasks!</p>
</ng-template>
<!-- } -->
```

## Create a "task" component

- to create inside the tasks folder `ng g c tasks/task --skip-tests`

### Outputting User-specific Tasks

- add some dummy tasks to a class property `tasks`

```ts
export class TasksComponent {
  @Input() name?: string;
  tasks = [
    {
      id: 't1',
      userId: 'u1',
      title: 'Master Angular',
      summary:
        'Learn all the basic and advanced features of Angular & how to apply them.',
      dueDate: '2025-12-31',
    },
```

- iterate over it in the tasks template

```html
<ul>
  @for (task of tasks; track task.id) {
  <li>
    <app-task />
  </li>
  }
</ul>
```

- to only render the task of the given user, pass the id to the component
- create a computed property to obtain the selected users tasks only

```ts
export class TasksComponent {
  @Input({ required: true }) userId!: string;
  @Input({ required: true }) name!: string;

  get selectedUserTasks() {
    return this.tasks.filter(task => task.userId === this.userId)
  }
  tasks = [
    {
      id: 't1',
      userId: 'u1',
      title: 'Master Angular',
      ...
    }
    ...]
```

- pass `userId` in the app template

```html
@if (selectedUser) {
<app-tasks [userId]="selectedUser.id" [name]="selectedUser.name"></app-tasks>
} @else {
<p id="Fallback">Select a user to see their tasks!</p>
}
```

- use the computed property in the tasks template to only show the tasks of the user

```html
<ul>
  @for (task of selectedUserTasks; track task.id) {
  <li>
    <app-task />
  </li>
  }
</ul>
```

### Outputting Task Data in the Task Component

- pass it to the task

```ts
interface Task {
  id: string;
  userId: string;
  title: string;
  summary: string;
  dueDate: string;
}
export class TaskComponent {
  @Input({ required: true }) task!: Task;
}
```

```html
<ul>
  @for (task of selectedUserTasks; track task.id) {
  <li>
    <app-task [task]="task" />
  </li>
  }
</ul>
```

- task template

```html
<article>
  <h2>{{ task.title }}</h2>
  <time>{{ task.dueDate }}</time>
  <p>{{ task.summary }}</p>
  <p class="actions">
    <button>Complete</button>
  </p>
</article>
```

### Storing Data Models in Separate Files

- move user inteface into `user/user.model.ts`
- export it
- can reuse user everywhere, import it
- you can make if super explicit that it is only a type definition by adding the `type`, but not required

### Which user was selected? Dynamic CSS styling with Class Bindings

- use the `.active` CSS class from `user.component.css`

- add a new Input to UserComponent `@Input({required: true}) selected!: boolean;`

- in the app template

```html
<app-user [user]="user" [selected]="user.id === selectedUserId"></app-user>
```

- user template: use the `selected` property
  - binding to the class goes like [class.{name of class you want to conditionally bind eg. "active"}]

```html
<button [class.active]="selected" ...></button>
```

### "Complete" button: deleting tasks

- add a click listener on the `Complete` button in the task template

```html
<button (click)="onCompleteTask()">Complete</button>
```

- we want to communicate this to the tasks from the task
- add an `Output` to the task component

```ts
export class TaskComponent {
  ...
  @Output() complete = new EventEmitter<string>();

  onCompleteTask() {
    this.complete.emit(this.task.id);
  }
}
```

- listen for the `onCompleteTask` in the tasks template with `(complete)=""` and trigger any method of our choice

```html
<ul>
  @for (task of selectedUserTasks; track task.id) {
  <li>
    <app-task [task]="task" (complete)="onCompleteTask($event)" />
  </li>
  }
</ul>
```

- add a method that can be triggered once the parent component tasks receives and emitted `complete` from the `task`
- tasks component

```ts
export class TasksComponent {
  ...
  onCompleteTask(id: string) {
    this.tasks = this.tasks.filter((task) => task.id !== id);
  }
```

### Add task: create, add render a component
- there is already an `Add Tasks` button available in the `tasks` template

- ok I oviously took a way easier version of this like 
```html
<button (click)="onAddTask()">Add Tasks</button>
```

- tasks component
```ts
  onAddTask() {
    let newTask = {
      id: 't100',
      userId: `${this.userId}`,
      title: 'Brand new task',
      summary: 'Lorem Ipsum',
      dueDate: '2025-12-31',
    };
    this.tasks.push(newTask);
  }
```
-----------------------------------------
- a new component `newTask` should be rather created
```sh
ng g c tasks/new-task --skip-tests
```
```html
<button (click)="onStartAddNewTask()">Add Tasks</button>
```
- new method on the tasks `onStartAddNewTask`

- plus extra property whether the extra new task is visible or not
```ts
export class TasksComponent {
  ...
  isAddingTask = false
}

onStartAddNewTask() {
  this.isAddingTask = true;
}
```
- use `isAddingTask` in the tasks template to conditionally show the new empty task
- tasks template
```html
@if (isAddingTask) {
<app-new-task></app-new-task>
}
```

#### lets make it sure we can close the dialog
- emit an event from new-task component, if backdrop or cancel button are clicked, that can be handled by the tasks component, that is rendering the new-task, sets `isAddingTask` to false

##### My solution


- (similar to `onCompleteTask` (from the `task`))
- `onCancelAddTask`
  - should trigger a an event emission on the Cancel button, on the new-task
  ```html
  <button (click)="onCancelAddTask()" type="button">Cancel</button>
  ```
  which is in the new-task template, so define at new-task component
  ```ts
  export class NewTaskComponent {
  @Output() cancelAdd = new EventEmitter<boolean>();

  onCancelAddTask() {
    this.cancelAdd.emit(false);
  }
  ```
  - and catch on the `tasks` component with a method of same name
  ```ts
  export class TasksComponent {
     this.tasks = this.tasks.filter((task) => task.id !== id);
   }
 
  onStartAddNewTask() {
     this.isAddingTask = true;
   }

  onCancelAddTask(isAddingTask: boolean) {
    this.isAddingTask = isAddingTask;
  }

  ```
- invoke this method on the tasks template with the emitted event
```html
  @if (isAddingTask) {
    <app-new-task (cancelAdd)="onCancelAddTask($event)"></app-new-task>
 }
```

##### Instructors solution
```ts
export class TasksComponent {}
onCancelAddTask() {
    this.isAddingTask = false;
  }
```

- new-task template
```html
<div class="backdrop" (click)="onCancel()"></div>
...
<button (click)="onCancel()" type="button">Cancel</button>
```

- new-task component
```ts
export class NewTaskComponent {
  @Output() cancel = new EventEmitter<void>();

  onCancel() {
    this.cancel.emit();
  }
}
```

- tasks template
```html
@if (isAddingTask) {
<app-new-task (cancel)="onCancelAddTask()"></app-new-task>
}
```

### Extract the form data: Using directives & two-way-binding
- we have already new-task template
```html
 <form>
    <p>
      <label for="title">Title</label>
      <input type="text" id="title" name="title" />
    </p>
```

- define a property that will correspond to the form data, like `enteredTitle`
```ts
export class NewTaskComponent {
  @Output() cancel = new EventEmitter<void>();
  enteredTitle = '';
  
 ...
}
```

- add `[(ngModel)]` to establish two-way binding with directives
```html
<input type="text" id="title" name="title" [(ngModel)]="enteredTitle"/>
```

- also register the directive with the `FormsModule`
```ts
import { FormsModule } from '@angular/forms';

@Component({
  ...
  imports: [FormsModule],
})
export class NewTaskComponent {
  @Output() cancel = new EventEmitter<void>();
  enteredTitle = '';
 ...
}
```
- now also do 
  - `summary` -> `enteredSummary`
  - `due-date` -> `enteredDate`
- add to the respective html tag the `[(ngModel)]="enteredSummary"` and `[(ngModel)]="enteredDate"`

```ts
export class NewTaskComponent {
  @Output() cancel = new EventEmitter<void>();
  enteredTitle = '';
  enteredSummary = '';
  enteredDate = '';
 ...
}
```
#### Extract the form data: Using signals
- wrap the initial value `''` with the signal function
```ts
import { signal } from '@angular/core';

export class NewTaskComponent {
  ...
  enteredSummary = signal('');

```
- the template for two-way binding with signals is still valid, no need to change
```html
<input type="text" id="title" name="title" [(ngModel)]="enteredTitle"/>
```

### Handling Form Submission
- now we dont want to do the default behavior
- html works by defaaul if there is a `submit` button in a `form`, then upon clicking the button the browser would try to send the form data to the server that served the website
- our development server is not configured to take incoming data. The development server is only configured to serve the index.html file
- we want to close the dialog and add the new-task to the tasks array, handle it only on the client side
- if you import the `FormsModule` it will prevent the default submission
- to take control over the form submission behavior use `ngSubmit`

```html
<form (ngSubmit)="onSubmit()">
```

```ts
export class NewTaskComponent {
  ...
  onSubmit() {

  }
}
```

### Using the Submitted Data
- use `enteredTitle`, `enteredSummary``enteredDate` in `onSubmit()` to construct a new task
- let the `tasks` know about the new `task` data that has been submitted, to add a new task to the tasks array, also close the newTask dialog by setting `isAdding` to false in the tasks
- emit an event from newTask with an object data in it
- in `onSubmit` emit the event
```ts
export class NewTaskComponent {
  @Output() add = new EventEmitter<{title: string; summary: string; date: string}>();

  onSubmit() {
    this.add.emit({
      title: this.enteredTitle,
      summary: this.enteredSummary,
      date: this.enteredDate,
    })
  }
}
```

- listen for this event `add` in the `tasks` template
```html
<app-new-task ... (add)="onAddTask()" />
```

- you can define a `NewTaskData` interface eg. at the `task.model.ts`
```ts
export interface NewTaskData {
  title: string;
  summary; string;
  date: string;
}
```

- adapt at new-task component
```ts
export class NewTaskComponent {
  @Output() add = new EventEmitter<NewTaskData>();
```

- create the new method `onAddTask` at the tasks component
```ts
onAddTask(taskData: NewTaskData) {
  this.tasks.push({
    id: new Date().getTime().toString(),
    userId: this.userId,
    title: taskData.title,
    summary: taskData.summary,
    dueDate: taskData.date
  })
  this.isAddingTask = false;
}
```

- tasks template, the `add` event at the new-task calling the `onCancelAddTask` on the tasks
```ts
<app-new-task (cancel)="onCancelAddTask()" (add)="onAddTask($event)"></app-new-task>
```

## Content Projection with ng-content
- get the same styling by wrapping components into shared, reusable components

- there is a styling component which is already defined for the user component, but would be nice to reuse for other components
- lets create a folder for shared UI components, put a `card` component to it
```sh
ng g c shared/card --skip-tests
```

- move this from the user css into the cards css dfile
```css
div {
    border-radius: 6px;
    box-shadow: 0 1px 6px rgba(0, 0, 0, 0.1);
    overflow: hidden;
  }
```
- put a div into the card template `<div>...</div>`
- the idea is to use the `card` as a wrapper around the `user` button, instead of a "div"
- now in user template:
```html
<div>
  <button [class.active]="selected" (click)="onSelectUser()">
    <img [src]="imagePath" [alt]="user.name" />
    <span>
      {{ user.name }}
    </span>
  </button>
</div>
```
- to
```html
<app-card>
  <button [class.active]="selected" (click)="onSelectUser()">
    <img [src]="imagePath" [alt]="user.name" />
    <span>
      {{ user.name }}
    </span>
  </button>
</app-card>
```

- also reuse it for the task template, instead of the "article"
```html
<article>
  <h2>{{ task.title }}</h2>
  <time>{{ task.dueDate }}</time>
  <p>{{ task.summary }}</p>
  <p class="actions">
    <button (click)="onCompleteTask()">Complete</button>
  </p>
</article>
```
- to
```html
<app-card>
  <article>
    <h2>{{ task.title }}</h2>
    <time>{{ task.dueDate }}</time>
    <p>{{ task.summary }}</p>
    <p class="actions">
      <button (click)="onCompleteTask()">Complete</button>
    </p>
  </article>
</app-card>

```

- this is now just replacing the wrapped markup with the `...`

- if you want to combine the markup of the `card` and wrapped component, you need the special content in the markup of the warapper `ng-content`, which acts as a placeholder for the wrapped markup and merge these different markups together
```html
<div>
  <ng-content />
</div>
```


## Transforming Template Data with Pipes
- more like Outpu transformers
- the date from the task template
```html
<app-card>
  <article>
    ...
    <time>{{ task.dueDate }}</time>
```
- is like 2015-12-28
- import it
```ts
import { DatePipe } from '@angular/common';

@Component({
  ...
  imports: [... DatePipe],
```
- use it like
```html
<time>{{ task.dueDate | date }}</time>
```

- can be configured like
```html
<time>{{ task.dueDate | date:'fullDate' }}</time>
```