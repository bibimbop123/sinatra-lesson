# 🧠 Sinatra Guide — Beginner to Intermediate

A complete walkthrough for learning Sinatra, from setup to building modular apps.

---

## 🥇 BEGINNER LEVEL

### ⚙️ 1. Setup and Installation

```bash
gem install sinatra
```

Create `app.rb`:

```ruby
require 'sinatra'

get '/' do
  "Hello, Sinatra!"
end
```

Run:

```bash
ruby app.rb
```

Visit 👉 `http://localhost:4567`

---

### 🧩 2. Folder Structure

A simple Sinatra app can live in one file, but as it grows, use this:

```
my_app/
├── app.rb
├── views/
│   ├── layout.erb
│   ├── index.erb
│   └── about.erb
├── public/
│   └── style.css
└── config.ru
```

---

### 🚦 3. Routing Basics

```ruby
get '/' do
  "Home Page"
end

get '/hello/:name' do
  "Hello, #{params[:name]}!"
end

post '/submit' do
  "Form submitted with #{params[:data]}"
end
```

---

### 🎨 4. Using Views

**app.rb**
```ruby
get '/about' do
  @title = "About Us"
  erb :about
end
```

**views/about.erb**
```erb
<h1><%= @title %></h1>
<p>Welcome to our site!</p>
```

**views/layout.erb**
```erb
<html>
  <head><title>My Site</title></head>
  <body><%= yield %></body>
</html>
```

---

### 💾 5. Params and Forms

**views/form.erb**
```erb
<form action="/greet" method="post">
  <input type="text" name="name">
  <input type="submit" value="Say Hi">
</form>
```

**app.rb**
```ruby
post '/greet' do
  "Hi #{params[:name]}!"
end
```

---

### 🔒 6. Sessions

```ruby
enable :sessions

get '/login' do
  session[:user] = "Brian"
  "Logged in as #{session[:user]}"
end

get '/logout' do
  session.clear
  "Logged out"
end
```

---

## 🥈 INTERMEDIATE LEVEL

### 🧱 7. Modular Sinatra Architecture

When apps grow, switch from **classic** style to **modular** style.

**config.ru**
```ruby
require './app'
run MyApp
```

**app.rb**
```ruby
require 'sinatra/base'

class MyApp < Sinatra::Base
  get '/' do
    "Modular Sinatra App"
  end

  run! if app_file == $0
end
```

**Why modular?**
- ✅ Easier testing  
- ✅ Mount multiple apps  
- ✅ Cleaner organization  

---

### 🧰 8. Configuration & Environments

```ruby
configure :development do
  set :show_exceptions, true
  set :port, 4000
end

configure :production do
  disable :show_exceptions
end
```

---

### 🔄 9. RESTful CRUD Example (with Sinatra ActiveRecord)

Add to Gemfile:

```ruby
gem 'sinatra-activerecord'
gem 'sqlite3'
```

Run:

```bash
bundle install
rake db:create_migration NAME=create_posts
```

**app.rb**

```ruby
require 'sinatra/activerecord'

set :database, {adapter: 'sqlite3', database: 'db.sqlite3'}

class Post < ActiveRecord::Base; end

get '/posts' do
  @posts = Post.all
  erb :index
end

post '/posts' do
  Post.create(title: params[:title], body: params[:body])
  redirect '/posts'
end
```

---

### ⚡ 10. Middleware and Filters

**Before/After Filters**
```ruby
before do
  @start_time = Time.now
end

after do
  puts "Request took #{Time.now - @start_time} seconds"
end
```

---

### 🧠 11. Helpers (Reusable Logic)

```ruby
helpers do
  def logged_in?
    !!session[:user]
  end
end

get '/dashboard' do
  halt 403, "Forbidden" unless logged_in?
  "Welcome to your dashboard"
end
```

---

### 🧾 12. Error Handling

```ruby
not_found do
  status 404
  erb :not_found
end

error do
  erb :error
end
```

---

### 🧩 13. Serving Static Files

Anything in `/public` is auto-served:

```
/public/js/main.js
/public/css/style.css
```

Use in HTML:

```html
<link rel="stylesheet" href="/css/style.css">
```

---

### 🧪 14. Testing (Rspec + Rack::Test)

Add to Gemfile:

```ruby
gem 'rspec'
gem 'rack-test'
```

**spec/app_spec.rb**

```ruby
require 'rack/test'
require_relative '../app'

describe 'My Sinatra App' do
  include Rack::Test::Methods
  def app; MyApp; end

  it "loads the home page" do
    get '/'
    expect(last_response).to be_ok
  end
end
```

---

### 🚀 15. Deployment (Render or Heroku)

**config.ru**
```ruby
require './app'
run MyApp
```

Then commit and push to GitHub → connect on Render or Heroku.

Render auto-detects Sinatra apps with a `config.ru`.

---

## 🎯 Bonus: Best Practices

✅ Keep logic out of routes — use helpers or models  
✅ Use modular apps for anything beyond prototypes  
✅ Store secrets in environment variables  
✅ Use partials (`erb :_partial, locals: { var: val }`) for DRY views  
✅ Use rack middleware for logging or authentication  

---

## ⚡ Pro Command Cheats

| Command | Description |
|----------|-------------|
| `ruby app.rb` | Run Sinatra classic app |
| `rackup` | Run via Rack (`config.ru`) |
| `set :port, 8080` | Change port |
| `enable :sessions` | Enable sessions |
| `halt 404, "Not Found"` | Stop execution with status |

---
