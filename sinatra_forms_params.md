# Sinatra Forms and Params – Beginner to Intermediate Guide

Sinatra is a lightweight Ruby web framework that makes handling forms and user input straightforward. This guide covers how to create forms, access parameters, handle nested data, and more.

---

## 1. Introduction to Forms in Sinatra

Forms allow users to submit data to your Sinatra application. Sinatra automatically parses form data into a `params` hash.

A basic form looks like this:

```erb
<!-- views/form.erb -->
<form action="/submit" method="post">
  <label for="name">Name:</label>
  <input type="text" name="name" id="name">

  <label for="age">Age:</label>
  <input type="number" name="age" id="age">

  <button type="submit">Submit</button>
</form>
```

**Key points:**
- `action` → the route where the form is submitted.
- `method` → HTTP method (GET or POST).
- `name` → the key used in the `params` hash.

---

## 2. Accessing Params in Sinatra

When a form is submitted, Sinatra stores all data in the `params` hash.

```ruby
# app.rb
require 'sinatra'

get '/' do
  erb :form
end

post '/submit' do
  name = params[:name]
  age  = params[:age]

  "Hello #{name}, you are #{age} years old!"
end
```

**Notes:**
- `params` works for both GET query strings and POST form bodies.
- You can access values using symbols (`params[:name]`) or strings (`params['name']`).

---

## 3. GET vs POST Forms

### GET Form
```erb
<form action="/search" method="get">
  <input type="text" name="query">
  <button type="submit">Search</button>
</form>
```

- Data is appended to the URL: `/search?query=sinatra`
- Useful for search forms or links.

### POST Form
```erb
<form action="/submit" method="post">
  <input type="text" name="username">
  <button type="submit">Submit</button>
</form>
```

- Data is sent in the request body.
- Recommended for sensitive or large data.

---

## 4. Nested Params

You can group related form fields using square brackets:

```erb
<form action="/user" method="post">
  <input type="text" name="user[name]" placeholder="Name">
  <input type="email" name="user[email]" placeholder="Email">
  <button type="submit">Create User</button>
</form>
```

In Sinatra:

```ruby
post '/user' do
  user_name  = params[:user][:name]
  user_email = params[:user][:email]

  "User #{user_name} with email #{user_email} created!"
end
```

---

## 5. File Uploads

Forms that upload files require `enctype="multipart/form-data"`:

```erb
<form action="/upload" method="post" enctype="multipart/form-data">
  <input type="file" name="avatar">
  <button type="submit">Upload</button>
</form>
```

In Sinatra:

```ruby
post '/upload' do
  file = params[:avatar][:tempfile]
  filename = params[:avatar][:filename]

  File.open("./uploads/#{filename}", "wb") do |f|
    f.write(file.read)
  end

  "File uploaded successfully!"
end
```

---

## 6. Common Tips

1. **Symbols vs Strings:**  
   `params[:key]` and `params['key']` both work, but symbols are preferred in Ruby.

2. **Default Values:**
   ```ruby
   name = params[:name] || "Guest"
   ```

3. **Multiple Values:**  
   Use `name="colors[]"` for checkboxes to get an array:
   ```ruby
   params[:colors] # => ["red", "blue"]
   ```

4. **Security:**
   - Always validate user input.
   - Escape HTML if displaying raw input.

---

## 7. Full Example

```ruby
require 'sinatra'

get '/' do
  erb :form
end

post '/submit' do
  name  = params[:name]
  age   = params[:age]
  email = params[:user][:email] rescue nil

  "Hello #{name}, age #{age}, email: #{email || 'not provided'}"
end
```

```erb
<!-- views/form.erb -->
<form action="/submit" method="post">
  <label>Name:</label>
  <input type="text" name="name">

  <label>Age:</label>
  <input type="number" name="age">

  <label>Email:</label>
  <input type="email" name="user[email]">

  <button type="submit">Submit</button>
</form>
```

This example covers:
- Basic form submission
- Nested params
- Optional fields
- Combining GET/POST style understanding

---

## 8. References

- [Sinatra Official Docs](http://sinatrarb.com/)
- [Sinatra Forms Guide](http://sinatrarb.com/intro.html#Forms)
- [Ruby Params Cheat Sheet](https://www.rubyguides.com/2019/11/ruby-hash/)

---

*End of Guide*

