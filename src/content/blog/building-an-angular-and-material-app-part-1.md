---
author: Ryan Jordan
pubDatetime: 2017-03-08T15:07:00Z
title: Building an Angular & Material App Part I
slug: building-an-angular-and-material-app-part-1
draft: false
tags:
  - angular
  - material
  - angular-cli
description: I thought it would be fun to write an app that uses the [Punk API](https://punkapi.com/) with the [Angular CLI](https://cli.angular.io) and [Angular Material](https://material.angular.io/). My plan is to build this app over the span of a few blog posts. Part I will focus on the set up of Angular, Angular Material.
---

I thought it would be fun to write an app that uses the [Punk API](https://punkapi.com/) with the [Angular CLI](https://cli.angular.io) and [Angular Material](https://material.angular.io/). My plan is to build this app over the span of a few blog posts. Part I will focus on the set up of Angular, Angular Material. At the end of this tutorial, we will have an app that displays a list of beers that we can page through. Or if you want to just view the finished product, you can view the [repo](https://github.com/krjordan/brewski-catalogue) or the [live](https://krjordan.github.io/brewski-catalogue/) demo.

<!--more-->

## Prerequisites

I'm going to use the [Angular CLI](https://cli.angular.io/) with [Yarn](https://yarnpkg.com/) to get up and running. So, if you don't already have them installed you can do so by running:

```language-none
npm install -g @angular/cli yarn
```

### Versions

- Yarn: `^0.27.5`
- Angular: `^4.0.0`
- Angular-CLI: `1.2.0`
- Angular Material: `2.0.0-beta.7`

## Getting Started

Let's get to it.

```language-none
ng set --global packageManager=yarn
ng new brewski-catalogue --style scss
```

This creates a new Angular project called <code class="language-none">brewski-catalogue</code> and then installs our dependencies using Yarn. Once complete, we can type <code class="language-none">ng serve</code> or <code class="language-none">yarn start</code> and point our browser to <code class="language-none">localhost:4200</code>. You should see "**app works!**" displayed on the page.

## Suds Service

We will need a service to make an http request and retrieve the list of beers to use in our app. Let's do that now by making a new directory called <code class="language-none">services</code> at <code class="language-none">src/app/services</code> and then use the Angular CLI to generate our service.

```language-none
mkdir src/app/services
ng generate service services/suds
```

This creates two files for you in the <code class="language-none">src/app/services</code> directory and imports the service. Let's take a look at the <code class="language-none">suds.service.ts</code> and add our http request.

```typescript
import "rxjs/add/operator/map";
import { Injectable } from "@angular/core";
import { Http, Response } from "@angular/http";

@Injectable()
export class SudsService {
  API_PATH: string = "https://api.punkapi.com/v2";
  MAX_PER_PAGE: number = 10;

  constructor(private http: Http) {}

  getSuds() {
    return this.http
      .get(`${this.API_PATH}/beers?per_page=${this.MAX_PER_PAGE}`)
      .map((res: Response) => res.json());
  }
}
```

Notice that I'm limiting our response to 10 beers per page. Next, we will need to add our services to the <code class="language-none">src/app.module.ts</code>:

```typescript
// All of our other imports
...
// Import the services here
import { SudsService } from './services';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    FormsModule,
    HttpModule
  ],
  providers: [
    // Add your services here
    SudsService
  ],
  bootstrap: [AppComponent]
})
export class AppModule{ }
```

## Beers List Component

Let's add a component that will display our list of beers that we get back from the http request. Again, we will first have to make a directory for all of our components to live.

```language-none
mkdir src/app/components
ng generate component components/beer-list
```

Let's add our beer-list component to the <code class="language-none">app.component.html</code> so we can see it.

```html
<h1>{{ title }}</h1>
<app-beer-list></app-beer-list>
```

Now you should see "**app works!**" along with "**beer list works!**" in the browser. So lets test out our beer service and display some beer! Inside our <code class="language-none">beer-list.component.ts</code> file lets use the beer service to spit out the json that gets returned back from our service. The <code class="language-none">beer-list.component.ts</code> should look like this.

```typescript
import { Component, OnInit } from "@angular/core";
import { SudsService } from "../../services/suds.service";

@Component({
  selector: "app-beer-list",
  templateUrl: "./beer-list.component.html",
  styleUrls: ["./beer-list.component.css"],
})
export class BeerListComponent implements OnInit {
  suds: any[];

  constructor(private sudsService: sudsService) {}

  ngOnInit() {
    this.sudsService.getSuds().subscribe(suds => (this.suds = suds));
  }
}
```

And in our template (<code class="language-none">beer-list.component.html</code>) for now let's just put <code class="language-markup">{{ suds | json }}</code>. You should now have a huge json list of beers displayed in your browser.

## Angular Material

At this point, I'm going to stop where we are and take some time to make our app more visually appealing. I'm going to add Angular Material to the project to help us do that. Let's install it, <code class="language-none">yarn add @angular/material</code>. Next, we'll add the material module to our <code class="language-none">app.module.ts</code> file.

```typescript
// Other imports
...
import { MaterialModule } from '@angular/material';

@NgModule({
  declarations: [...],
  imports: [
    ...
    HttpModule,
    // Add Angular Material inside our imports
    MaterialModule
  ],
  providers: [
    services
  ],
  bootstrap: [AppComponent]
})
export class AppModule{ }
```

Don't forget to add a comma to the import before our <code class="language-none">MaterialModule</code>. Then you will need to add <code class="language-typescript">@import "~@angular/material/prebuilt-themes/indigo-pink.css";</code> to the <code class="language-none">src/styles.scss</code> file. Once that's complete, we can start building our beer-list component.

Lets remove the <code class="language-markup">{{ beers | json }}</code> from our <code class="language-none">beer-list.component.html</code> and replace it with a material card layout:

```html
<md-card *ngFor="let beer of beers">
  <md-card-header>
    <md-card-title>{{ beer.name }}</md-card-title>
    <md-card-subtitle>{{ beer.tagline }}</md-card-subtitle>
  </md-card-header>
  <md-card-content>
    <p>{{ beer.description }}</p>
  </md-card-content>
  <md-card-actions>
    <button md-button color="primary">VIEW</button>
  </md-card-actions>
</md-card>
```

And drop a few styles into the <code class="language-none">beer-list.component.scss</code>:

```css
md-card {
  max-width: 37.5rem;
  margin: 0.9375rem auto;
}
p {
  margin: 0 0.5rem;
}
```

This should give our component a bit of a makeover. I've limited the cards to <code class="language-css">600px</code> and centered them. The <code class="language-none">VIEW</code> button will take us to a more detailed page that we will set up later.

Now lets go into our <code class="language-none">app.component.html</code> file and give it some pizzaz.

```html
<md-toolbar color="primary">{{ title }}</md-toolbar>
<app-beer-list></app-beer-list>
```

We will also need to update the title in our <code class="language-none">app.component.ts</code> to <code class="language-none">Angular Brewski Catalogue</code>.

At this point we should have our beer app looking just peachy. Everything works as it should, but we only have a list of 10 beers. We need to figure out a way to paginate through our list of beers. So lets add a <code class="language-none">beer-list-controls</code> component.

```language-none
ng g c components/beer-list-controls
```

Let's add them to our <code class="language-none">beer-list.component.html</code> template:

```html
<md-card *ngFor="let beer of beers>
 ...
</md-card>
<app-beer-list-controls></app-beer-list-controls>
```

Open up the <code class="language-none">src/app/components/beer-list-controls/beer-list-controls.ts</code> and add the following code:

```typescript
import { Component, OnInit } from "@angular/core";

@Component({
  selector: "app-beer-list-controls",
  templateUrl: "./beer-list-controls.component.html",
  styleUrls: ["./beer-list-controls.component.css"],
})
export class BeerListControlsComponent implements OnInit {
  constructor() {}

  ngOnInit() {}

  nextPage() {}

  prevPage() {}
}
```

Add two buttons to our component with Angular Material:

```html
<button md-raised-button color="primary" (click)="prevPage()">Previous</button>
<button md-raised-button color="primary" (click)="nextPage()">Next</button>
```

And add some styles to make it look presentable:

```css
:host {
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 1.5625rem 0;
}

button {
  margin: 0 0.625rem;
}
```

Our buttons look great, but they don't actually do anything yet. Lets hook everything up. We will need a way to keep track with the current page number along with a way to increment and decrement that variable. So let's add that to our <code class="language-none">src/app/components/beer-list-controls.component.ts</code>.

```typescript
import { Component, OnInit } from "@angular/core";

@Component({
  selector: "app-beer-list-controls",
  templateUrl: "./beer-list-controls.component.html",
  styleUrls: ["./beer-list-controls.component.scss"],
})
export class BeerListControlsComponent implements OnInit {
  currentPage = 1;

  constructor() {}

  ngOnInit() {}

  prevPage() {
    this.currentPage--;
  }

  nextPage() {
    this.currentPage++;
  }
}
```

You can now <code class="language-javascript">console.log(this.currentPage)</code> inside the <code class="language-javascript">prevPage()</code> and <code class="language-javascript">nextPage()</code> methods and you will see the variable increment and decrement when we click on our buttons. Now we need a way to tell our Beer List Component that we want to update the page number. So lets add an event emitter, and an Output decorator. I'm also going to go ahead and hook the event emitter up to be used on a method called <code class="language-javascript">updatePage()</code> that we will create in a second.

```typescript
import { Component, OnInit, EventEmitter, Output } from '@angular/core';

...
...
export class BeerListControlsComponent implements OnInit {
  @Output() updatePage = new EventEmitter();
  currentPage = 1;

  ...

  prevPage() {
    this.currentPage--;
    this.updatePage.emit(this.currentPage);
  }

  nextPage() {
    this.currentPage++;
    this.updatePage.emit(this.currentPage);
  }
}
```

Now when our previous or next buttons are clicked, our component methods will be called to decrement/increment our property and then emit an event to our <code class="language-typescript">BeerListComponent</code>. Now lets add our <code class="language-javascript">updatePage()</code> method to the <code class="language-typescript">BeerListComponent</code>.

```typescript
// Change our OnInit method to call updatePage
ngOnInit() {
  this.updatePage();
}

updatePage(page?: number) {
  this.sudsService.getSuds(page)
    .subscribe(suds => this.beers = suds);
}
```

Notice, I refactored our code so that the <code class="language-typescript">ngOnInit()</code> method calls the <code class="language-typescript">updatePage()</code> method. I have also added an optional parameter (note the <code class="language-none">?</code>). Now lets edit our template to finish wiring up our event emitter.

```markup
<app-beer-list-controls (updatePage)="updatePage($event)"></app-beer-list-controls>
```

The last thing we need to do is fix our service to use a default parameter. In the <code class="language-none">src/app/services/suds.service.ts</code>, edit the <code class="language-typescript">getSuds()</code> method to take a parameter with a default value of 1 and add the parameter to our url:

```typescript
getSuds(page: number = 1) {
  return this.http.get(`${this.API_PATH}/beers?page=${page}&per_page=${this.MAX_PER_PAGE}`)
    .map((res: Response) => res.json());
}
```

And there we have it, we should now be able to change the page of beers.

Part II will focus on fixing/adding our tests. You can view the [code](https://github.com/krjordan/brewski-catalogue) on GitHub or the [live](https://krjordan.github.io/brewski-catalogue/) demo of the completed tutorial (currently only up to part I).

I hope this tutorial has been helpful. If you spot errors or would like to suggest an edit, please feel free to [open an issue or submit a pull request to this article on GitHub](https://github.com/krjordan/mycodingblog/blob/master/content/post/building-an-angular-and-material-app-part-1.md).
