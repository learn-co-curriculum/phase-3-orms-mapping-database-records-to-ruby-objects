# Converting Database Records to Ruby Objects

## Learning Goals

- Build methods that read from a database table
- Build a `Song.all` class method that returns all songs from the database
- Build a `Song.find_by_name` class method that accepts one argument, a name,
  and searches the database for a song with that name and returns the matching
  song entry if one is found
- Convert what the database gives you into a Ruby object

## Introduction

In this lesson, we'll cover the basics of reading from a database table that is
mapped to a Ruby object.

Our Ruby program gets most interesting when we add data. To do this, we use a
database. When we want our Ruby program to store things, we send them off to a
database. When we want to retrieve those things, we ask the database to send
them back to our program. This works very well, but there is one small problem
to overcome — our Ruby program and the database don't speak the same language.

Ruby understands objects. The database understands raw data.

We don't store Ruby objects in the database, and we don't get Ruby objects back
from the database. We store raw data describing a given Ruby object in a table
row, and when we want to reconstruct a Ruby object from the stored data, we
select that same row in the table.

When we query the database, it is up to us to write the code that takes that
data and turns it back into an instance of the appropriate class. We, the
programmers, will be responsible for translating the raw data that the database
sends into Ruby objects that are instances of a particular class.

## Code Along

Let's continue building out the `Song` class and its object-relational mapping
methods from the previous lesson. We can use our code to make new songs and
persist them to the database, but what if we want to access existing songs from
the database?

We need to build three methods to access all of those songs and convert them to
Ruby objects.

To start, review the code from the `Song` class. Then take a look at this code
in the `bin/run` file:

```rb
#!/usr/bin/env ruby

require 'pry'
require_relative '../config/environment'

def reset_database
  Song.drop_table
  Song.create_table
  Song.create(name: "Hello", album: "25")
  Song.create(name: "99 Problems", album: "The Black Album")
end

reset_database

binding.pry
"pls"
```

This file is set up so that you can explore the database using the `Song` class
from a Pry session. We'll use this code later on during this code along.

## `Song.new_from_db`

The first thing we need to do is convert what the database gives us into a Ruby
object. We will use this method to create all the Ruby objects in our next two
methods.

One thing to know is that the database, SQLite in our case, will return an array
of data for each row. For example, a row for Michael Jackson's "Billie Jean"
from the album "Thriller" that has an id of 1 would look like this:
`[1, "Billie Jean", "Thriller"]`.

```ruby
class Song

  # ... rest of methods

  def self.new_from_db(row)
    # self.new is equivalent to Song.new
    self.new(id: row[0], name: row[1], album: row[2])
  end

end
```

Now, you may notice something — since we're retrieving data from a database, we
are using `new`. We don't need to _create_ records. With this method, we're
reading data from SQLite and temporarily representing that data in Ruby.

## `Song.all`

Recall that in previous lessons with Ruby classes, we used the `Class.all`
method along with the `@@all` class variable to return an array of all instances
of our class. In those examples, `@@all` was the **single source of truth** for
instances in a particular class.

That approach showed some limitations, however. Using that method meant that our
Ruby objects were only persisted in memory as long as our Ruby program was running.
If we exited the program and re-ran our code, we'd lose access to that data.

Now that we have a SQL database, our classes have a new way to persist data:
using the database!

To return all the songs in the database, we need to execute the following SQL
query: `SELECT * FROM songs`. Let's store that in a variable called `sql` using
a heredoc (`<<-`) since our string will go onto multiple lines:

```ruby
sql = <<-SQL
  SELECT *
  FROM songs
SQL
```

Next, we will make a call to our database using `DB[:conn]`. This `DB` hash is
located in the `config/environment.rb` file:

```rb
DB = { conn: SQLite3::Database.new("db/music.db") }
```

Notice that the value of the connection in this hash is actually a new instance
of the `SQLite3::Database` class. This is how we will connect to our database.
Our database instance responds to a method called `execute` that accepts raw SQL
as a string. Let's pass in that SQL we stored above:

```ruby
class Song

  # ... rest of methods

  def self.all
    sql = <<-SQL
      SELECT *
      FROM songs
    SQL

    DB[:conn].execute(sql)
  end

end
```

This will return an array of rows from the database that matches our query. Now,
all we have to do is iterate over each row and use the `self.map` method to
create a new Ruby object for each row:

```ruby
class Song
  def self.all
    sql = <<-SQL
      SELECT *
      FROM songs
    SQL

    DB[:conn].execute(sql).map do |row|
      self.new_from_db(row)
    end
  end
end
```

With this method in place, let's try using the `Song.all` method from Pry to
access all the songs in the database. Run `ruby bin/run`, and then follow along
in the Pry terminal:

```rb
Song.all
# => [#<Song:0x00007ffc7a093098 @album="25", @id=1, @name="Hello">,
 #<Song:0x00007ffc7a093048 @album="The Black Album", @id=2, @name="99 Problems">]
```

Success! We can see both songs in the database as an array of song instances. We
can interact with them just like any other Ruby objects:

```rb
Song.all.first
# => #<Song:0x00007ffc7a0b1480 @album="25", @id=1, @name="Hello">
Song.all.last
# => #<Song:0x00007ffc7a0c4a08 @album="The Black Album", @id=2, @name="99 Problems">
Song.all.last.name
# => "99 Problems"
Song.all.last.name.reverse
# => "smelborP 99"
```

## `Song.find_by_name`

This one is similar to `Song.all`, with the small exception being that we have to
include a name in our SQL statement. To do this, we use a question mark where we
want the `name` parameter to be passed in, and we include `name` as the second
argument to the `execute` method:

```ruby
class Song

  # ... rest of methods

  def self.find_by_name(name)
    sql = <<-SQL
      SELECT *
      FROM songs
      WHERE name = ?
      LIMIT 1
    SQL

    DB[:conn].execute(sql, name).map do |row|
      self.new_from_db(row)
    end.first
  end
end
```

Don't be freaked out by that `#first` method chained to the end of the
`DB[:conn].execute(sql, name).map` block. The return value of the `#map` method
is an array, and we're simply grabbing the `#first` element from the returned
array. Chaining is cool!

Let's try out this new method. Exit Pry, and run `ruby bin/run` again:

```rb
Song.find_by_name("Hello")
# => #<Song:0x00007f7f579ae6c8 @album="25", @id=1, @name="Hello">
```

Success!
