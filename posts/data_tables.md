# A Data Table view component

Here at Zaikio, we're building a next-generation cloud platform for the print industry.
Our primary technology choice is Ruby on Rails, which also allows us to leverage the wider
Ruby ecosystem.

One of our favourite developments in recent years is the view_component gem, headed up by
GitHub, which provides a great way to build and share reusable interface components. Since
January 2022, we've been migrating our existing Rails helpers to ViewComponents for all
new interface development, and now we're keen to share some of those with you!

This time, we'd like to talk about data tables.

## What is a "data table"?

Broadly speaking, a "data table" is just a UI component which encapsulates the display of
a collection of data. It could look like a simple HTML table, but it might offer some
advanced features like pagination, sorting and filtering.

There's a [great article over on Pencil &
Paper](https://pencilandpaper.io/articles/ux-pattern-analysis-enterprise-data-tables)
which talks about common design patterns, pitfalls as well as some great examples of data
tables in practice.

## Basic setup

For this article, let's assume the following data model in our
application:

```ruby
create_table :animals do |t|
  t.text :name
  t.text :category
end

class Animal < ActiveRecord::Base; end

Animal.create!(name: "Tony", category: "tiger")
Animal.create!(name: "Nelly", category: "elephant")
```

We'd like to present this information in a simple table, like so:

| Name  | Category |
|-------|----------|
| Tony  | tiger    |
| Nelly | elephant |

In a world before view components, our initial instinct might be to do something like this:

```erb
# index.html.erb
<table>
  <thead>
    <th>Name</th>
    <th>Category</th>
  </thead>

  <tbody>
    <% render collection: @animals %>

# _animal.html.erb
<tr>
  <td><%= animal.name %></td>
  <td>
    <span class="tag"><%= animal.category.upcase %></span>
  </td>
</tr>
```

However, there are a couple of problems with this approach:

* The table container, and the rows, are in separate files, which makes it easy to
  accidentall add, change or remove columns from the table without updating the other
  file.
* It's difficult to share styles between the same sorts of table in different places.
* Introducing sorting or filtering is going to make the index file much more complicated.

## Converting to a ViewComponent

I like to start a new component with the interface/examples first, and then work backwards
to flesh out the implementation. My goals for the component are:

* Can be easily setup using a collection (e.g. an ActiveRecord collection from a
  controller)
* If the column is simply rendering an attribute of the same name, do that
* If you need more complex rendering options, like buttons or wrapping elements, you can
  pass a block which is executed for every row.

So, my first pass at the design looked like this:

```erb
# index.html.erb
<%= render DataTableComponent.new(@animals) do |t| %>
  <% t.column :name %>

  <% t.column :category do |animal| %>
    <span class="tag"><%= animal.category.upcase %></span>
  <% end %>
<% end %>
```

## Problem #1: blocks don't work like that
