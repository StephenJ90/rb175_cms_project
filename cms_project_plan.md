RB175_cms_project > cms.md

---

CMS Project
===

The goal of this project is to build a simple file-based content management system.

## Getting Started

### Requirements

When a user visits the path "/", the application should display the text "Getting started."

### Implementation

1. Set up initial files
  - Gemfile, file to contain the application ex. cms.rb
2. Run `bundle install` to install all dependencies
3. Code the route that will display "Getting started".

### Solution

```ruby
# Gemfile
source "https://rubygems.org"

gem "sinatra"
gem "sinatra-contrib"
```
```ruby
# cms.rb
require "sinatra"

get "/" do
  "Getting started."
end
```
`bundle install`

### LS Implementation

1. Create a new directory on your computer for this project.
2. Within this directory, create a Gemfile and within it, list the two dependencies needed by the application initially: sinatra and sinatra-contrib.
3. Create a file to contain the application called cms.rb.
4. Ensure you can install all dependencies by running bundle install within the project's directory.
5. Ensure that you can start the Sinatra application by running ruby cms.rb.

### LS Solution

```ruby
# Gemfile
source "https://rubygems.org"

gem "sinatra"
gem "sinatra-contrib"
```
```ruby
# cms.rb
require "sinatra"
require "sinatra/reloader"

get "/" do
  "Getting started."
end
```
`$ bundle install`
`$ bundle exec ruby cms.rb`

---

## Adding an Index Page

Many content management systems store their content in databases, but some use files stored on the filesystem instead. This is the path we will follow with this project. Each document within the CMS will have a name that includes an extension. This extension will determine how the contents of the page are displayed in later steps.

### Requirements

1. When a user visits the home page, they should see a list of the documents in the CMS: `history.txt`, `changes.txt` and `about.txt`.

### Implementation

1. create `public` directory in project directory
2. create above mentioned files and place in `public` directory
3. create `views` directory in project directory
4. create file called `index.erb` and place it in views directory
  - this file will display the list of files in our home directory
5. update Gemfile to include `erubis` gem
6. update `cms.rb` file to require `tilt/erubis`
7. determine route to display names of files from `public` directory
8. Update Gemfile with `erubis` gem and require `"tilt/erubis"` in `cms.rb` file.

LS provides the following code:

```ruby
root = File.expand_path("..", __FILE__)
```
This will get us the absolute path name of the directory where our program or where `__FILE__` lives.

So if we need to access any of the above mentioned files (ex. history.txt), we will have the path to retrieve them. If we were to output the variable `root` from above into our browser when we visit the homepage or `"/"` we would get the following:

`/omitted/omitted/Launch School/rb175_cms_project`

That is because `__FILE__` is currently representing our application file which is `cms.rb` and resides in the `rb175_cms_project` directory.

### Solution

Updating Gemfile:

```ruby
source "https://rubygems.org"

gem "sinatra"
gem "sinatra-contrib"
gem "erubis" # Add in this line
```

For step 7, we need get access to the file names that reside in our public directory. This will require some digging into the Ruby documentation. I decided to do a google search "how to retrieve multiple files from directory in ruby". 

I found the method `Dir.children` which "Returns an array containing all of the filenames except for “.” and “..” in the given directory". So I used the following code:

```ruby
Dir.children("public") # => ["about.txt", "changes.txt", "history.txt"]
```

This returns an array with all of the files from the `public` directory. I can use this in the homepage route like so:

```ruby
get "/" do
  @files = Dir.children("public")
  erb :index
end
```

It is assigned to an instance variable so that it is available to the `index.erb` file. A side note that cause some frustration. I initially had `@files = Dir.children("public")` in the global space, outside of the root and then just tried passing in `@files` to the route on its own. If I were to go that way, I would need to place the code in a `before` block like so:

```ruby
before do
  @files = Dir.children("public")
end
```

and my route would now look like:

```ruby
get "/" do
  erb :index  # @files is now accessible because of the before block
end
```

The `index.erb` file is as follows:

```html
<ul>
  <% @files.each do |file| %>
    <li><%= file %></li>
  <% end %>
</ul>
```

My solution does not use LS's `root` variable provided, but I have achieved desired result all the same.

## LS Solution

### Implementation

1. Create a data directory in the same directory as your Sinatra application. Within the data directory, create three text files with following names: history.txt, changes.txt and about.txt.
2. Use methods from the File class and the Dir class to get a list of documents. Together, these classes provide many useful methods for manipulating file names, file paths, and directories.
3. Use an ERB template to render the list of documents.

### Solution

```ruby
# Gemfile
source "https://rubygems.org"

gem "sinatra"
gem "sinatra-contrib"
gem "erubis"
```

```ruby
# cms.rb
require "sinatra"
require "sinatra/reloader"
require "tilt/erubis"

root = File.expand_path("..", __FILE__)

get "/" do
  @files = Dir.glob(root + "/data/*").map do |path|
    File.basename(path)
  end
  erb :index
end
```

```html
<!-- views/index.erb -->
<ul>
  <% @files.each do |file| %>
    <li><%= file %></li>
  <% end %>
</ul>
```

LS Explanation of:

```ruby
root = File.expand_path("..", __FILE__)
```

As you can see, expand_path lets you determine the absolute path name that corresponds to a relative pathname. In this case, the relative name is `..`, which indicates the parent directory. Furthermore, the `__FILE__` argument tells expand_path to use the name specified by `__FILE__` as the starting point.

The `__FILE__` argument is a well-known Ruby feature, but it's poorly documented. It represents the name of the file that contains the reference to `__FILE__`. For instance, if your file is name `myprog.rb` and you run it with the ruby myprog.rb, then `__FILE__` is `myprog.rb`. When we combine this value with .. in the call to expand_path, we get the absolute path name of the directory where our program lives. For instance, if myprog.rb is in the /Users/me/project directory, then `File.expand_path("..", __FILE__)` returns `/Users/me/project`. This value lets us access other files in our project directory without having to use relative path names.

Looking at this code, you may conclude that it's a bit too complicated. Is there an easier way to determine where a program resides? For specific use cases, there are. However, the problem of determining a program's directory is one that has a lot of edge cases. For instance, what happens if your current directory differs from the one where the program file is? Is it in your PATH? Are there symbolic links involved? (We won't discuss symbolic links in detail, but they provide a way to give a file an alias.)

All these edge cases mean that the problem is more complicated than it would seem. Somewhere along the line, a Ruby programmer realized that he could determine the working directory easily, accurately, and with minimal code by writing `File.expand_path("..", __FILE__)`; the rest is history. Some other programmers liked this solution and copied it, and still more programmers copied them. Thus, a Ruby idiom was born. Do you need to know what directory the current file is in? Then use `File.expand_path("..", __FILE__)`. You don't have to understand how it works; you just need to know that it does.

An alternative to this idiom uses `__dir__` instead: `File.expand_path(__dir__)`. Though simpler, there are some subtle differences, so most Ruby developers still use the more complicated idiom.

**Further Understanding:**

Q: Why do we need to use the absolute path in cms.rb?
A: What happens if we start the program from another directory? You can't guarantee that the person running the program will always be in the proper directory - in fact, you will more than likely not be since deploying a program usually involves moving the program to a separate directory.

---

## Viewing Text Files

### Requirements

1. When a user visits the index page, they are presented with a list of links, one for each document in the CMS.
2. When a user clicks on a document link in the index, they should be taken to a page that displays the content of the file whose name was clicked.
3. When a user visits the path /history.txt, they will be presented with the content of the document history.txt.
4. The browser should render a text file as a plain text file.


### Implementation

1. Update index.erb to make each file name in the list a link to the document
2. Create new view file that will be used for file display
3. Update cms.rb application with route that will link to the content of requested document.

### Solution

1.

```html
<ul>
  <% @files.each do |file| %>
    <li><a href="/<%= file %>"><%= file %></a></li>
  <% end %>
</ul>
```

2. A very simplified solution would be just output the content as is. This takes the `@content` instance variable as shown in route below.

```html
<p><%= @content %></p>
```

3. This step will require that we get the correct content that matches the file requested and have it available for our new erb file to display.

```ruby
get "/:file_name" do
  filename = params[:file_name]
  file = File.open(root + "/data/#{filename}")
  @content = file.read

  erb :file
end
```
This solution utilizes the `root` variable from earlier which displays the absolute path to where the file is located. This will ensure that we use the correct files for this project. 

These solutions have produced the desired results. Let's compare with LS's solution now.

## LS Solution

### LS Implementation

1. Update `views/index.erb` to make each document name a link.
2. Add a new route that will handle viewing the contents of documents.
3. In the new route, read the contents of the document to be viewed.
4. Set an appropriate value for the `Content-Type` header to tell browsers to display the response as plain text.

### Solution

```ruby
# cms.rb
get "/:filename" do
  file_path = root + "/data/" + params[:filename]

  headers["Content-Type"] = "text/plain"
  File.read(file_path)
end
```
```html
<!-- views/index.erb -->
<ul>
  <% @files.each do |file| %>
    <li><a href="/<%= file %>"><%= file %></a></li>
  <% end %>
</ul>
```

### Comparison

- No need to use `File.open`, therefore could reduce 3 lines down to 1 line of code.
- `file.erb` could be avoided here with way the route is written. 
- The route will return the value of `read` method which is the contents of the selected file. The browser is able to output the content without `file.erb`

I will use this route going forward.

---

## Adding Tests

```ruby
ENV["RACK_ENV"] = "test"

require "minitest/autorun"
require "rack/test"

require_relative "../cms"

class CMSTest < Minitest::Test
  include Rack::Test::Methods

  def app
    Sinatra::Application
  end

  def test_index
    get "/"

    assert_equal 200, last_response.status
    assert_equal "text/html;charset=utf-8", last_response["Content-Type"]
    assert_includes last_response.body, "about.txt"
    assert_includes last_response.body, "changes.txt"
    assert_includes last_response.body, "history.txt"
  end

  def test_viewing_text_document
    get "/history.txt"

    assert_equal 200, last_response.status
    assert_equal "text/plain", last_response["Content-Type"]
    assert_includes last_response.body, "Ruby 0.95 released"
  end
end
```

---

## Handling Requests for Nonexistent Documents

### Requirements

1. When a user attempts to view a document that does not exist, they should be redirected to the index page and shown the message: $DOCUMENT does not exist.

2. When the user reloads the index page after seeing an error message, the message should go away.

### Implementation

1. update the filename route so that if a user tries to link to a non-existent document, they are redirected back to the index page with a message explaining the error. 

2. Update the `index.erb` file. We'll need to access a session. Not exactly sure how to do this. Once this is figured out, we'll need to add a message to the session that will output if the user tried to access a non-existent file.

### Solution

If we have a list of all the files, we can see if the requested file is included within the list of files.

```ruby
# cms.rb
enable :sessions
```

```ruby
# cms.rb
get "/:filename" do
  file_path = root + "/data/" + params[:filename]
  @files = Dir.children("data")

  if @files.include?(params[:filename])
    headers["Content-Type"] = "text/plain"
    File.read(file_path)
  else
    session[:message] = "#{params[:filename]} does not exist."
    redirect "/"
  end
end
```
```erb
<!-- index.erb -->
<% if session[:message] %>
  <p><%= session.delete(session[:message]) %></p>
<% end %>
```
The redirect works, but no message is output to the user.

After some debugging and some Stack Overflow searching, I found that when I enable `sessions`: "A random secret is generated for you by Sinatra. However, since this secret will change with every start of your application, you might want to set the secret yourself, so all your application instances share it.". So updating my application with:

```ruby
enable :sessions
set :session_secret, 'secret'
```

And then fixing my `index.erb` file,

```erb
<% if session[:message] %>
  <p><%= session.delete(:message) %></p>
<% end %>
```

I get the desired results.

## LS Solution

### Implementation

1. Check to see if a file exists before attempting to read in its content.
2. Enable sessions in the application so we can persist data between requests.
3. If a document doesn't exist, store an error message in the session and redirect the user.
4. In the index template, if there is error message, print it out and delete it.

### Solution

```ruby
# cms.rb
configure do
  enable :sessions
  set :session_secret, 'super secret'
end

...

get "/:filename" do
  file_path = root + "/data/" + params[:filename]

  if File.file?(file_path)
    headers["Content-Type"] = "text/plain"
    File.read(file_path)
  else
    session[:message] = "#{params[:filename]} does not exist."
    redirect "/"
  end
end
```
```erb
<!-- views/index.erb -->
<% if session[:message] %>
  <p><%= session.delete(:message) %></p>
<% end %>

<ul>
  <% @files.each do |file| %>
    <li><a href="/<%= file %>"><%= file %></a></li>
  <% end %>
</ul>
```

### Comparison

The use of `configure do`. The sinatra documentation says "Run once, at startup, in any environment". I'm not completely sure why it's needed or why it's used.

Again, the LS route is much cleaner. It uses `File.file?` method which "Returns true if the named file exists and is a regular file."(Ruby docs) I will be using both of these solutions going forward.

---

## Viewing Markdown Files

Provided by LS:

1. Add `redcarpet` to your Gemfile and run bundle install.
2. Add `require "redcarpet"` to the top of your application.
3. To actually render text into HTML, create a `Redcarpet::Markdown` instance and then use it to process the text:

```ruby
markdown = Redcarpet::Markdown.new(Redcarpet::Render::HTML)
markdown.render("# This will be a headline!") # => "<h1>This will be a headline!</h1>\n"
```

### Requirements

1. When a user views a document written in Markdown format, the browser should render the rendered HTML version of the document's content.

### Implementation

Currently, the routes in the app only handle one type of file. Thus...
1. We need to come up with a way to determine what _type_ of file is being requested and return the appropriate content for that file. So far that's just `.txt` and `.md` file types. Let's try updating the route with brute force to handle different file types.

### Solution

```ruby
get "/:filename" do
  file_path = root + "/data/" + params[:filename]

  if File.file?(file_path) && File.extname(file_path) == ".md"
    markdown = Redcarpet::Markdown.new(Redcarpet::Render::HTML)
    markdown.render(File.read(file_path))
  elsif File.file?(file_path)
    headers["Content-Type"] = "text/plain"
    File.read(file_path)
  else
    session[:message] = "#{params[:filename]} does not exist."
    redirect "/"
  end
end
```

Very messy, but gets the desired results. A refactoring is needed. 

## LS Solution

### Implementation

1. Rename `about.txt` to `about.md` Add some Markdown-formatted text to this file.
2. Create a helper method called `render_markdown` that takes a single argument, the text to be processed, and returns rendered HTML.
3. When a user is viewing a file with the extension `md`, render the file's content using `RedCarpet` and return the result as the response's body.

### Solution

LS defines two methods to make the code clear and readable:

```ruby
def render_markdown(text)
  markdown = Redcarpet::Markdown.new(Redcarpet::Render::HTML)
  markdown.render(text)
end

def load_file_content(path)
  content = File.read(path)
  case File.extname(path)
  when ".txt"
    headers["Content-Type"] = "text/plain"
    content
  when ".md"
    render_markdown(content)
  end
end
```
And updates the route to use the `load_file_content` method:

```ruby
get "/:filename" do
  file_path = root + "/data/" + params[:filename]

  if File.exist?(file_path)
    load_file_content(file_path)
  else
    session[:message] = "#{params[:filename]} does not exist."
    redirect "/"
  end
end
```

### Comparison

The use of those two new methods really keep the code clean and readable. I knew a refactor was in order, but wasn't quite sure how to implement the change.

---

## Editing Document Content

Now it’s time to allow users to modify the content stored within our CMS.

### Requirements

1. When a user views the index page, they should see an “Edit” link next to each document name.
2. When a user clicks an edit link, they should be taken to an edit page for the appropriate document.
3. When a user views the edit page for a document, that document's content should appear within a textarea.
4. When a user edits the document's content and clicks a “Save Changes” button, they are redirected to the index page and are shown a message: `$FILENAME has been updated.`.

### Implementation

1. Add a link (<a> element) to the `index.erb` file for `Edit`. 
2. A route with `/edit/:filename` as the path parameter will also need to be created.
3. a new view file called `edit` needs to be created to present the user with the content of the requested file in a text box field that allows the content to be edited.
4. Add a save changes button to the `edit` view file
5. a `session message` will have to be initialized when the user clicks on the "save changes" button.

### Solution

The update `index.erb` file to link to the page to edit files.

```html
<% if session[:message] %>
  <p><%= session.delete(:message) %></p>
<% end %>

<ul>
  <% @files.each do |file| %>
    <li>
      <a href="/<%= file %>"><%= file %></a>
      <a href="/edit/<%= file %>">Edit</a>
    </li>
  <% end %>
</ul>
```

The route would look like this. I'm unsure of how to get each page to display the content in a text box available for the user to edit.

```ruby
get "/edit/:filename" do
  file_path = root + "/data/" + params[:filename]
  @content = load_file_content(file_path)

  erb :edit
end
```

My lack-lustre attempt with the `edit.erb` file. 

```html
<h3>Edit content of <%= params[:filename] %></h3>
<input name="file_content" placeholder="Edit document" type="text" value="<%= @content %>">
```

## LS Solution

### Implementation

1. Update views/index.erb to add edit links after each document name.
2. Create a new route for editing a document. Within this route, render a new view template that contains a form and a text area.
3. Create a new route for saving changes to the document. Within this route, update the contents of the appropriate document, add a message to the session, and redirect the user.

### Solution

The route to the edit a document.

```ruby
get "/:filename/edit" do
  file_path = root + "/data/" + params[:filename]

  @filename = params[:filename]
  @content = File.read(file_path)

  erb :edit
end
```

The route to `post` an update of an edited document.

```ruby
post "/:filename" do
  file_path = root + "/data/" + params[:filename]

  File.write(file_path, params[:content])

  session[:message] = "#{params[:filename]} has been updated."
  redirect "/"
end
```

```html
<!-- views/edit.erb -->
<form method="post" action="/<%= @filename %>">
  <label for="content">Edit content of <%= @filename %>:</label>
  <div>
    <textarea name="content" id="content" rows="20" cols="100"><%= @content %></textarea>
  </div>
  <button type="submit">Save Changes</button>
</form>
```

My `views/index.erb` is the almost the same as LS. I would need to change this line:

```html
<a href="/edit/<%= file %>">Edit</a>
```
to:
```html
<a href="/<%= file %>/edit">edit</a>
```

I had the path order mixed up.

---

## Adding Global Style and Behaviour

When a message is displayed to a user anywhere on the site, it should be styled in a way that is easily distinguished from the rest of the page. This will help attract the user's attention to the information in the message that would otherwise be easy to miss.

While we're adding styling, we can also change the default display of the site to use a sans-serif font and have a little padding around the outside.

### Requirements

1. When a message is displayed to a user, that message should appear against a yellow background
2. Messages should disappear if the page they appear on is reloaded.
3. Text files should continue to be displayed by the browser as plain text.
4. The entire site (including markdown files, but not text files) should be displayed in a sans-serif typeface.

### Implementation

1. Unlearned in the art of CSS, so we'll use LS's solution.
2. I implemnted this in an earlier on.
3. Maybe set headers["Content-Type"] = "text/plain"
4. Hmmm not sure.

### LS Solution

layout.erb
```html
<html>
  <title>CMS</title>
  <link href="/cms.css" rel="stylesheet" type="text/css" />
  <body>
    <% if session[:message] %>
      <p class="message"><%= session.delete(:message) %></p>
    <% end %>
    <%= yield %>
  </body>
</html>
```
index.erb
```html
<ul>
  <% @files.each do |file| %>
    <li>
      <a href="/<%= file %>"><%= file %></a>
      <a href="/<%= file %>/edit">edit</a>
    </li>
  <% end %>
</ul>
```

cms.css
```css
body {
  padding: 1em;
  font-family: sans-serif;
}

.message {
  padding: 10px;
  background: #FFFF99;
}
```

---

## Creating New Documents

### Requirements

1. When a user views the index page, they should see a link that says "New Document".
2. When a user clicks the "New Document" link, they should be taken to a page with a text input labeled "Add a new document:" and a submit button labeled "Create"
3. When a user enters a document name and clicks "Create", they should be redirected to the index page. The name they entered in the form should now appear in the file list. They should see a message that says "$FILENAME was created.", where $FILENAME is the name of the document just created
4. If a user attempts to create a new document without a name, the form should be re-displayed and a message should say "A name is required."

### Implementation

1. Add new link to index.erb file that will display an link to create a new file
2. create `new.erb` file 
3. write a new GET route to take a user to the create a file page
4. write a POST route that creates a new file and redirects the use back to the index page where they should see mew file listed.
5. validate the user input to only accept valid file names.

### Solution

1. Updated `index.erb` file.
```html
<ul>
  <% @files.each do |file| %>
    <li>
      <a href="/<%= file %>"><%= file %></a>
      <a href="/<%= file %>/edit">Edit</a>
    </li>
  <% end %>
</ul>

<p><a href="/new">New Document</a></p>
```
2. `new.erb` file
```html
<form method="post" action="/create">
  <label for="document_name">Add a New Document:</label>
  <input type="text" name="file_name" placeholder="New Document">
  <button type="submit">Create</button>
</form>
```

3. GET route
```ruby
get "/new" do
  erb :new
end
```

3. POST route
```ruby
post "/create" do
  # Code to create a new file
  # See LS solution

  session[:message] = "#{params[:filename]} was created."
  redirect "/"
end
```

See LS solution

### LS Solution

```html
<!-- views/new.erb -->
<form method="post" action="/create">
  <label for="filename">Add a new document:</label>
  <div>
    <input name="filename" id="filename" />
    <button type="submit">Create</button>
  </div>
</form>
```
My `new.erb` file didn't work. I had to change it to the above for a new document to be created successfully. I experimented with changing `for="filename"` and `name="filename"` and they both had to be matching as `filename` for it to work. Ex. `file` wouldn't work as I suspect `filename` needs to match the `filename` used in the `post` route below.
```ruby
# cms.rb
get "/new" do
  erb :new
end

post "/create" do
  filename = params[:filename].to_s

  if filename.size == 0
    session[:message] = "A name is required."
    status 422
    erb :new
  else
    file_path = File.join(data_path, filename)

    File.write(file_path, "")
    session[:message] = "#{params[:filename]} has been created."

    redirect "/"
  end
end
```

---








