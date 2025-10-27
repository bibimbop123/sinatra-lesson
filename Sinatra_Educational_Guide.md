# 🧠 Sinatra Guide — Beginner to Intermediate
**Learn Sinatra by understanding not just how to use it, but how it works.**

Sinatra is a **lightweight Ruby web framework** built on Rack. Unlike Rails, it’s minimalist: you define routes, logic, and views with almost no boilerplate. This guide teaches **how and why** Sinatra behaves the way it does.

---

## 🥇 BEGINNER LEVEL

### ⚙️ 1. Setup and Installation
```bash
gem install sinatra
```

**app.rb**
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

🧠 **Concept:** Sinatra runs on **Rack**, which is the interface between your Ruby code and the web server. Your `get '/'` block is registered as a **route handler** that Rack calls when a matching request comes in.


# Rack vs Puma

## 1. Rack
- **What it is:** A **Ruby interface** between web servers and Ruby web applications (Rails, Sinatra, etc.).
- **Role:** Standardizes how HTTP requests and responses are handled in Ruby.
- **Not a server itself:** Rack doesn’t serve requests; it defines a **callable interface**.
- **Key concept:** Any Ruby web app implementing `call(env)` can work with any Rack-compatible server.

**Example: Minimal Rack app**
```ruby
app = Proc.new do |env|
  ['200', {'Content-Type' => 'text/html'}, ['Hello world']]
end

require 'rack'
Rack::Handler::WEBrick.run app
```

---

## 2. Puma
- **What it is:** A **web server** for Ruby apps.
- **Role:** Handles **HTTP requests from clients**, manages threads, and passes requests to your Rack-compatible app.
- **Key features:**
  - Threaded → handles many requests concurrently.
  - High performance → great for Rails production apps.
  - Rack-compatible → can run Rails, Sinatra, and other Rack-based apps.

**Example (Rails default server):**
```bash
rails server
# Rails defaults to Puma
```

---

## 3. Analogy
- **Rack = language rules** (like “English grammar”)
- **Puma = speaker/actor** that uses those rules to deliver messages (requests/responses) to the audience (browser/client).

---

## 4. Summary
| Component | Role | Notes |
|-----------|------|-------|
| Rack      | Interface/standard | Defines `call(env)` for Ruby web apps |
| Puma      | Web server          | Handles HTTP, speaks Rack, threaded & high performance |

> **Try This:** Change the string to `"Hi, [Your Name]!"` and reload the page.

###require vs require_relative

require loads libraries or gems that Ruby knows about (either built-in or installed).

require_relative loads your own files, using a relative path (like "./my_file.rb").

---

### 🧩 2. Sinatra Philosophy & Structure

- **Minimalism:** You only write what you need.  
- **Explicit control:** Everything in routes is yours to define.  
- **Rack-based:** Every Sinatra app is basically a tiny Rack app.  

**Folder Structure (Optional for small apps, recommended for scaling):**
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

**Rails Comparison:**
| Concept | Rails | Sinatra |
|---------|-------|---------|
| Request → Controller → Action → View | ✅ | ❌ (Route → Action → View) |
| Convention over configuration | ✅ | Minimal / manual |
| Controllers | Separate classes | Usually inline blocks |

> Understanding Sinatra’s simplicity makes Rails easier to grasp later.

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

**Conceptual Notes:**
- `params` includes **query string + route captures + form data**  
- Sinatra checks routes **in order** and executes the first match

> **Try This:** Add `get '/hello/:name/:age'` and display `params[:age]`.

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

🧠 **Concept:** Sinatra automatically wraps views in `layout.erb` unless `layout: false` is specified. Instance variables like `@title` are passed to views.

> **Try This:** Disable the layout with `erb :about, layout: false` and see the difference.

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

> **Concept:** POST requests send data to the server body; GET requests send data in the URL. Sinatra unifies access through `params`.

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

**Concept:** Sessions store **state across HTTP requests** (e.g., login info).  

> **Try This:** Add a `/profile` route that halts with 403 unless `session[:user]` exists.

---

## 🥈 INTERMEDIATE LEVEL

### 🧱 7. Classic vs Modular Sinatra
**Classic style:** Everything in `app.rb` — good for small apps.  
**Modular style:** Subclass `Sinatra::Base` — better for testing, multiple apps, and clean organization.

**app.rb (Modular)**
```ruby
require 'sinatra/base'

class MyApp < Sinatra::Base
  get '/' do
    "Modular Sinatra App"
  end

  run! if app_file == $0
end
```

**config.ru**
```ruby
require './app'
run MyApp
```

> **Concept:** Modular apps avoid polluting the global namespace and allow **multiple apps to mount together**.

> **Try This:** Add another modular app and mount it under `/admin` in `config.ru`.

---

### ⚙️ 8. Configuration & Environments
```ruby
configure :development do
  set :show_exceptions, true
  set :port, 4000
end

configure :production do
  disable :show_exceptions
end
```

💡 Use `.env` for secrets (with `dotenv` gem):  
```ruby
require 'dotenv/load'
API_KEY = ENV['API_KEY']
```

> **Concept:** Sinatra’s `configure` blocks are evaluated **once per app load**, making it easy to adjust settings per environment.

---

### 🔄 9. RESTful CRUD (Sinatra + ActiveRecord)
**Gemfile**
```ruby
gem 'sinatra-activerecord'
gem 'sqlite3'
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

> **Concept:** Sinatra integrates with ActiveRecord like Rails — but you manually wire routes, models, and views.

---

### ⚡ 10. Filters and Middleware
```ruby
before do
  @start_time = Time.now
end

after do
  puts "Request took #{Time.now - @start_time} seconds"
end
```

**Concept:** Filters run **before or after route blocks**, allowing logging, authentication, or preprocessing.

**Middleware Example**
```ruby
use Rack::Logger
```
> Rack middleware wraps the Sinatra app, allowing modular behavior like logging or auth.

---

### 🧠 11. Helpers
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

> **Concept:** Helpers are **shared methods** available inside route blocks and views.

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

> **Concept:** Sinatra uses a simple **catch-all system** for exceptions and missing routes.

---

### 🧩 13. Serving Static Files
Anything under `/public` is auto-served:  
```
/public/css/style.css
/public/js/main.js
```
**HTML Example**
```html
<link rel="stylesheet" href="/css/style.css">
```

> **Concept:** Static files bypass route matching entirely — Rack serves them directly.

---

### 🧪 14. Testing (RSpec + Rack::Test)
```ruby
require 'rack/test'
require_relative '../app'

describe 'MyApp' do
  include Rack::Test::Methods
  def app; MyApp; end

  it "loads home page" do
    get '/'
    expect(last_response).to be_ok
  end
end
```

> **Concept:** Rack::Test simulates HTTP requests without a real server, making Sinatra **easily testable**.

---

### 🚀 15. Deployment (Render / Heroku)
**config.ru**
```ruby
require './app'
run MyApp
```
- Render and Heroku auto-detect Sinatra apps via `config.ru`.  
- Modular apps scale better in production.

---

## 🧭 Best Practices

| ✅ Do | 🚫 Don’t |
|------|-----------|
| Keep logic out of routes (helpers/models) | Put all logic in route blocks |
| Use modular apps for scalability | Everything in `app.rb` |
| Store secrets in `.env` | Hardcode API keys |
| Use partials for DRY views | Repeat HTML across pages |
| Use Rack middleware for logging/auth | Rebuild common logic manually |

---

## ⚡ Quick Commands

| Command | Description |
|---------|-------------|
| `ruby app.rb` | Run classic Sinatra app |
| `rackup` | Run modular app |
| `set :port, 8080` | Change port |
| `enable :sessions` | Enable sessions |
| `halt 404, "Not Found"` | Stop with status & message |

---

## 🎯 Takeaways
1. Sinatra is **lightweight and explicit** — you define what you need.  
2. It’s **built on Rack**, giving low-level access to requests and responses.  
3. **Classic vs Modular**: choose modular for real-world apps and testing.  
4. Learning Sinatra first teaches **core web principles** that scale to Rails or other frameworks.

> **Final Challenge:** Build a small API in Sinatra that returns JSON for posts and allows POST/GET. Experiment with filters, helpers, and modular design.
