---
author: Ryan Jordan
pubDatetime: 2020-04-23T02:02:00Z
title: Mocking out Angular's ActivatedRoute for Unit Tests
slug: mocking-out-angulars-activatedroute-for-unit-tests
draft: false
tags:
  - angular
  - unit-tests
  - angular-cli
description: I've been writing a ton of unit tests and snapshot tests lately. While doing so, I have had to mock out a few things. This includes the `ActivatedRoute`. So let's say we have the following component.
---

I've been writing a ton of unit tests and snapshot tests lately. While doing so, I have had to mock out a few things. This includes the `ActivatedRoute`. So let's say we have the following component.

<!--more-->

```typescript
@Component({
  selector: "app-test",
  templateUrl: "./test.component.html",
  styleUrls: ["./test.component.scss"],
})
export class TestComponent implements OnInit {
  importantData: any;

  constructor(private route: ActivatedRoute) {}

  ngOnInit(): void {
    this.importantData = this.route.snapshot.data["importantData"];
  }
}
```

In order to mock the data out, you will need to use your mocked router object. So your test should look something like:

```typescript
import { TestBed, async, ComponentFixture } from "@angular/core/testing";
import { TestComponent } from "./test.component";
import { ActivatedRoute } from "@angular/router";

describe("TestComponent", () => {
  let component: TestComponent;
  let fixture: ComponentFixture<TestComponent>;
  const activatedRouteMock = {
    snapshot: {
      data: {
        importantData: {
          content: "Really Important String",
        },
      },
    },
  };

  beforeEach(async(() => {
    TestBed.configureTestingModule({
      declarations: [TestComponent],
      providers: [{ provide: ActivatedRoute, useValue: activatedRouteMock }],
    })
      .compileComponents()
      .then(() => {
        fixture = TestBed.createComponent(TestComponent);
        component = fixture.componentInstance;
      });
  }));

  it("should be defined", () => {
    expect(component).toBeDefined();
  });

  it("should render the component", () => {
    fixture.detectChanges();
    expect(fixture).toMatchSnapshot();
  });
});
```

We can take this a step further and mock out queryParams as well. In our `OnInit` method we would have something like:

```typescript
this.route.queryParams.subscribe(params => {
  this.moreImportantData.firstEntry = params["firstEntry"];
  this.moreImportantData.secondEntry = params["secondEntry"];
});
```

In order to set the params we would simply mock it out like so:

```typescript
const activatedRouteMock = {
  queryParams: of({
    firstEntry: "This is the first entry",
    secondEntry: "This is the second entry",
  }),
};
```

Mocking out the `ActivatedRoute` is fairly straightforward. But just like every skill you learn, if you haven't written any tests in a while or mocked it out, a quick reference like this is very helpful!
