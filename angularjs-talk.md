# Encapsulation with expression locals

I'm going to tell you about a pattern I think is useful and underused.

- **Me / Intro / Klara** - *who I am, what I do, where I work*
- **The Problem / Demo** - *what problem does this solve?*
- **The Pattern** - *what's the pattern?*
- **The Code** - *show me the code!*

## Me / Intro / Klara

My name's **Will Madden**, I'm the Head of Product at Klara ([w.madden@klara.com](mailto:w.madden@klara.com)).

Before that I wrote code :/ - specifically **Angular** (in CS) and **Ruby** (on Rails).

**Klara** is a service that lets doctors extend the same service they offer in their practice to their patients online.

- Native iOS and Android apps **for patients**
- A web client **for doctors** (using Bootstrap/Marionette)
- API in Ruby (with a splash of Rails)

We're looking for outstanding **JS and Ruby devs**.

## The Problem / Demo

```html
<infinite-scroll provider="dataProvider"
                 fetch-when="remainingDistance < containerHeight / 2">
  <ul class="list-group">
    <li class="list-group-item" ng-repeat="item in items">
      {{ item.description }}
    </li>
  </ul>
</infinite-scroll>
```

Directive knows:
- Which items have been fetched from the data provider
- The size of the rendered list
- How far down it the user has scrolled

Caller knows:
- How to render the list
- Where the data comes from
- How long it'll take to fetch more items
- When it's appropriate to begin fetching

<iframe style="width: 100%; height: 300px;" src="http://wmadden.github.io/cf-challenge/"></iframe>

## The Pattern

```html
<infinite-scroll fetch-when="remainingDistance < containerHeight / 2">
</infinite-scroll>
```

- `fetch-when` is an expression evaluated by the directive to check if it should fetch more items or not
- The logic is implemented by the caller, in scope
- The directive selectively exposes relevant information to the caller
- The directive remains ignorant of the actual logic
- The caller remains ignorant of the details of populating and scrolling through the list
- The directive remains ignorant of how the list is rendered

## The Code

```javascript
// (Directive boilerplate here)
link: function($scope, $element, $attrs) {
  // ...
  function onScroll() {
    var containerHeight            = $element.height();
    var lengthOfContentAlreadySeen = $element.scrollTop() + containerHeight;
    var remainingDistance          = getTotalContentHeight() - lengthOfContentAlreadySeen;

    $scope.$apply(function() {
      if ( weShouldFetchMoreItems(remainingDistance, containerHeight) ) {
        fetchItems();
      }
    });
  }

  function weShouldFetchMoreItems(remainingDistance, containerHeight) {
    return $scope.$eval($attrs.fetchWhen, {
      remainingDistance: remainingDistance,
      containerHeight:   containerHeight
    });
  }
  // ...
}
```
