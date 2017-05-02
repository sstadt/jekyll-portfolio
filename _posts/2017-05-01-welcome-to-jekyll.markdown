---
layout: default
title: "One Post"
date: 2017-05-01 15:07:56 -0500
links:
  - name: Blog
    route: /blog
---
Hi, I'm an excerpt. Apparently you can't start a post off with a highlight! Good to know!

{% highlight javascript %}
var foo = 'foo';
var num = 437280491;

(function ($) {
  $(document).ready(function () {
    var showCalendarGrid = sessionStorage.showCalendarGrid || false,
      initialView = showCalendarGrid ? 'grid' : 'list';

    // initialize calendar view
    $(`.js-calendar-tab[data-type="${initialView}"]`).fadeIn('fast');
    $(`.js-calendar-nav[data-target="${initialView}"]`).addClass('active');
    delete sessionStorage.showCalendarGrid;

    $('.js-month-controls').click(function (event) {
      var $btn = $(this),
        link = $btn.attr('href');

      event.preventDefault();

      sessionStorage.showCalendarGrid = true;
      window.location.href = link;
    });

    // calendar view toggle
    $('.js-calendar-nav').click(function (event) {
      var $btn = $(this),
        target = $btn.data('target');

      event.preventDefault();

      $('.js-calendar-nav.active').removeClass('active');
      $btn.addClass('active');

      $(`.js-calendar-tab[data-type="${target}"]`).each(function () {
        var $tab = $(this);

        $tab.siblings('.js-calendar-tab:visible').fadeOut('fast', () => $tab.fadeIn('fast'));
      });
    });

    $('.js-calendar-filter-category-select').change(function () {
      var $categorySelect = $(this),
        $filterSelect = $('.js-calendar-filter-select'),
        filters = $categorySelect.find('option:selected').data('filters'),
        newOptions = '';

      if (filters && filters.length) {
        for (var i = 0, j = filters.length; i < j; i++) {
          newOptions += `<option value="${filters[i].slug}">${filters[i].name}</option>`;
        }
      }

      $filterSelect.find('option:not(:first-child)').remove();
      $filterSelect.append(newOptions);
    });

    $('.js-calendar-filter-select').change(function () {
      var baseLink = $('.js-reset-filters').attr('href'),
        category = $('.js-calendar-filter-category-select').val(),
        filter = $(this).val();

      if (filter !== 'all') {
        window.location = `${baseLink}/${category}/${filter}`;
      }
    });

  });
}(window.jQuery));
{% endhighlight %}

{% highlight html %}
<!DOCTYPE html>
<html>
  <head>
    <title>ScottStadt.com - Welcome</title>
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">

    <meta property="og:url" content="https://www.scottstadt.com" />
    <meta property="og:type" content="website" />
    <meta property="og:title" content="ScottStadt.com" />
    <meta property="og:image" content="//placehold.it/500x500" />
    <meta property="og:description" content="Lorem ipsum dolor sit amet." />
    <meta property="description" content="Lorem ipsum dolor sit amet." />

    <link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Abel|Ubuntu:300,400i,700,700i">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/normalize/6.0.0/normalize.min.css" />
    <link rel="stylesheet" href="/css/style.css" />

  </head>

  <body>
    <div class="container">
      <div class="header">
        <h1>Welcome.</h1>

        <ul class="nav">
          <li><a href="/foo">Foo</a></li>
          <li><a href="/bar">Bar</a></li>
          <li><a href="/baz">Baz</a></li>
        </ul>
      </div>

      <div class="content">
        <p>Lorem ipsum dolor sit amet</p>
      </div>
    </div>
  </body>

</html>
{% endhighlight %}

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}
