---
layout: post
title: "Using AngularJS for existing Rails app"
date: 2013-05-03 21:05
comments: true
keywords: AngularJS, Rails, Ruby
description: How to using AngularJS for existing Rails app
categories: [AngularJS, Rails, Ruby]
---

In one of my projects, we were tasked to put AngularJS on top of an existing Rails app.  We wanted to keep the business impact at a minimum.  This means that we had to gradually make the transition while adding more end-user features.

Since retro fitting a client side framework is pretty common for many Rails apps, I'm going to share some of the problems I've encoutered, and possible approaches I used.

## HAML or HTML?

Our views were written in haml and generated from the server side. I liked it and would like to continue using it for generating view markup. Unfortunatelly, we didn't find a javascript library that work *exact* the same as ruby's haml.

We ended up created a `TemplatesController` and route all of the template fetching requests though this `TemplatesController`. From browser's perspective, we are still serving static HTML, but we were able to write it in HAML from the server side.

```ruby
# TemplatesController
class TemplatesController
  def show
    path = params[:path]
    render file: "/views/templates/#{path}", handler: [:haml], layout: false
  end
end
```

```ruby
# router.rb
# routes template/* to a template controller
match '/template/*path' => 'templates#show'
```

One added benefit was that because we are still on the server side, we have access to things like `User::ROLES` and `I18n.t 'store.title'`, which we otherwise have to duplicate at the client side.

## Do we have to write validations twice?

I really like the ActiveRecord error messages. It has good defaults, easily configurable, and intergate well with i18n. At the same time, I want the good user experience that’s provided by the client-side validation. Refreshing the page to see errors is so 2008.

I also don't want to write our validations and messages twice. The server side already has validations, we just need to pipe the error messages out to the page via ajax. Its easy with AngualrJS

{% raw %}
```html
<form action='/users', methpd='post'>
  <fieldset>
    <label>Name</label>
    <input type='text' ng-model='name' />
    <div>{{errors['name']}}</div>
  </fieldset>
  <input type='submit' ng-click='submit' />
</form>
```
{% endraw %}

```javascript
function FooCtrl ($scope) {
  $scope.submit = function () {
    httpRequest({}, scope.success, scope.setErrors)
  };

  $scoppe.setErrors = function (response) {
    scope.errors = response.data.errors
  }
}

```

Every time the user clicks submit, without any client-side validation, the form issues an ajax request. We rely on ActiveRecord to do the validation, feed us with the correct error messages, and store the errors in an errors object in javascript, with Angular's two-way binding. The view then gets updated automatically.


Note: The approach works for us becuase our application only validates the input after the user attempts to submit the form. It might not work well in your case.


## Turn functions into services
We use CanCan for access control. In the view, we show and hide certain elements depending on the current user's role. The code snippet below hides the Edit button if `current_user` does not have the `ability` to `manage` `Post`.

```haml
  - if current_user.ability.can?(:manage, Post)
    <button>Edit</button>
```

Since we are switching to client side templates, we don't have direct access to `ability` object anymore, but we can turn the CanCan ability into a service call. A get request to `/abilities.json` will return something like this for the current user:

```json
{
  "manage": {
    "User": true,
    "Post": false,
    ...
  },
  "read": {
    "User": true,
    "Post": true
    ...
  },
  "Update": {
    "User": true,
    "Post": false
  }
  ...
}
```

In the view, we can choose to show/hide elements based on the user's role again.

```html
  <button ng-show="canManage('User')">
```

```javascript
  function FooCtrl () {
    scope.canManage = function (model) {
      scope.manage[model]
    };
  }
```
