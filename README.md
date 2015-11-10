#Mapping tables to objects 
###Basics of reading from a database table that is mapped to a Ruby object


##Objectives
1. Build methods that read from a database table
2. Build `Song.all` that returns all songs from the database
3. Build 	`Song.find_by_name` method that return a song from the database by the song's name
4. How to convert what the database gives you into a Ruby object

##Overview
Our Ruby program gets most interesting when we 
lego-type analogy: we don't store ruby objects in the database and we don't get ruby objects back from the database when we execute something like a select statement. We store the raw data that describes a given ruby object in a table row and we get back raw data that describes a ruby object when we select from that table. It is up to us to write the code that takes that data and turns it back into an instance of whatever class.

Then write the Song.all method (SELECT) that returns and Song object. No Meta! Then write a `find_by_name` method. 

Concept we want students to understand: building methods that read from a database table.


##`Song.all` 

```ruby
class Song
  def self.all

  end
end
```

##`Song.find_by_name`

```ruby
class Song
  def self.find_by_name	(name)
    
  end
end
```

