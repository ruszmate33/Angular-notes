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
- `ng g c investment-results` or `ng g c user-input`

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

- `ng g c tasks --skip-tests`

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

---

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
</form>
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
<input type="text" id="title" name="title" [(ngModel)]="enteredTitle" />
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
<input type="text" id="title" name="title" [(ngModel)]="enteredTitle" />
```

### Handling Form Submission

- now we dont want to do the default behavior
- html works by defaaul if there is a `submit` button in a `form`, then upon clicking the button the browser would try to send the form data to the server that served the website
- our development server is not configured to take incoming data. The development server is only configured to serve the index.html file
- we want to close the dialog and add the new-task to the tasks array, handle it only on the client side
- if you import the `FormsModule` it will prevent the default submission
- to take control over the form submission behavior use `ngSubmit`

```html
<form (ngSubmit)="onSubmit()"></form>
```

```ts
export class NewTaskComponent {
  ...
  onSubmit() {

  }
}
```

### Using the Submitted Data

- use `enteredTitle`, ` enteredSummary``enteredDate ` in `onSubmit()` to construct a new task
- let the `tasks` know about the new `task` data that has been submitted, to add a new task to the tasks array, also close the newTask dialog by setting `isAdding` to false in the tasks
- emit an event from newTask with an object data in it
- in `onSubmit` emit the event

```ts
export class NewTaskComponent {
  @Output() add = new EventEmitter<{
    title: string;
    summary: string;
    date: string;
  }>();

  onSubmit() {
    this.add.emit({
      title: this.enteredTitle,
      summary: this.enteredSummary,
      date: this.enteredDate,
    });
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
  summary;
  string;
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
    <span> {{ user.name }} </span>
  </button>
</div>
```

- to

```html
<app-card>
  <button [class.active]="selected" (click)="onSelectUser()">
    <img [src]="imagePath" [alt]="user.name" />
    <span> {{ user.name }} </span>
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
  </article></app-card
>
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

## Getting started with services

- `tasks` is doing a lot
- refactor methods into services
- keeping comopnents and classes as lean as possible
- many methods on the `tasks` are used by other components
- `newTask` adds a task with `NewTaskComponent.onSubmit()` -> `TasksComponent.onAddTask()`
- `task` deletes a task: `TaskComponent.onCompleteTask()` -> `TasksComponent.onCompleteTask()`

- create services at /tasks: `/tasks/tasks.service.ts`
- service performs operation, manages data
- move the `tasks` array here, make it `private`
- add some public methods that can be reached outside the class and can get or manipulate these tasks
- like `getUserTasks(userId: string)`, addTask, removeTask

```ts
export class TasksService {
  private tasks = [
    {
      id: 't1',
      userId: 'u1',
      ...
    },
    ...
  ];
  getUserTasks(userId: string) {
    return this.tasks.filter((t) => t.userId === userId);
  }
```

### Getting started with dependency injection

- export the `TasksService` to be able to use it in the TasksComponents, where you import it like `import { TasksService } from './tasks.service';`

One possibility:

- instantiate the class as a private property on the component

```ts
export class TasksComponent {
  ...
  private tasksService = new TasksService()

get selectedUserTasks() {
  return this.tasksService.getUserTasks(this.userId);
}
}
```

- huge drawback: to use the service in a different component, we would need to instantiate it over there too, and that is a different instance, not shared
- dependency injection solves this

#### Dependency injection

- add a `constructor()` method to the class eg. TaskComponent
- will automatically executed once the class is instantiated, like TaskComponent
- Angular calls it
- specify the dependency and the tyoe, pass the class in it

```ts
export class TasksComponent {
  ...
  private tasksService: TasksService;
  constructor(tasksService: TasksService) {
    this.tasksService = tasksService
  }
}
```

OR a shortcut

- no need to declare property tasksService on the class but only in the constructor

```ts
export class TasksComponent {
  ...
  constructor(private tasksService: TasksService) {}
}
```

- also need to register `TasksService` class as an injectable
- with `Injectable` decorator, that is also called and a config object is passed to it

```ts
import { Injectable } from "@angular/core";

@Injectable({ providedIn: "root" })
export class TasksService {}
```

### More Service Usage & Alternative Dependency Injection Mechanism

- `new-tasks.component` use the `TasksService` class name as an "injection-token"
- that is passed to the `inject` function

```ts
import { inject } from "@angular/core";
import { TasksService } from "../tasks.service";

export class NewTaskComponent {
  private tasksService = inject(TasksService);
}
```

- now you can get swap the `onSubmit`
- also get the userId as an input, bc it will need it

```ts
onSubmit() {
    this.add.emit({
      title: this.enteredTitle,
      summary: this.enteredSummary,
      date: this.enteredDate,
    });
  }
```

to

```ts
export class NewTaskComponent {
@Input({required: true}) userId!: string;

onSubmit() {
      this.tasksService.addTask({
        title: this.enteredTitle,
        summary: this.enteredSummary,
        date: this.enteredDate,
      },
      this.userId);
    }
```

- set the userId in `task.component.html`

```html
<app-new-task [userId]="userId" ...></app-new-task>
```

- we no longer emit the add event and we can get rid of it in the `NewTasksComponent`, from this `@Output() add = new EventEmitter<NewTaskData>();`

- still to close the dialog
- rename the `cancel` event to `close` and emit it after the data has been submitted

```ts
export class NewTaskComponent {
  @Output() close = new EventEmitter<void>();

  onCancel() {
    this.close.emit();
  }

  onSubmit() {
      this.tasksService.addTask({
        title: this.enteredTitle,
        summary: this.enteredSummary,
        date: this.enteredDate,
      },
      this.userId);
    this.close.emit();
  }
```

- adapt in the `tasks.component.html` to listen for a `close`, not a `cancel` event

from

```html
@if (isAddingTask) {
<app-new-task
  [userId]="userId"
  (cancel)="onCancelAddTask()"
  (add)="onAddTask($event)"
></app-new-task>
}
```

TO

```html
@if (isAddingTask) {
<app-new-task [userId]="userId" (close)="onCloseAddTask()"></app-new-task>
}
```

```ts
export class TasksComponent {
  ...

  // onCancelAddTask() {
  onCloseAddTask() {
    this.isAddingTask = false;
  }

  // remove onAddTask
```

#### add back the completion of tasks functionality

```ts
export class TasksComponent {


   onCompleteTask(id: string) {
+    this.tasksService.removeTask(id);
```

- this works for me
- remove `tasks.oncompleteTask`
- ok the main exercise is to not call this service inside `tasks` component
- is it a better practice to have the logic on the template?!
- right now:
- `tasks` template

```html
<ul>
  @for (task of selectedUserTasks; track task.id) {
  <li>
    <app-task [task]="task" (complete)="onCompleteTask($event)" />
  </li>
  }
</ul>
```

- `task` component

```ts
export class TaskComponent {
  @Input({ required: true }) task!: Task;
  @Output() complete = new EventEmitter<string>();

  onCompleteTask() {
    this.complete.emit(this.task.id);
  }
}
```

=================================

- it could be possible to not emit an event at `task` to `tasks`, but call the service?
- `task` component

```ts
export class TaskComponent {
  @Input({ required: true }) task!: Task;
  // @Output() complete = new EventEmitter<string>();
  private taskService = inject(TasksService);

  onCompleteTask() {
    // this.complete.emit(this.task.id);
    this.taskService.removeTask(this.task.id);
  }
}
```

- also remove `(complete)="onCompleteTask($event)"` from the `tasks` template

```html
<ul>
  @for (task of selectedUserTasks; track task.id) {
  <li>
    <app-task [task]="task" />
  </li>
  }
</ul>
```

- this works!!!
- so I guess if you maintain data in the services and manipulate it like child -> parnent -> service via event emission, you can just go call from child component the service

##### instructor does

- yes, this was the solution

### Using local storage for Data Storage

- not just to keep them in a local array in the service

```ts
export class TasksService {
  constructor() {
    const tasks = localStorage.getItem('tasks');

    if (tasks) {
      this.tasks = JSON.parse(tasks);
    }
  }

  private saveTasks() {
    localStorage.setItam('tasks', JSON.stringify(this.tasks))
  }
  // call saveTasks after adding or removing a task

  addTask(taskData: NewTaskData, userId: string) {
    this.tasks.push({
      id: new Date().getTime().toString(),
      userId: userId,
      title: taskData.title,
      summary: taskData.summary,
      dueDate: taskData.date,
    });
    this.saveTasks();
  }
  removeTask(id: string) {
    this.tasks = this.tasks.filter((task) => task.id !== id);
    this.saveTasks()
  }
```

## Angular Modules (MgModule)

- instead of standalone components, the old way

### Turn a standalone component to a module

- create `src/app/app.module.ts`
- in the decorator `declarations` go all the components you need to register to work together
- also directives, but that comes later
- from all the components to use at `NgModule` remove the `standalone: true` (now from `AppComponent`)
- also remove the `imports: [HeaderComponent, ...]` at `AppComponent` as this is only supported for standalone components

```ts
import { NgModule } from "@angular/core";

import { AppComponent } from "./app.component";

@NgModule({
  declarations: [AppComponent],
})
export class AppModule {}
```

### Adapt the main.ts

- this only work with standalone component `AppComponent`
- what is the root component

main.ts

```ts
import { bootstrapApplication } from "@angular/platform-browser";
import { appConfig } from "./app/app.config";
import { AppComponent } from "./app/app.component";

bootstrapApplication(AppComponent, appConfig).catch((err) =>
  console.error(err)
);
```

- turn into

```ts
import { platformBrowserDynamic } from "@angular/platform-browser-dynamic";

import { AppComponent } from "./app/app/module";

platformBrowserDynamic().bootstrapModule(AppModule);
```

- also configure `bootstrap` in the root module, put all the root components into the array (typically only one)

app.module.ts

```ts
import { NgModule } from '@angular/core';

import {AppComponent} from './app.component';

@NgModule({
  declarations: [AppComponent, HeaderComponent]
  bootstrap: [AppComponent]
})
export class AppModule {

}
```

- we need to declare all the components in app.module.ts `declarations` that must know about each other. The `imports: [HeaderComponent, ...]` is gone from the component

### Mix and Match standalone components with modules

...67-68

## Angular Essentials - Practice

- in the root of the new project is a `public` folder; this can be used not via

```html
<img src="public/image.png" />
```

but

```html
<img src="image.png" />
```

### Create a HeaderComponent with title & image

```sh
ng g c header --skip-tests
```

CREATE src/app/header/header.component.css (0 bytes)
CREATE src/app/header/header.component.html (21 bytes)
CREATE src/app/header/header.component.ts (234 bytes)

- header.component.html

```html
<header>
  <h1 class="header">Investment Calculator</h1>
  <img class="header img" src="investment-calculator-logo.png" alt="logo" />
</header>
```

- header.component.css

```css
header {
  text-align: center;
  margin: 3rem auto;
}

header img {
  width: 5rem;
  height: 5rem;
  object-fit: contain;
  background-color: transparent;
}

header h1 {
  font-size: 1.5rem;
}
```

- app.component.ts

```ts
import { HeaderComponent } from './header/header.component';

@Component({
  ...
  imports: [HeaderComponent]
})
```

## User-input

```sh
ng g c user-input --skip-tests
```

- rather use `annual-investment` and not camelCase "annualInvestment" for label identifiers, so not
- also specify the type as `number` in the input, not `text`

```html
<label for="annualInvestment">Annual Investment</label>
<input
  type="number"
  id="annualInvestment"
  name="annualInvestment"
  [(ngModel)]="annualInvestment"
/>
```

buy

```html
<label for="annual-investment">Annual Investment</label>
<input
  type="number"
  id="annual-investment"
  name="annual-investment"
  [(ngModel)]="annual-investment"
/>
```

app.component.html

```html
<app-header /> <app-user-input />
```

app.component.ts

```ts
@Component({
  ...
  imports: [HeaderComponent, UserInputComponent], // make available for the template
})
export class AppComponent {}
```

- submit the form

```html
<form (ngSubmit)="onSubmit()">
  <div class="input-group">...</div>
</form>
```

- import `FormsModule` so the form gets extended and can listen for the `ngSubmit` event
- define the `onSubmit` so the ngSubmit-listener calls it

```ts
@Component({
  ...
  imports: [FormsModule],
  ...
  })
export class UserInputComponent {
  onSubmit() {

  }
```

- two-way binding with the `ngModel` directive (available via the `FormsModule`)

```html
<label for="initial-investment">Initial Investment</label>
<input
  type="number"
  id="initial-investment"
  name="initial-investment"
  [(ngModel)]="enteredInitialInvestment"
/>
```

```ts
export class UserInputComponent {
  enteredInitialInvestment = signal('');
```

- alternatvely, not through signals

```ts
export class UserInputComponent {
  enteredInitialInvestment = '0';
```

- Turning the input string into a number
- instead of `parseFloat` it can be just a `+`
- `initialInvestment: parseFloat(this.enteredInitialInvestment)`
  vs
- `initialInvestment: +this.enteredInitialInvestment`

```ts
@Component({
  selector: "app-user-input",
  standalone: true,
  imports: [FormsModule],
  templateUrl: "./user-input.component.html",
  styleUrl: "./user-input.component.css",
})
export class UserInputComponent {
  @Output() calculate = new EventEmitter<InvestmentInput>();

  enteredInitialInvestment = "0";
  enteredAnnualInvestment = "0";
  enteredExpectedReturn = "5";
  enteredDuration = "10";

  onSubmit() {
    this.calculate.emit({
      initialInvestment: +this.enteredInitialInvestment,
      annualInvestment: +this.enteredAnnualInvestment,
      expectedReturn: +this.enteredExpectedReturn,
      duration: +this.enteredDuration,
    });
  }
}
```

- in the app template

```html
... <app-user-input (calculate)="calculateInvestment($event)" />
```

- this `$event` is to access the emitted data, that is attached to the emitted custom event output

### Define an interface for InvestmentInput

- you can put the object definition into a `investment-input.model.ts` file next to app.component

```ts
export interface InvestmentInput {
  initialInvestment: number;
  annualInvestment: number;
  expectedReturn: number;
  duration: number;
}
```

### Investment results

```ts
export class InvestmentResultsComponent {
  @Input() annualData?: AnnualInvestmentData[] = [];
}
```

```ts
export interface AnnualInvestmentData {
  year: number;
  interest: number;
  valueEndOfYear: number;
  annualInvestment: number;
  totalInterest: number;
  totalAmountInvested: number;
}
```

- the best is to store the result of the calculation in the app component as a property

```ts
export class AppComponent {
  private service = inject(InvestmentService);
  annualData: AnnualInvestmentData[] = [];
```

- pass it in the app template

```ts
<app-investment-results [annualData]="annualData"/>
```

- this is a common pattern in angular: one child component (userinput) provides the inputs for the parent
- the parent calculates/manipulates the data (calculateInvestmentResults)
- passes the data to another child component (investment results)

```html
<app-header />
<app-user-input (calculate)="calculateInvestment($event)" />
<app-investment-results [annualData]="annualData" />
```

#### Outputting data in the Table

```html
<table *ngIf="annualData; else noData">
  <thead>
    <tr>
      <th scope="col">Year</th>
      <th scope="col">Interest</th>
      <th scope="col">Value End of the Year</th>
      <th scope="col">Annual Investment/th></th>
      <th scope="col">Totoal Interest</th>
      <th scope="col">Total Amount Invested</th>
    </tr>
  </thead>
  <tbody *ngFor="let annualD of annualData">
    <tr>
      <th scope="row">{{ annualD.year }}</th>
      <td>{{ annualD.interest }}</td>
      <td>{{ annualD.valueEndOfYear }}</td>
      <td>{{ annualD.annualInvestment }}</td>
      <td>{{ annualD.totalInterest }}</td>
      <td>{{ annualD.totalAmountInvested }}</td>
    </tr>
  </tbody>
</table>

<ng-template #noData>
  <p>No investment data available.</p>
</ng-template>
```

- alternatively with the more modern iteration

```html
@if (!annualData) {
<p class="center">Please enter some values and press "Calculate"</p>
} @else {
<table>
  <thead>
    <tr>
      <th scope="col">Year</th>
      <th scope="col">Interest</th>
      <th scope="col">Value End of the Year</th>
      <th scope="col">Annual Investment/th></th>
      <th scope="col">Totoal Interest</th>
      <th scope="col">Total Amount Invested</th>
    </tr>
  </thead>
  <tbody>
    @for (annualD of annualData; track annualD.year) {
    <tr>
      <td>{{ annualD.year }}</td>
      <td>{{ annualD.interest | number: '1.0-2'}}</td>
      <td>{{ annualD.valueEndOfYear | currency }}</td>
      <td>{{ annualD.annualInvestment | currency }}</td>
      <td>{{ annualD.totalInterest | number: '1.0-0' }}</td>
      <td>{{ annualD.totalAmountInvested | currency }}</td>
    </tr>
    }
  </tbody>
</table>
}
```

- also import these to use the pipes:

```ts
import { CurrencyPipe, DecimalPipe } from
```

## Overview of Inputs and Outputs in the first Angular app

### Component TS files

- `AppComponent`

  - nothing, but `selectedUserId?: string;` and `onSelectUser(id: string) {this.selectedUserId = id;}`

- `User`

  - user (input)
  - selected (input)
  - select (output)

- `TasksComponent`

  - userId (input)
  - name (input)

- `TaskComponent`

  - task (input)

- `NewTaskComponent`
  - userId (input)
  - close (output)

### Templates

#### `App` template

- for `User`

```html
<app-user
  [user]="user"
  [selected]="user.id === selectedUserId"
  (select)="onSelectUser($event)"
/>
```

- TODO where is `onSelectUser` defined?
- for `tasks`

```html
<app-tasks [userId]="selectedUser.id" [name]="selectedUser.name"></app-tasks>
```

===================

#### `User` template

```html
<button [class.active]="selected" (click)="onSelectUser()"></button>
```

#### `NewTask` template

- listening for `(click)="onCancel()"` and `(ngSubmit)="onSubmit()"`
  - these methods are defined on `NewTask`
  - `onCancel()` will emit the `close` event
  - the input `userId` is not handled here

#### `Tasks` template

- the `NewTasks` is used here, `userId` is passed to it
- the `Task` is also used and `task` inputis passed to to it

- `NewTasks` listening for `(close)="onCancelAddTask()"` and `(click)="onStartAddNewTask()"` which are defined on `Tasks`

- `userId` and `name` inputs are not received here: `<app-task [task]="task" />`

#### `App` template

- `Tasks` is used here and `userId` and `name` are passed to it

```html
<app-tasks [userId]="selectedUser.id" [name]="selectedUser.name"></app-tasks>
```

- `User` is used here

  - `user`, `selected` is passed to it
  - listening for `select` with `onSelectUser`, defined on the `app`

  ```html
  <app-user
    [user]="user"
    [selected]="user.id === selectedUserId"
    (select)="onSelectUser($event)"
  />
  ```

  ### Case study 1: selecting a user

  - in `app` template `user` is listening for `select` with `(select)="onSelectUser($event)"`
  - this will set the `selectedUserId` on the `app`, which is used for filtering
  - it needs to receive the `id` from the `user`
  - it passes `user` and `selected` (bool) to user
  - the `select` event is emitted from the `user`

  ```ts
  onSelectUser() {
    this.select.emit(this.user.id);
  }
  ```

  - there is a `onSelectUser` defined on the `app` too
  - will this be used too?

  ```ts
  onSelectUser(id: string) {
    console.log('Selected user with id' + id);
    this.selectedUserId = id;
  }
  ```

  - which one is invoked at `app-user` in the app template

  ```html
  <app-user
    [user]="user"
    [selected]="user.id === selectedUserId"
    (select)="onSelectUser($event)"
  />
  ```

  - this is `AppComponent.onSelectUser`
  - the User - `onSelectUser` is listening for a `click` on its own template `(click)="onSelectUser()"`

### Question: Should a child component also listen to an emitted event of a parent component?

Or what is the idiomatic approach there?

In Angular, a `child component typically does not listen to events emitted by its parent component`. Instead, the - parent-to-child communication is usually handled through `@Input` bindings, which allow the parent to pass data or state down to the child.

- Conversely, child-to-parent communication is handled through `@Output` and EventEmitter, as you've seen.

Angular encourages unidirectional data flow:

    Parent to Child: Using @Input() for data binding.
    Child to Parent: Using @Output() and EventEmitter for event emission.

#### Parent Component Template (AppComponent):

<app-user
[id]="selectedUserId"
[highlight]="isUserHighlighted"

> </app-user>

Here, the parent is passing two values down to the child:

    selectedUserId (which might change based on user actions).
    isUserHighlighted (which controls whether the child user component should be highlighted).

Child Component (UserComponent):

```ts
import { Input, Component } from "@angular/core";

@Component({
  selector: "app-user",
  template: `
    <div [class.highlight]="highlight">
      <p>User ID: {{ id }}</p>
    </div>
  `,
})
export class UserComponent {
  @Input() id!: string;
  @Input() highlight: boolean = false;
}
```

In this case, whenever the parent updates selectedUserId or isUserHighlighted, the child component (UserComponent) receives these updated values via @Input(), and it can react to them as needed.

====================================

### Using Signals and resetting the form after submission

- Also solve the Investment Calculator with Signals

```ts
export class AppComponent {
  private service = inject(InvestmentService);
  annualData?: AnnualInvestmentData[];

  calculateInvestment(investmentInput: InvestmentInput) {
      this.annualData = this.service.calculateInvestmentResults(investmentInput);
      ...
  }
}
```

- this is a stateful data, which when changes would have an impact on the UI
- so it could be managed by a signal, instead of like this

```ts
import { signal } from '@angular/core';
export class AppComponent {
  ...
  annualData = signal<AnnualInvestmentData[] | undefined>(undefined);

  calculateInvestment(investmentInput: InvestmentInput) {
      this.annualData.set(this.service.calculateInvestmentResults(investmentInput));
      ...
    }
}
```

- also change, how it how it gets updated, not `=` but `.set()`

- the app template, add `()`
  from

```html
<app-investment-results [annualData]="annualData" />
```

to

```html
<app-investment-results [annualData]="annualData()" />
```

#### Signals for the user input components too

- two-way binding

from

```ts
export class UserInputComponent {
  @Output() calculate = new EventEmitter<InvestmentInput>();

  enteredInitialInvestment = "0";
  enteredAnnualInvestment = "0";
  enteredExpectedReturn = "5";
  enteredDuration = "10";

  onSubmit() {
    this.calculate.emit({
      initialInvestment: +this.enteredInitialInvestment,
      annualInvestment: +this.enteredAnnualInvestment,
      expectedReturn: +this.enteredExpectedReturn,
      duration: +this.enteredDuration,
    });
  }
}
```

to

```ts
export class UserInputComponent {
  ...
  enteredInitialInvestment = signal('0');
  enteredAnnualInvestment = signal('0');
  enteredExpectedReturn = signal('5');
  enteredDuration = signal('10');
  ...
```

- you dont need to call them in the template with `()`, ngModel is accepting Signals as well, reads them and sets up a subscription for the

```html
<label for="duration">Duration</label>
<input
  type="number"
  id="duration"
  name="duration"
  [(ngModel)]="enteredDuration"
/>
```

- However, `onSubmit` need to emit the values of the signals and not the signals, so we read the signals by calling them

```ts
  onSubmit() {
    this.calculate.emit({
      initialInvestment: +this.enteredInitialInvestment(),
      annualInvestment: +this.enteredAnnualInvestment(),
      expectedReturn: +this.enteredExpectedReturn(),
      duration: +this.enteredDuration(),
    });
  }
}
```

- also instead of the `@Output` decorator, you could use the `output` function

```ts
  @Output() calculate = new EventEmitter<InvestmentInput>();
```

to

```ts
calculate = output<InvestmentInput>();
```

- also from this

```ts
export class InvestmentResultsComponent {
  @Input() annualData?: AnnualInvestmentData[];
}
```

to

```ts
export class InvestmentResultsComponent {
  annualData = input<AnnualInvestmentData[]>();
}
```

and in the template call the signal to read it and set up the subscription

```html
@if (!annualData()) {

<tbody>
  @for (annualD of annualData(); track annualD.year) {
  <tr></tr>
</tbody>
```

#### Reset the input values

```ts
export class UserInputComponent {
  ...
  onSubmit() {
    this.calculate.emit({
      initialInvestment: +this.enteredInitialInvestment,
      annualInvestment: +this.enteredAnnualInvestment,
      expectedReturn: +this.enteredExpectedReturn,
      duration: +this.enteredDuration,
    });
    this.enteredInitialInvestment.set('0');
    this.enteredAnnualInvestment.set('0');
    this.enteredExpectedReturn.set('5');
    this.enteredDuration.set('10');
  }
```

### Using a Service for cross-component communication

- I primarily just put a function into the service from the app component, but already the userInput could put the input to it and the results could get it from there

- this is still using the classic paradigm of: userInput -> appComponent -> resultsComponent

- the service

```ts

```

from this

```ts
@Component({
  selector: "app-root",
  standalone: true,
  templateUrl: "./app.component.html",
  imports: [HeaderComponent, UserInputComponent, InvestmentResultsComponent],
})
export class AppComponent {
  private service = inject(InvestmentService);
  // annualData = signal<AnnualInvestmentData[] | undefined>(undefined);
  annualData = signal<AnnualInvestmentData[] | undefined>(undefined);

  calculateInvestment(investmentInput: InvestmentInput) {
    this.annualData.set(
      this.service.calculateInvestmentResults(investmentInput)
    );
    console.log(`annualData on app component: ${this.annualData}`);
  }
}
```

to

```ts
@Component({
  selector: "app-root",
  standalone: true,
  templateUrl: "./app.component.html",
  imports: [HeaderComponent, UserInputComponent, InvestmentResultsComponent],
})
export class AppComponent {}
```

- the service

```ts
@Injectable({ providedIn: "root" })
export class InvestmentService {
  resultsData?: AnnualInvestmentData[];
  calculateInvestmentResults(investmentInput: InvestmentInput) {
    let annualData = [];
    let investmentValue = investmentInput.initialInvestment;

    for (let i = 0; i < investmentInput.duration; i++) {
      const year = i + 1;
      const interestEarnedInYear =
        investmentValue * (investmentInput.expectedReturn / 100);
      investmentValue +=
        interestEarnedInYear + investmentInput.annualInvestment;
      const totalInterest =
        investmentValue -
        investmentInput.annualInvestment * year -
        investmentInput.initialInvestment;
      annualData.push({
        year: year,
        interest: interestEarnedInYear,
        valueEndOfYear: investmentValue,
        annualInvestment: investmentInput.annualInvestment,
        totalInterest: totalInterest,
        totalAmountInvested:
          investmentInput.initialInvestment +
          investmentInput.annualInvestment * year,
      });
    }
    console.log(`InvestmentService: ${annualData}`);
    this.resultsData = annualData;
  }
}
```

- use the `InvestmentService` already in the userInput
  - don't output the InvestmentInput to the usercomponent
  - remove `calculate = output<InvestmentInput>();`
  - dont just emit in onSubmit
  - make service available

```ts
export class UserInputComponent {
  // remove this
  // calculate = output<InvestmentInput>();

  // make service available
  constructor(private investmentService: InvestmentService) {}


  // but reach out to the service at onSubmit
    onSubmit() {
    // remove
    this.calculate.emit({
      initialInvestment: +this.enteredInitialInvestment(),
      annualInvestment: +this.enteredAnnualInvestment(),
      expectedReturn: +this.enteredExpectedReturn(),
      duration: +this.enteredDuration(),
    });

    // use service
    this.investmentService.calculateInvestmentResults({
      initialInvestment: +this.enteredInitialInvestment(),
      annualInvestment: +this.enteredAnnualInvestment(),
      expectedReturn: +this.enteredExpectedReturn(),
      duration: +this.enteredDuration(),
    })

```

- use the `InvestmentService` also in the investmentResults
  - remove the `input`
  - inject the service
  - access the calculation result from the service

```ts
export class InvestmentResultsComponent {
  constructor(private investmentService: InvestmentService) {}

  // remove
  annualData = input <AnnualInvestmentData[]>();
  annualData = this.investmentService.

  // add a getter to access it
  get results() {
    return this.investmentService.resultsData;
  }
}
```

- in the results-template, not a signal anymore so not `annualData()` but `results`

- since the appCompnent is not the center of dataflow through the service, remove the method calls from there and the input and event-binding from app-template

```html
<app-header />
<app-user-input (calculate)="calculateInvestment($event)" />
<app-investment-results [annualData]="annualData()" />
```

```html
+<app-user-input /> +<app-investment-results />
```

- with this it is not input -> app -> results, but the input uses the service to calculate and put data into the service, the results just to get the calculated results

### Using Signals in Services

from

```ts
@Injectable({ providedIn: 'root' })
export class InvestmentService {
  resultsData?: AnnualInvestmentData[];
  ...
  this.resultsData = annualData;
}
```

to

```ts
@Injectable({ providedIn: 'root' })
export class InvestmentService {
  resultsData:  = signal<AnnualInvestmentData[] | undefined>(undefined);
  ...
  this.resultsData.set(annualData);
}
```

- in the template it has to be called to get the value of the signal and set up the subsciprtion

- also you can setup a computed value instead a getter to make it read-only, and not change accidentally the value
  from

```ts
export class InvestmentResultsComponent {
 ...
  get results() {
    return this.investmentService.resultsData;
  }
```

to

```ts
export class InvestmentResultsComponent {
 ...
  results = computed(() => this.investmentService.resultsData());
```

- alternatively, this also provides a read-only version of the signal

```ts
results = this.investmentService.resultsData.asReadonly();
```

## Debugging

```sh
❯ ng serve
Application bundle generation failed. [2.547 seconds]

✘ [ERROR] NG2: Type '{ id: string; name: string; avatar: string; }' is not assignable to type 'string'. [plugin angular-compiler]

    src/app/app.component.html:13:16:
      13 │     <app-tasks [userId]="selectedUser" [name]="selectedUser.name" />
         ╵                 ~~~~~~

  Error occurs in the template of component AppComponent.

    src/app/app.component.ts:11:17:
      11 │     templateUrl: './app.component.html',
         ╵                  ~~~~~~~~~~~~~~~~~~~~~~
```

- use the source debugger of the browser for logical errors
- also worth to use the Angular DevTools extension in Chrome

## Deep-dive into components and templates

### Splitting a component into multiple templates

- to create a component inside a folder, for better structure
  `ng g c dashboard/server-status --skip-tests`

### Create reusable components

- `ng g c dashboard/dashboard-item --skip-tests`
- from

```html
<article>
  <header>
    <img src="status.png" alt="A signal symbol" />
    <h2>Server Status</h2>
  </header>
  <div>...</div>
</article>
```

- and from

```html
<app-header />
<main>
  <div id="dashboard">
    <div class="dashboard-item">
      <app-server-status />
    </div>
    <div class="dashboard-item">
      <app-traffic />
    </div>
    <div class="dashboard-item">
      <app-tickets />
    </div>
  </div>
</main>
```

to

- dashboard-item

```html
<div class="dashboard-item">
  <article>
    <header>
      <img [src]="image().src" [alt]="image().alt" />
      <h2>{{ title() }}</h2>
    </header>
  </article>
</div>
```

```html
<app-header />
<main>
  <div id="dashboard">
    <app-dashboard-item
      [image]="{ src: 'status.png', alt: 'A signal symbol' }"
      title="Server status"
    >
      <app-server-status />
    </app-dashboard-item>
    <app-dashboard-item
      [image]="{ src: 'globe.png', alt: 'A globe' }"
      title="Traffic"
    >
      <app-traffic />
    </app-dashboard-item>
    <app-dashboard-item
      [image]="{ src: 'list.png', alt: 'A list of items' }"
      title="Support Tickets"
    >
      <app-tickets />
    </app-dashboard-item>
  </div>
</main>
```

#### Property Binding Revisited

- image expects an object
- so we need to set up property-binding with `[image]=` and between the double-quotes some dynamically evaluated TS expression `"{ src: 'status.png', alt: 'A signal symbol' }"`, set up an object here, evaluated as object and not as text
- for title we don't need property-binding
  - the square-brackets wrapped around a property would tell to evaluate as TS code, but we need just a plain string
  - `[title]="A signal symbol"` would try to evaluate `A signal symbol` as TS syntax
  - so we go with either `[title]="'A signal symbol'"`
  - or even simpler `title="A signal symbol"`

```ts
export class DashboardItemComponent {
  image = input.required<{ src: string; alt: string }>();
  title = input.required<string>();
}
```

```html
<app-dashboard-item
  [image]="{ src: 'status.png', alt: 'photo' }"
  title="A signal symbol"
></app-dashboard-item>
```

#### Using content projection and ng-content

- put `ng-content` element into the template

```html
<div class="dashboard-item">
  <article>
    <header>
      <img [src]="image().src" [alt]="image().alt" />
      <h2>{{ title() }}</h2>
    </header>
    <ng-content />
  </article>
</div>
```

@import url("https://fonts.googleapis.com/css2?family=Open+Sans:ital,wght@0,300..800;1,300..800&display=swap");

- {
  box-sizing: border-box;
  }

html {
font-family: "Open Sans", sans-serif;
}

body {
margin: 0;
background-color: #c0bdc3;
}

main {
width: 80%;
margin: 3rem 10%;
}

header {
padding: 0.75rem 0.1rem;
display: flex;
flex-direction: column;
gap: 1rem;
justify-content: space-between;
align-items: center;
font-size: 1rem;
}

#logo {
width: 5.5rem;
height: 5.5rem;
background-color: #eee8f2;
padding: 1.25rem;
border-radius: 50%;
box-shadow: 0 0 8px rgba(0, 0, 0, 0.35);
}

#logo img {
width: 100%;
height: 100%;
filter: drop-shadow(0 0 4px rgba(29, 29, 29, 0.35));
}

nav ul {
display: flex;
gap: 2rem;
list-style: none;
align-items: center;
padding: 0;
margin: 0;
}

nav ul li a {
color: #3e3b3e;
text-decoration: none;
font-weight: bold;
}

nav ul li a:hover {
color: #77207a;
}

@media (min-width: 768px) {
header {
font-size: 1.25rem;
flex-direction: row;
gap: 0;
padding: 1.5rem 10%;
}
}

#dashboard {
display: flex;
flex-direction: column;
align-items: flex-start;
justify-content: space-between;
gap: 1.5rem;
max-width: 85rem;
}

@media (min-width: 768px) {
#dashboard {
flex-direction: row;
}
}

#traffic {
display: block;
width: 15rem;
}

p {
margin: 0 0 1rem 0;
font-size: 0.9rem;
color: #4f4b53;
}

#chart {
height: 10rem;
display: flex;
align-items: flex-end;
gap: 0.5rem;
padding: 0 0.5rem;
border-bottom: 1px solid #76737a;
}

#chart div {
flex: 1;
background: linear-gradient(to bottom, #36166f, #ca19a4);
border-top-left-radius: 4px;
border-top-right-radius: 4px;
}

@media (min-width: 768px) {
#traffic {
width: 20rem;
}
}

#status {
display: block;
width: 15rem;
}

.status p:first-of-type {
font-weight: bold;
animation: pulse 2s infinite;
margin: 0 0 0.5rem 0;
font-size: 1.15rem;
}

.status-online p:first-of-type {
color: #6a3cb0;
}

.status-offline p:first-of-type {
color: #b22084;
}

.status-unknown p:first-of-type {
color: grey;
}

p:last-of-type {
margin: 0;
color: #625e67;
}

@keyframes pulse {
0% {
opacity: 1;
}
50% {
opacity: 0.5;
}
100% {
opacity: 1;
}
}

.dashboard-item {
display: block;
padding: 1rem;
border: 1px solid #ccc;
border-radius: 6px;
background-color: #f8f8f8;
box-shadow: 0 1px 6px 0 rgba(0, 0, 0, 0.2);
}

.dashboard-item header {
display: flex;
padding: 0;
gap: 0.75rem;
align-items: center;
margin-bottom: 1rem;
}

.dashboard-item header img {
width: 1.5rem;
height: 1.5rem;
object-fit: contain;
}

h2 {
margin: 0;
font-size: 0.9rem;
text-transform: uppercase;
color: #504e50;
}

@media (min-width: 768px) {
.dashboard-item {
padding: 2rem;
}
}

button {
display: inline-block;
padding: 0.65rem 1.35rem;
border-radius: 0.25rem;
font-size: 1rem;
text-align: center;
cursor: pointer;
background-color: #691ebe;
color: white;
border: none;
}

button:hover {
background-color: #551b98;
}

.icon {
display: inline-block;
margin-left: 0.5rem;
transition: transform 0.2s ease-in-out;
}

button:hover .icon {
transform: translateX(4px);
}

#### Adding forms to components

- now just remove the refactored parts from the `ticket`, `traffic`, ... components

- `ng g c dashboard/tickets/new-ticket --skip-tests`
- `ng g c dashboard/tickets/ticket --skip-tests`

- new-ticket, add some form

```html
<form>
  <p>
    <label>Title</label>
    <input name="title" id="title" />
  </p>
  <p>
    <label>Request</label>
    <textarea name="title" id="title" rows="3"></textarea>
  </p>
  <p>
    <button>
      <span> Submit </span>
      <span class="icon"> |> </span>
    </button>
  </p>
</form>
```

- tickets.html

```html
<div id="new-ticket">
  <app-new-ticket />
</div>
```

- displays some form
- update the apps styles

#### Extending built-in components

- create a "shared" folder
  `ng g c shared/button --skip-tests`

- we could put the button like this here:

```html
<button>
  <span> Submit </span>
  <span class="icon"> |> </span>
</button>
```

- if we now use the `app-button` in other components, we end up with

```html
<app-button>
  <button>
    <app-button /></button
></app-button>
```

- we can make the DOM leaner
- if you just want to extend a built-in component, like the button, there is this approach with "attibute selectors"

#### Extending built-in components with attribute selectors

- instead of this

```html
<button>
  <span> Submit </span>
  <span class="icon"> |> </span>
</button>
```

- just focus on the content:

```html
<span> Submit </span> <span class="icon"> |> </span>
```

- and change its selector from `selector: 'app-button',` to `selector: 'button[appButton]',` to extend the built-in button

```ts
@Component({
  selector: "button[appButton]",
  standalone: true,
  imports: [],
  templateUrl: "./button.component.html",
  styleUrl: "./button.component.css",
})
export class ButtonComponent {}
```

- to use it: `<button>` -> `<button appButton>`
- also import `ButtonComponent`, otherwise it fails silently

```html
<li>
  <button>
    <span> Logout </span>
    <span class="icon"> → </span>
  </button>
</li>
```

#### Supporting Content Projection with Multiple Slots

- this is now hardcoded in the button

```html
<span> Submit </span> <span class="icon"> |> </span>
```

- we could accept an input for text, another for the icon

- alternatively, use `ng-content`

```html
<span>
  <ng-content />
</span>
<span class="icon">
  <ng-content />
</span>
```

- now where the button is used, like at header

```html
<li>
  <button appButton>Logout →</button>
</li>
```

- this now buts both "Logout" and the icon in the second span text, the first is empty

- add a CSS-selector to specify

```html
<ng-content /> <ng-content select=".icon" />
```

- where we use it

```html
<li>
  <button appButton>
    Logout
    <span class="icon">→</span>
  </button>
</li>
```

#### Exploring Advanced Content Projection

- if we dont want this icon class wrapped in a span

```html
<li>
  <button appButton>
    Logout
    <span class="icon">→</span>
  </button>
</li>
```

from

```html
<ng-content /> <ng-content select=".icon" />
```

to

```html
<ng-content />
<span class="icon">
  <ng-content select=".icon" />
</span>
```

but this creates two html elements with icon class

so rather ngProjectAs="icon", not ngProjectAs=".icon"

```html
<li>
  <button appButton>
    Logout
    <span ngProjectAs="icon">→</span>
  </button>
</li>
```

in the button select for this `icon` instead of `.icon`, use `<ng-content select="icon" />`

```html
<span>
  <ng-content />
</span>
<span class="icon">
  <ng-content select="icon" />
</span>
```

- on the ticket compoent

```html
<p>
  <button appButton>
    Submit
    <span class="icon">|></span>
  </button>
</p>
```

to

```html
<p>
  <button appButton>
    Submit
    <span ngProjectAs="icon">|></span>
  </button>
</p>
```

#### Defining Content Projection Fallbacks

- from `button`

```html
<span class="icon">
  <ng-content select="icon" />
</span>
```

- to use a fallback icon if no icon is defined, wrap ng-content around that fallback

```html
<ng-content select="icon"> -> </ng-content>
```

- eg. if you were to remove in `header`

```html
<button appButton>
  Logout
  <span ngProjectAs="icon">→</span>
</button>
```

- with this it uses the fallback

```html
<button appButton>
  Logout
</button>
```


#### Multi-element Custom Components & Content Projection
- create a `control` component
- we move a part of the form there
```html
  <p>
    <label>Title</label>
    <input name="title" id="title" />
  </p>
```
- set it from the outside
```ts
export class ControlComponent {
  label = input.required<string>();
}
```

```html
<p>
  <label>{{ label() }}</label>
  <input name="title" id="title" />
</p>
```

- be more retrictive with `select` to make it selective what can be projected into this
```html
<p>
  <label>{{ label() }}</label>
  <ng-content select="input, textarea"></ng-content>
</p>
```

- use the `control` component
from this
```html
  <p>
    <label>Title</label>
    <input name="title" id="title" />
  </p>
```
to
```html
  <app-control label="Title">
    <input name="title" id="title" />
  </app-control>
```

and from
```html
  <p>
    <label>Request</label>
    <textarea name="title" id="title" rows="3"></textarea>
  </p>
```
to
```html
  <app-control label="Request">
    <textarea name="title" id="title" rows="3"></textarea>
  </app-control>
```



#### Understanding & Configuring View Encapsulation

- sometimes you need to disable style encapsulation, to use the global styles for the component
- eg. you want to use the style of a button elsewhere

- the form style is broken inside control
```css
.control label {
    display: block;
    font-size: 0.8rem;
    font-weight: bold;
    margin-bottom: 0.15rem;
    color: #4f4b53;
  }
  
  .control input,
  .control textarea {
    width: 100%;
    padding: 0.5rem;
    border: 1px solid #ccc;
    border-radius: 4px;
    font: inherit;
    font-size: 0.9rem;
    color: #4f4b53;
  }
```

- in the control compoentn html file add the "control" css class
```html
<p class="control">
  <label>{{ label() }}</label>
  <ng-content select="input, textarea"></ng-content>
</p>
```

- Angular only sees the placeholder `ng-content` inside the component now, not the `input` and `ng-content` that might end up inside it

- diable the scoping of the styling

```ts
@Component({
  selector: 'app-control',
  standalone: true,
  imports: [],
  templateUrl: './control.component.html',
  styleUrl: './control.component.css',
  encapsulation: ViewEncapsulation.None
})
export class ControlComponent {
  label = input.required<string>();
}
```

- the default is `ViewEncapsulation.ShadowDom`


#### Component host elements

- the button is still broken
- every Angular component has a "Host Element", which is the element, that is selected by the selector
- eg.: `control` component -> `app-control`
- this renders
```html
<app-control>
  <p class="control">
```

- for the button: selector: 'button[appButton]', so this is its host element
- in the "button" css you can target this with
from
```css
button {
  display: inline-block
  ...
}
```
to
```css
:host {
  display: inline-block
  ...
}
```
- this allows you to apply styles to the rendered host element
- also `button:hover` -> `:host:hover`

#### Using Host elemets as Reusable elements
- we have this rendered
```html
<app-control>
  <p class="control">
```
because in the template
```html
<p class="control">
  <label>{{ label() }}</label>
  <ng-content select="input, textarea"></ng-content>
</p>
```
- we dont this nesting
- also in the css we are
```css
.control label {
    display: block;
    font-size: 0.8rem;
    font-weight: bold;
    margin-bottom: 0.15rem;
    color: #4f4b53;
  }
  
  .control input,
  .control textarea {
    width: 100%;
    padding: 0.5rem;
    border: 1px solid #ccc;
    border-radius: 4px;
    font: inherit;
    font-size: 0.9rem;
    color: #4f4b53;
  }
```

- removing the `<p class="control">` makes the styles broken, because in the css we look for .control class
```html
<p class="control">
  <label>{{ label() }}</label>
  <ng-content select="input, textarea"></ng-content>
</p>
```
to
```html
<label>{{ label() }}</label>
<ng-content select="input, textarea"></ng-content>
```

- adding `:host` selector to the css wont help because we disabled view-encapsulation with `ViewEncapsulation.ShadowDom`
- the component still has the `app-control` host element in the DOM
- but the :host css selector wont work: the css styles of this component are no longer scoped to this component, they are applied as global styles
- `:host` css selector will not work with `ViewEncapsulation.ShadowDom`

- we could add the `control` class where the `app-control` is used like in new-ticket

from
```html
<form>
  <app-control label="Title">
    <input name="title" id="title" />
  </app-control>
```
to
```html
<form>
  <app-control class="control" label="Title">
    <input name="title" id="title" />
  </app-control>
```