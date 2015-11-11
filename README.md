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
Imagine we have a Song class. 

##`Song.all` 

```ruby
class Song
  def self.all
    sql = <<-SQL
      SELECT *
      FROM songs
    SQL
    
    DB[:conn].execute(sql,name).map do |row|
      self.new_from_db(row)
    end
  end
end
```

##`Song.find_by_name`

```ruby
class Song
  def self.find_by_name	(name)
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

