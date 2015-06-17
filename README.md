
README - Final project Shrdlu - Artificial Intelligence 	
Group Potatissallad/Goto fail;				
2015 - period 4							

The project is entirely coded in Typescript and fully working in the html view.

-= Features =-
 
 "Laziest-first strategy" which means that to resolve ambiguities (parsing as well as interpretation),
 the heuristic will be run for each possibility of goal and the lowest score will be chosen.
 
 - Implemented the any, all feature
 - When "the" is used and matches several possibilities, we get notified how much objects are designed.
 - When "a" (or any) is used, the simplest move is chosen
 - All these relations have been implemented: on top, inside, beside, above, under, right of, left of, holding.
 - It is possible to ask to "take" an object and then refer to it by "it" in the next utterance.
 - It is possible to ask several propositions at a time with relations "and" and "or":
 	For example: "Put a table in a red box and take a brick or put a box on the floor."
 	This would try to resolve "put a table in a red box" and "take a brick" simultaneously, or "put a box on
 	the floor". The cheapest solution according to the heuristic would be chosen.
 
 See the example section at the end for some interesting demos!
 
 
-= Parser modifications =-

 [see Parser.ts file, first function]
 The parser has been modified to parse sentences with "and"/"or", with priority on "and" as in boolean algebra.
 
 
-= Interpreter implementation =-
 
 Some comments are writen in the code.
 
 
-= Planner implementation =-
 
 --> Graph search: AStar algorithm [implemented in AStar.ts file]
 
	 We represent one state as a configuration of the piles. The position of the arm and whether
	 he's holding anything is not taken into account. If we start or finish with an object in the arm,
	 the additional moves are made before or after the AStar algorithm. (If there's an object hold by
	 the arm at start, we "drop" it before launching the AStar research, and if we want to hold an object
	 at the end, we just ensure in the AStar that this object is on top of a pile, and grab it after the AStar.)
	 
	 So, one transition is called a "Move" and is determined by a "pick" column and a "drop" column.
	 We build the graph dynamically. We start from the current state, then build every neighbor state by
	 trying every possible Move (see computeNeighbor method of class Node). Then we compute the path cost
	 and heuristic for each neighbor and add them to the frontier of the AStar search.
	 A same state may be reached by different path, we prevent this by using a string representation of
	 a state (hash) so that we can compare if two instances of a Node are actually the same "State". We would
	 set this Node with the shortest path.
	 
	 Graph representation:
	 	Node with the attributes:
	 	- content: Planner.State
	 			Planner.State contains:
	 			- stacks: string[][] the stacks of objects
	 			- moves: Move[] the list of moves from start to this state
	 			- hash: a string representing the state uniquely
	 	- neighbors: Arc[]
	 			Arc contains:
	 			- destination: Node a destination node
	 			- weight: associated weight (in our case, always 1)
	 	- g_score: number the cost of the path from start
	 	- f_score: number evaluation of the total cost: g_score + heuristic

 --> Heuristic: [implemented in Planner.ts, at the end of the file]
 
	 * Prototype:
	 function heuristic(stacks: string[][], goalConditions: Interpreter.Literal[]) : number
	 
	 It computes an under-estimate of the number of Moves to perform in order to fulfill each goal conditions.
	 
	 * Args:
	 - stacks: the objects in stacks for a given state
	 - goalConditions: list of Literals describing binary relations (or unary for the holding constraint)
	 between 2 objects. Ex: ontop<"a","floor">, leftof<"d","b">
	 
	 It relies on the number of object to be moved.
	 There are different cases depending on the relation:
	 "ontop"/"inside", "beside", "above"/"under", "rightof"/"leftof", "holding".
	 
	 The goal is to fill a list of the objects we need to move, without duplicate, and simply return the length
	 of this list. The number of Moves needed would be at least the number of objects to be moved and therefore,
	 we guarantee the admissibility of the heuristic. If all the goal constraints are reached, the heuristic is
	 null (0).
	 
	 * Example:
			Stacks:
			    c
			_ a b _
		 
		 	Goal:
		 ontop<"a","b">, leftof<"c","b">, leftof<"c","a">
		 
		 objMove=[]
		 for ontop<"a","b"> : we need to move all objects above a and b, plus a itself: objMove=[a,c]
		 for leftof<"c","b"> : we need to move all objects from above one of both concerned objects,
		 						plus one of themselves.
		 						b has one object over itself, so it should be faster to move c,
		 						which is already in objMove. So objMove=[a,c]
		 for leftof<"c","a"> : same consideration and we still have objMove=[a,c].
		 
		 The heuristic value for this state according to these goal conditions is therefore 2.
		 Here, we can see that we can indeed solve the problem in two moves. If there weren't the left-most stack,
		 it would have been 3 moves.
	 
	 Our AStar algorithm with this admissible heuristic can therefore reach the goalConditions in the fewest
	 Moves possible!
	 
	 Note that the optimisation is on the number of moves and not in the distance travelled by the arm.
	 
	 This could have been adapted to this method as well, but we didn't try to implement it. (The transition
	 cost in the graph wouldn't be always 1 anymore but the distance between both columns + pick and drop.)


-= Examples =-

[To run in complex world. /!\ Reset the world for each example.]

- Take the yellow object.
- Take a yellow object. (The easiest one to take will be chosen)
- Put all red objects above a yellow object on the floor.
- Put the white ball right of all objects.
- Put a red table on the floor and put a box on the floor and put a blue table in a box and take a brick.
- Put a table in a red box and take a brick or put a box on the floor.
- Put all boxes on the floor and take a yellow object.
- Put a black ball left of all objects.



INITIAL PROJECT: The Shrdlite course project
============================

Shrdlite is a programming project in Artificial Intelligence, a course given 
at the University of Gothenburg and Chalmers University of Technology.
For more information, see the course webpages:

- <http://www.cse.chalmers.se/edu/course/TIN172/>

The goal of the project is to create an interpreter and a planner so that
a person can control a robot in a blocks world to move around objects,
by giving commands in natural language.

To make the project more interesting, there is a web-based graphical 
interface from which the user can interact with the blocks world.

The interface is written in TypeScript (which compiles into Javascript),
and it can be run in several different modes:

- as a HTML web application, using SVG animations for displaying the world

- as a text application, using ANSI graphics for displaying the world
  (requires an ASNI-capable terminal, and that Node.JS is installed)

- as an offline text application, where input is provided at the command line
  (requires that Node.JS is installed, but nothing of the terminal)

To be able to run the system you need to install Node.JS and TypeScript.
Do that. Now.


What is already implemented and what is missing
------------------------------------------------

The natural language parser is already implemented using the 
[Nearley parsing library] (https://github.com/Hardmath123/nearley).

Furthermore, there are three different implementations of the blocks
world: the SVGWorld, the ANSIWorld and the simple TextWorld.

What is not implemented correctly is the natural language interpreter
and the robot planner. What you are given are stubs that return
a dummy interpretation resp. a dummy plan. Your goal is to implement
the interpreter and the planner so that the robot behaves as it should.


Compiling to Javascript or using Ajax CGI
------------------------------------------

The preferred way to implement this is to write your programs in a 
language that can be compiled directly into Javascript, such as
TypeScript. The advantage with this is that you can use
all three ways of interacting (web, text and offline), and that there's
much less overhead when running. The (possible) disadvantage is that 
you cannot use any programming language.


Using TypeScript
-----------------

TypeScript is a typed superset of JavaScript that compiles to plain JavaScript.
It is open-source and not specific to any browser or operating system.
(And it's developed by Microsoft...)

For information about the language, please visit the official site:

- <http://www.typescriptlang.org/>

### Using another language that can be compiled to Javascript

The surrounding code for the Shrdlite project is all written in TypeScript,
which is an argument for continuing with that language. But there are other
alternatives that should be possible to use, such as:

- [CoffeeScript](http://coffeescript.org) is like a more readable version 
  of Javascript, with a very simple one-to-one translation into Javascript.

- [PureScript](http://www.purescript.org) is very inspired from Haskell, with 
  static types, higher-order functions and Haskell-like syntax.

Using Ajax CGI and a local web server
--------------------------------------

(Note: you don't need this if you don't use the CGI approach)

If you really don't want to implement in TypeScript (or JavaScript or CoffeeScript or ...), 
you can create a CGI script that the HTML file communicates with.
To be able to use this, and to make the following minor change to the file `shrdlite.html`:

- comment the line importing the file `shrdlite-html.js`, and
  instead uncomment the line importing the file `shrdlite-ajax.js`

To be able to run the graphical interface you need a web server. 
There are several options (a very common one is Apache), but for this
project it is enough to use Python's built-in server. 

### Using the Python 3 web server

For this you need to have Python 3 installed. To start the server, 
just run this from the command line, from the same directory as the 
file `shrdlite.html`:

    python3 -m http.server --cgi 8000

Now let the webserver keep running and browse to any of these addresses:

- <http://localhost:8000/shrdlite.html>
- <http://127.0.0.1:8000/shrdlite.html>
- <http://0.0.0.0:8000/shrdlite.html>

Your CGI script has to be executable and reside in the `cgi-bin` directory.
There is an example dummy CGI Python 3 script in the file `shrdlite_cgi.py`.

### Using another programming language via CGI

If you want to use another language that Python, you can either call the other
language from within Python, or use another web server. E.g., if you want to 
use Haskell, there are lots of opportunities (such as Happstack or Snap).

Note that if you choose to use another web server, you have to do some changes 
in the file `shrdlite-ajax.ts`, depending on your choice of server.


Additional information
-----------------------

There is a Makefile if you want to use the GNU Make system. Here's what it can do:

- `make clean`: Removes all auto-generated Javascript files
- `make all`: Calls TypeScript and Closure for each target
- `make html | ajax | ansi | offline`:
  Calls TypeScript and Closure for the given target,
  i.e., it compiles the file `shrdlite-X.ts` into `shrdlite-X.js`

### Data structures

You probably want to use some kind of collection datatype (such as a heap
and/or priority queue), so here are two possible TypeScript libraries:

- [TypeScript-STL] (https://github.com/vovazolotoy/TypeScript-STL)
- [typescript-collections] (https://github.com/basarat/typescript-collections)

If you're using another language (such as Haskell or Java), please see the 
public libraries of that language.

### Using JavaScript modules in TypeScript

If you want to use standard JavaScript libraries in TypeScript, you have to
have a TypeScript declaration file for that library. 
The [DefinitelyTyped library] (https://github.com/borisyankov/DefinitelyTyped)
contains declaration files for several libraries, such as the following two:

- `node.d.ts`
- `jquery.d.ts`

### JavaScript chart parser

The parser is generated by [Nearley] (http://github.com/Hardmath123/nearley).
The grammar is in the file `grammar.ne`, and it is compiled into the 
Javascript file `grammar.js`. You don't have to install Nearley if you 
don't plan to make any changes in the grammar.


List of files
--------------

BSD Makefile for automatically creating `.js` files from `.ts` files:
- `Makefile`

Main browser files:
- `shrdlite.html`, `shrdlite.css`

Wrapper files for the browser-based interfaces:
- `shrdlite-html.ts`, `shrdlite-ajax.ts`

Wrapper files for the Node.JS-based interfaces:
- `shrdlite-ansi.ts`, `shrdlite-offline.ts`

Main TypeScript module:
- `Shrdlite.ts`

TypeScript interfaces and classes for the different implementations of the blocks world:
- `World.ts`, `SVGWorld.ts`, `TextWorld.ts`, `ANSIWorld.ts`, `ExampleWorlds.ts`

TypeScript modules for parsing, interpretation and planning:
- `Parser.ts`, `Interpreter.ts`, `Planner.ts`

Grammar files used by the Nearley chartparser:
- `grammar.js`, `grammar.ne`

Example CGI script that is called by the Ajax web interface:
- `cgi-bin/shrdlite_cgi.py`

TypeScript declaration files for non-TypeScript libraries:
- `lib/jquery.d.ts`, `lib/node.d.ts`

External Javascript libraries:
- `lib/jquery-1.11.0.min.js`, `lib/nearley.js`

Assorted documentation (currently only the TypeScript language definition):
- `doc`

