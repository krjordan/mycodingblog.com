---
author: Ryan Jordan
pubDatetime: 2018-09-13T17:05:00Z
title: Reference Dom Elements in Angular
slug: reference-dom-elements-in-angular
draft: false
description: Sometimes you need to reference a DOM element inside an angular template. We can do this with JavaScript using the `document.getElementById('title');`, however this is considered bad practice.
---

Sometimes you need to reference a DOM element inside an angular template. We can do this with JavaScript using the `document.getElementById('title');`, however this is considered bad practice.

In Angular, there are three solutions that I'm aware of. The first is what I mentioned above and **not recommended**.

### Bad

```html
<h1 id="title">This is my title</h1>
```

```typescript
...
export class AppComponent implements OnInit {
  ngOnInit(): void {
    const title = document.getElementById('title');
    title.style.color = 'pink';
  }
}
```

As you can see this changes the color of the `h1` to `pink`. The second way is to inject the document into the component.

### Better

```typescript
import { Component, OnInit, Inject } from "@angular/core";
import { DOCUMENT } from "@angular/common";

@Component({
  selector: "app-root",
  templateUrl: "./app.component.html",
  styleUrls: ["./app.component.scss"],
})
export class AppComponent implements OnInit {
  constructor(@Inject(DOCUMENT) private _document: any) {}

  ngOnInit(): void {
    const title = this._document.getElementById("title");
    title.style.color = "blue";
  }
}
```

### Best

Finally, the path I usually try to take using `ViewChild`.

```html
<h1 #title>This is my title</h1>
```

```typescript
import { Component, OnInit, ViewChild } from "@angular/core";

@Component({
  selector: "app-root",
  templateUrl: "./app.component.html",
  styleUrls: ["./app.component.scss"],
})
export class AppComponent implements OnInit {
  @ViewChild("title")
  title;

  ngOnInit(): void {
    this.title.nativeElement.style.color = "orange";
  }
}
```
