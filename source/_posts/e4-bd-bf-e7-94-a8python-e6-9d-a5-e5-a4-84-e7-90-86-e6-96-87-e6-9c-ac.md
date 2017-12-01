---
title: 使用Python来处理文本
tags:
  - python
id: 747
categories:
  - 学习
date: 2015-04-29 20:28:38
---

今天一个朋友需要处理MovieLens的数据集，他想获取评分、用户性别、年龄、职业、电影分类这样的信息，但原始数据集中是分成三个文本文件（使用::分割的CSV文件）存储的，分别是movies.dat（MovieID::Title::Genres）、users.dat（UserID::Gender::Age::Occupation::Zip-code）、ratings.dat（UserID::MovieID::Rating::Timestamp）。<!--more-->

这个需求其实很简单的，本来想用熟悉的Java快速完成，但想到好久没用Python了，很多东西都忘了，所以就用Python帮他实现了一下，就当作回顾Python的知识
以下就是这部分代码，没有做什么优化（对这块了解的不是很深入），如果有什么建议大家可以告诉我！

```python
#!/usr/bin/python

import os

__author__ = 'Liu.D.H'

class Movie(object):
    """This is movie class which contains movie_id, title, and genres fields"""

    def __init__(self, movie_id, title, genres):
        self.movie_id = movie_id
        self.title = title
        self.genres = genres

class User(object):
    """This is user class which contains user_id, gender, age, occupation and zipcode fields"""

    def __init__(self, user_id, gender, age, occupation, zipcode):
        self.user_id = user_id
        self.gender = gender
        self.age = age
        self.occupation = occupation
        self.zipcode = zipcode

class Rating(object):
    """This is movie class which contains user_id, movie_id, rating and timestamp fields"""

    def __init__(self, user_id, movie_id, rating, timestamp):
        self.user_id = user_id
        self.movie_id = movie_id
        self.rating = rating
        self.timestamp = timestamp

def main():
    # define movie, user and rating storage
    movie_dict = {}
    user_dict = {}
    rating_list = []
    detailed_rating_list = []
    occupations = {0: "other", 1: "academic/educator", 2: "artist", 3: "clerical/admin", 4: "college/grad student",
                   5: "customer service", 6: "doctor/health care", 7: "executive/managerial", 8: "farmer",
                   9: "homemaker", 10: "K-12 student", 11: "lawyer", 12: "programmer", 13: "retired",
                   14: "sales/marketing", 15: "scientist", 16: "self-employed", 17: "technician/engineer",
                   18: "tradesman/craftsman", 19: "unemployed", 20: "writer"}

    # Parse movie, user, rating file
    # Here we use absolute path because the CWD may not be the directory of the current script
    # For example, in PyCharm the "os.getcwd()" only returns "."
    root_dir = os.path.dirname(os.path.abspath(__file__)) + os.sep + "ml-1m"
    movie_file_path = root_dir + os.sep + "movies.dat"
    user_file_path = root_dir + os.sep + "users.dat"
    rating_file_path = root_dir + os.sep + "ratings.dat"
    movie_fd = open(movie_file_path)
    user_fd = open(user_file_path)
    rating_fd = open(rating_file_path)
    # To get all lines of the file, notice that each line may end with '\n', So we need "rstrip" function
    # Or we can get stripped lines via movie_fd.read().splitlines()
    # see also http://stackoverflow.com/questions/15233340/getting-rid-of-n-when-using-readlines
    movie_lines = movie_fd.readlines()
    user_lines = user_fd.readlines()
    rating_lines = rating_fd.readlines()

    for line in movie_lines:
        # The structure of movie files is "MovieID::Title::Genres"
        (movie_id, title, genres) = line.rstrip().split("::")
        movie_dict[movie_id] = Movie(movie_id, title, genres)
    for line in user_lines:
        # The structure of user files is "UserID::Gender::Age::Occupation::Zipcode"
        (user_id, gender, age, occupation, zipcode) = line.rstrip().split("::")
        user_dict[user_id] = User(user_id, gender, age, occupation, zipcode)
    for line in rating_lines:
        # The structure of movie files is "UserID::MovieID::Rating::Timestamp"
        (user_id, movie_id, rating, timestamp) = line.rstrip().split("::")
        rating_list.append(Rating(user_id, movie_id, rating, timestamp))

    for rating in rating_list:
        # Extract the needed information, assemble as dict and save in a list
        detailed_rating_list.append({"rating": rating.rating,
                                     "movie_genres": movie_dict[rating.movie_id].genres,
                                     "user_gender": user_dict[rating.user_id].gender,
                                     "user_age": user_dict[rating.user_id].age,
                                     "user_occupation": occupations[int(user_dict[rating.user_id].occupation)],
                                     "user_zipcode": user_dict[rating.user_id].zipcode})

    # Write the information to a file
    detailed_rating_fd = open("detailed_rating_list.txt", "w")
    for detailed_rating in detailed_rating_list:
        detailed_rating_fd.write("{0}, {1}, {2}, {3}, {4}\n".format(
            detailed_rating["rating"],
            detailed_rating["movie_genres"].split("|")[0],
            detailed_rating["user_gender"],
            detailed_rating["user_age"],
            detailed_rating["user_occupation"]))
    detailed_rating_fd.close()

if __name__ == "__main__":
    main()

```

**4/30/2015 更新**
顺便添加一个Ruby的版本（额，好多东西都忘了，边查边写的）

```ruby
# Author:: Donghua Liu (liudonghua123@gmail.com)
# The following documentation need to rewrite according to RDoc convention
# There are a lot of links for refering some basics
# http://rdoc.sourceforge.net/doc/index.html
# http://blog.firsthand.ca/2010/09/ruby-rdoc-example.html
# This is movie class which contains movie_id, title, and genres fields
class Movie
  # http://ruby.about.com/od/oo/ss/Using-Attributes.htm
  # http://www.quora.com/What-are-setters-and-getters-in-Ruby
  attr_accessor :movie_id, :title, :genres

  def initialize (movie_id, title, genres)
    @movie_id = movie_id
    @title = title
    @genres = genres
  end
end

# This is user class which contains user_id, gender, age, occupation and zipcode fields
class User
  attr_accessor :user_id, :gender, :age, :occupation, :zipcode

  def initialize (user_id, gender, age, occupation, zipcode)
    @user_id = user_id
    @gender = gender
    @age = age
    @occupation = occupation
    @zipcode = zipcode
  end
end

# This is movie class which contains user_id, movie_id, rating and timestamp fields
class Rating
  attr_accessor :user_id, :movie_id, :rating, :timestamp

  def initialize (user_id, movie_id, rating, timestamp)
    @user_id = user_id
    @movie_id = movie_id
    @rating = rating
    @timestamp = timestamp
  end
end

def main
  # define movie, user and rating storage
  # http://rubyquicktips.com/post/603292403/accessing-a-hash-with-either-string-or-symbol-keys
  require 'active_support/hash_with_indifferent_access' # HashWithIndifferentAccess.new
  movie_dict = {}
  user_dict = {}
  rating_list = []
  detailed_rating_list = []
  occupations = {0 => 'other', 1 => 'academic/educator', 2 => 'artist', 3 => 'clerical/admin', 4 => 'college/grad student',
                 5 => 'customer service', 6 => 'doctor/health care', 7 => 'executive/managerial', 8 => 'farmer',
                 9 => 'homemaker', 10 => 'K-12 student', 11 => 'lawyer', 12 => 'programmer', 13 => 'retired',
                 14 => 'sales/marketing', 15 => 'scientist', 16 => 'self-employed', 17 => 'technician/engineer',
                 18 => 'tradesman/craftsman', 19 => 'unemployed', 20 => 'writer'}

  root_dir = 'ml-1m'
  # http://stackoverflow.com/questions/377768/string-concatenation-and-ruby
  # http://stackoverflow.com/questions/597488/how-to-do-a-safe-join-pathname-in-ruby
  # http://ruby-doc.org/core-2.1.2/File.html
  movie_file_path = File.join(root_dir, 'movies.dat')
  user_file_path = File.join(root_dir, 'users.dat')
  rating_file_path =File.join(root_dir, 'ratings.dat')

  # http://stackoverflow.com/questions/6012930/read-lines-of-a-file-in-ruby
  File.readlines(movie_file_path).each do |line|
    # The structure of movie files is "MovieID::Title::Genres"
    splitted_fields = line.split('::')
    movie = Movie.new(splitted_fields[0], splitted_fields[1], splitted_fields[2])
    movie_dict[splitted_fields[0]] = movie
  end
  File.readlines(user_file_path).each do |line|
    # The structure of user files is "UserID::Gender::Age::Occupation::Zipcode"
    splitted_fields = line.split('::')
    user = User.new(splitted_fields[0], splitted_fields[1], splitted_fields[2], splitted_fields[3], splitted_fields[4])
    user_dict[splitted_fields[0]] = user
  end
  File.readlines(rating_file_path).each do |line|
    # The structure of movie files is "UserID::MovieID::Rating::Timestamp"
    splitted_fields = line.split('::')
    rating = Rating.new(splitted_fields[0], splitted_fields[1], splitted_fields[2], splitted_fields[3])
    rating_list.push rating
  end

  rating_list.each do |rating|
    detailed_rating_list.push({'rating' => rating.rating,
                               'movie_genres' => movie_dict[rating.movie_id].genres,
                               'user_gender' => user_dict[rating.user_id].gender,
                               'user_age' => user_dict[rating.user_id].age,
                               'user_occupation' => occupations[user_dict[rating.user_id].occupation.to_i],
                               'user_zipcode' => user_dict[rating.user_id].zipcode})
  end

  # http://stackoverflow.com/questions/2777802/how-to-write-to-file-in-ruby
  File.open('detailed_rating_list.txt', 'w') { |detailed_rating_fd|
    detailed_rating_list.each { |detailed_rating|
      detailed_rating_fd.write("#{detailed_rating['rating']},#{detailed_rating['movie_genres'].split('|')[0]},#{detailed_rating['user_gender']},#{detailed_rating['user_age']},#{detailed_rating['user_occupation']}\n")
    }
  }

end

main

```

参考资料
1\. [To Ruby From Python](https://www.ruby-lang.org/en/documentation/ruby-from-other-languages/to-ruby-from-python/)