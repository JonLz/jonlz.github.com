---
layout: post
title: "Things I Learned in Life"
date: 2012-10-19 16:22
comments: true
categories: [Conway's Game of Life, arrays, 2d arrays, variable references, ruby]
---
Conway's Game of Life
-
If you aren't familiar with Conway's Game of Life, it's a set of rules that simulates cellular evolution based on the initial state of a group of cells.  The game consists of a simple ruleset that algorithmically determines a cell's state based on its surroundings.  It's simple in practice but some of the resulting evolutions are fascinating.  It's also a great advanced beginner programming challenge!
<!--more-->
<hr/>
Here are the rules courtesy of Wikipedia:  

The universe of the Game of Life is an infinite two-dimensional orthogonal grid of square cells, each of 
which is in one of two possible states, alive or dead. Every cell interacts with its eight neighbours, 
which are the cells that are horizontally, vertically, or diagonally adjacent. At each step in time, the 
following transitions occur:  

+ Any live cell with fewer than two live neighbours dies, as if caused by under-population.
+ Any live cell with two or three live neighbours lives on to the next generation.
+ Any live cell with more than three live neighbours dies, as if by overcrowding.
+ Any dead cell with exactly three live neighbours becomes a live cell, as if by reproduction.

The initial pattern constitutes the seed of the system. The first generation is created by applying the above rules simultaneously to every cell in the seed—births and deaths occur simultaneously, and the discrete moment at which this happens is sometimes called a tick (in other words, each generation is a pure function of the preceding one). The rules continue to be applied repeatedly to create further generations.

Example:  
![Glider](http://upload.wikimedia.org/wikipedia/commons/e/e5/Gospers_glider_gun.gif)
<hr/>
 
Programming it in Ruby
-
I think the approach I took was rather brute force but it worked rather well and is fairly intuitive (at least to me it is!).  My approach was to create a blank board of size rows x cols and set the state with a pre-defined array of living cells.  I then iterate through each successive state by checking each cell, counting the number of alive neighbors, and applying the rules of the game to determine whether to kill the cell, let it live, or bring it to life.  Simple enough right?

If you want to follow along here is <a href="https://github.com/JonLz/rubysamples/blob/master/conways%20game%20of%20life.rb" target="_blank">my solution</a>

###The Board
I represented the board with a 2 dimensional array and here I learned something really useful.  And I thought this was going to be the easy part!  This was my first attempt at creating the @board array:
```  
	@board = Array.new(@rows, Array.new(@cols, DEAD_CELL))
```  
This code looks really intuitive at first glance.  Create a new array with @rows number of rows, and in each of those rows create a new array with @cols number of cols with default values of DEAD_CELLs which is a constant that has the character for the dead cells in it.  But what this is really doing is creating an array where **ALL** of your columns are just references to the same column array!

I was tearing my hair out trying to figure out why this wasn't working until I went back to look at the documentation for Array.new which reads **new(size=0, obj=nil)**.  It clicked, aha it references to an object.  So you actually need to use the block method of array creation: **new(size) {|index| block }**
```  
	@board = Array.new(@rows) {Array.new(@cols, DEAD_CELL)}
```  
So remarkably similar but it's actually creating new arrays in memory each time now.

###Iterating States
The code to cycle through each cell on the board isn't very exciting but I did run into another problem.  I guess I didn't learn my referencing problem from my board creation fiasco but this is something I really didn't even know *could* be a problem.

My iteration technique was to create a new blank board which would become the next iterative state based on the current state of the actual @board instance variable.  This is how I did it at first:
```  
def initialize
  ...initialize the board code...
  @static_board = @board
end

def next_state
  new_board = @static_board
  ...iterative code...
  @board = new_board
end
```  

This problem tied me up for a good hour trying to figure out what the heck was going on.  I was dumping puts statements everywhere to try and see what iteration I was on, what the iterator values were, the row and column variables, the number of alive neighbors, etc etc.  It wasn't until I went so far as to spit the board out on every successive cell check that I figured out what was happening.  My new board and static board were all actually just references to the original board so I was never actually creating a fresh new board!  Every time I would "set" my new board, the actual @board would get altered and it drove me nuts.  Oh the irony.  Here is my working version:
```  
def initialize
  @board = base_array
end

def base_array
  Array.new(@rows) {Array.new(@cols, DEAD_CELL)}
end

def next_state
  new_board = base_array
  ...iterative code...
  @board = new_board
end
```  

###Other Notes
Another neat trick when working with 2d arrays is that you can do this to loop through every surrounding cell easily.  It's pretty great that you can use each on something as basic as a range and it just works as you would expect it to.
```  
(-1..1).each do |x|
  (-1..1).each do |y|
```  
Coincidentally, I wish I had thought about this when I was writing my word scramble solver as when I did that I hardcoded an array of all adjacent cells. :)

You can also override the to_s function and make your own, so that every time you call your objects inspect method it will run your own to_s.
```  
def Game
  def to_s
    puts "hi"
  end
end

p Game.new # "hi" is displayed
```

I'd recommend if you're comfortable with array iteration to give this game a try, it's a great learning exercise and fun to play around once you've got it all working.  Long live R-Pentomino!