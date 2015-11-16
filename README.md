#Mapping tables to objects 
###Basics of reading from a database table that is mapped to a Ruby object


##Objectives
1. Build methods that read from a database table
2. Understand how to build `Song.all` that returns all songs from the database
3. Understand how to build `Song.find_by_name` method that return a song from the database by the song's name
4. How to convert what the database gives you into a Ruby object

##Overview
Our Ruby program gets most interesting when we add data. To do this, we use a database. When we want our Ruby program to store things we send them off to a database. When we want to retrieve those things, we ask the database to send them back to our program. This works very well, but there is one small problem to overcome -- our Ruby program and the database don't speak the same language.

Ruby understands objects. The database understands raw data.

We don't store Ruby objects in the database, and we don't get Ruby objects back from the database. We store the raw data that describes a given Ruby object in a table row and we get back raw data that describes a ruby object when we select from that table. 

When we query the database, it is up to us to write the code that takes that data and turns it back into an instance of whatever class. We, the programmers, will be the translators that translate the raw data that the database sends into Ruby objects that are instances of a particular class.

##Example
Let's use our song domain as an example. Imagine we have a Song class that is responsible for making songs. Every song will come with two attributes, a title and a length. We could make a bunch of new songs, but we want to look at all the songs we have that have already been created.

Imagine we have a database with 1 million songs. We need to build three methods to access all of those songs and convert them to Ruby objects.

##`.new_from_db`
The first thing we need to do is convert what the database gives us into a Ruby object. We will use this method to create all the Ruby objects in our next two methods.

The first thing to know is that the database (SQLite) in our case, will return an array of data for each row. For example, a row for Michael Jackson's "Thriller" (356 seconds long) that has a db id of 1 would look like this: `[1, "Thriller", 356]`.

```ruby
def self.new_from_db(row)
  new_ruby_object = self.new  # self.new is the same as running Student.new
  new_ruby_object.id = row[0]
  new_ruby_object.name =  row[1]
  new_ruby_object.length = row[2]
end
```

This can also be achieved using the `tap` method. Tap yields the object to the block and then returns the original object. If you're comfortable using `tap`, go for it. If it's confusing, you can ignore it. 

```ruby
def self.new_from_db(row)
  self.new.tap do |song|
    song.id = row[0]
    song.name =  row[1]
    song.length = row[2]
  end
end
```

##`Song.all` 
Now we can start writing our methods to retrieve the data. To return all the songs in the database we need the following SQL query: `SELECT * FROM songs`. Let's store that in a variable called `sql` using a heredoc (`<<-`) since our string will go onto multiple lines.

```ruby
sql = <<-SQL
      SELECT *
      FROM songs
    SQL
```
Next, we will make a call to our database using `DB[:conn]`. This is just the syntax for connecting to our db. In the lab, you can find the following setup that allows us to run the above hash in `config/environment.rb`. The connection will look like: `DB = {:conn => SQLite3::Database.new("db/songs.db")}`. `DB[:conn]` responds to a method called `execute` that accepts raw SQL as a string. Let's pass in that SQL we store above:

```ruby
class Song
  def self.all
    sql = <<-SQL
      SELECT *
      FROM songs
    SQL
    
    DB[:conn].execute(sql)
  end
end
```

This will return an array of rows from the database that match our query. Now, all we have to do is iterate over each row and use the `new_from_db` method to create a new Ruby object for each row.

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

##`Song.find_by_name`
This one is similar to `Song.all` with the small exception being that we have to include a name in our SQL statement. To do this, we use a question mark where we want to name to be passed in, and include the name as the optional argument to the `execute method`.

```ruby
class Song
  def self.find_by_name(name)
    sql = <<-SQL
      SELECT *
      FROM songs
      WHERE name = ?
      LIMIT 1
    SQL

    DB[:conn].execute(sql,name).map do |row|
      self.new_from_db(row)
    end.first
  end
end
```

