---
title: "Using Angular’s EventEmitter to Share Data Between Child and Parent Components"
date: 2021-08-08 20:00:00:00 +00:00
author: steven
layout: post
image: /assets/img/2021/08/angular-to-do-app-dialog.png
icon: angular
tags: angular
---

There are often scenarios in Angular where we need to communicate between components, specifically to send data from a child component to it's parent component. Consider this Angular 12 app for managing a to do list:

{%
    include image.html
    year='2021'
    month='08'
    file='angular-to-do-app.png'
    alt='Angular To Do App'
%}

When saving a new to do to the list, I'm using the **child component** *(the dialog window)* to send the new to do to an API to be stored in a database. In addition to this, I need to communicate to the **parent component** *(the to do list)* that the new to do was saved and add it to the list:

{%
    include image.html
    year='2021'
    month='08'
    file='angular-to-do-app-dialog.png'
    alt='Angular To Do App - Save Dialog'
%}

I'll begin by demonstrating the current Angular app ***(currently without an EventEmitter)*** and explain how we can implement an EventEmitter to send data between the child and parent components.

#### app.component.ts *(without EventEmitter)*

In **app.component.ts** *(the parent component)* I'm retrieving the list of to dos from an API and assigning them to an array:

```typescript
import { Component, OnInit } from '@angular/core';
import { Title } from '@angular/platform-browser';
import { ToDoListService } from './services/to-do-list.service';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
  public title = 'To-Do List';
  public toDoList: string[] = [];

  constructor(
    private titleService: Title,
    private toDoListService: ToDoListService) {}

  ngOnInit(): void {
    this.titleService.setTitle(this.title);
    this.toDoList = this.toDoListService.getToDoList();
  }
}
```

#### app.component.html *(without EventEmitter)*

In **app.component.html** I'm looping through the array of to dos and referencing **SaveNewToDoComponent** *(the child component)* that is being used as a dialog window to save a new to do:

```html
<nav class="navbar navbar-expand-lg navbar-dark bg-primary">
    ...
</div>

<div class="container mt-4">
  <div class="row">
    <div class="col-md-12">
      <div class="card">
        <div class="card-body">
          <h5 class="card-title">To Do List</h5>
          <p class="card-text"><small>View all of the to-dos or save a new to-do to the list.</small></p>
          <button type="button" class="btn btn-primary mt-1 mb-4" data-toggle="modal" data-target="#toDoModal">
            Save New To Do
          </button>
          <ul class="list-group">
            <li class="list-group-item list-group-item-action" *ngFor="let toDo of toDoList">{{ toDo }}</li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</div>

<app-save-new-to-do></app-save-new-to-do>
```

#### save-new-to-do.ts *(without EventEmitter)*

In **save-new-to-do.ts** *(the child component)* I'm retrieving the new to do from the form and sending it to an API to be stored in a database:

```typescript
import { Component, EventEmitter, OnInit, Output } from '@angular/core';
import { FormBuilder } from '@angular/forms';
import { ToDoListService } from 'src/app/services/to-do-list.service';

@Component({
  selector: 'app-save-new-to-do',
  templateUrl: './save-new-to-do.component.html',
  styleUrls: ['./save-new-to-do.component.css']
})
export class SaveNewToDoComponent implements OnInit {
  formGroup: any;

  constructor(
    private formBuilder: FormBuilder,
    private toDoListService: ToDoListService) { }

  ngOnInit(): void {
    this.formGroup = this.formBuilder.group({
      toDo: ''
    });
  }
  
  public saveNewToDo(formData: any) {
    let toDo = formData['toDo'];
    this.toDoListService.saveNewToDo(toDo);
  }
}
```

## Adding EventEmitter to the Angular App

I'll add an EventEmitter by adding an output decorator to the **SaveNewToDoComponent** class:

```typescript
@Output() saveToDoEvent = new EventEmitter<string>();
```

Also in the **SaveNewToDoComponent** class, I'll emit an event containing the new to do once the to do has been sent to the API:

```typescript
this.saveToDoEvent.emit(toDo);
```

#### save-new-to-do.ts  *(with EventEmitter)*

Below is the entire component **save-new-to-do.ts** using an EventEmitter:

```typescript
import { Component, EventEmitter, OnInit, Output } from '@angular/core';
import { FormBuilder } from '@angular/forms';
import { ToDoListService } from 'src/app/services/to-do-list.service';

@Component({
  selector: 'app-save-new-to-do',
  templateUrl: './save-new-to-do.component.html',
  styleUrls: ['./save-new-to-do.component.css']
})
export class SaveNewToDoComponent implements OnInit {
  formGroup: any;
  @Output() saveToDoEvent = new EventEmitter<string>();

  constructor(
    private formBuilder: FormBuilder,
    private toDoListService: ToDoListService) { }

  ngOnInit(): void {
    this.formGroup = this.formBuilder.group({
      toDo: ''
    });
  }
  
  public saveNewToDo(formData: any) {
    let toDo = formData['toDo'];
    this.toDoListService.saveNewToDo(toDo);
    this.saveToDoEvent.emit(toDo);
  }
}
```

Within **app.component.html**, I'll use event binding to communicate the event occurring in the **child component** and send the new to do to the **parent component** by executing the new method *'appendToDo'*, passing in the new to do as a parameter:

```html
<app-save-new-to-do (saveToDoEvent)="appendToDo($event)"></app-save-new-to-do>
```

Finally, I'll add the new method *'appendToDo'* to the **parent component** to push the new to do to the array:

```typescript
public appendToDo(toDo: string) {
    if (toDo?.length > 0) {
        this.toDoList.push(toDo);
    }
}
```

Now that I've added an EventEmitter into the Angular app, the list will be automatically updated with the new to do:

{%
    include image.html
    year='2021'
    month='08'
    file='angular-to-do-app-last-step.png'
    alt='Angular To-Do App - Last Step'
%}

## GitHub Repository

Thank you for reading this article! The full solution used in this guide is available at [github.com/stevenmclintock/angular-event-emitter-example](https://github.com/stevenmclintock/angular-event-emitter-example).