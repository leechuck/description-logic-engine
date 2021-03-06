# Description Logic Engine

## What this is

This is a description logic engine based on the **ALCQ language (ALC extended with number qualified restrictions)**. It is an implementation of the **Tableau** algorithm: provided an **ABOX** (assertions) and a **TBOX** (terminology definitions), the algorithm will determine satisfiability. It also can handle **Subsumption** questions (*"Is the PREMISE subsumed by the ABOX & TBOX?"*). 

## Syntax

### Code Syntax

This is written in Julia. Prefix syntax is used in the parsing, and Julia code is used to define the description logic statements. The engine has the following structures that can be used to define description logic statements (note the ":" is a julia symbol):


#### ALC Structs
- Concept
- Relation
- Object

#### Number Restrictions and Identity
- :>=
- :<=
- :!=

#### Subsumption
- :subsumes

#### Quantifiers
- :exists
- :all

#### Rule
- :rule 

#### First order logic
- :and 
- :not 
- :or  
- :implies 

#### Description Logic Statements
- EXPRESSION: is a list of ALC Structs, Number Restrictions, Subsumption, Quantifiers, Rules, & First Order Logic.
- ABOX: is a set of expressions
- TBOX: is a dicionary Dict mapping *Concepts => Expressions* 
- PREMISE: is an expression

### Examples


#### Abox and Tbox
```
include("logic_conversion.jl")


# Define Concepts and Relations and Objects
hasChild = Relation("has child")

Person = Concept("Person")
Female = Concept("Female")
Woman = Concept("Woman")
Man = Concept("Man")
Mother = Concept("Mother")

mary = Object("mary")
tom = Object("tom")

# A TBox is a Dictionary (a mapping of Concepts to Definitions)
tbox = Dict(
    Woman => [:and,  Person, Female],
    Man =>   [:and,  Person, [:not, Female]],
    Mother => [:and, Woman,
                     [:exists, [:rule, hasChild, Person]]]
)

# An abox is a collection of logical expressions in Prefix notation
abox = [
    [hasChild, mary, tom],
    [Woman, mary],
    [Person, tom],
    [Mother, mary]
]
```

to check consistency:

```
consistent, model = abox_consistent(abox, tbox)
```

#### Subsumption
```
GoodStudent = Concept("Good Student")
Smart = Concept("Smart")
Studious = Concept("Studious")
attendedBy = Relation("Attended by")
SmartPerson = Concept("Smart Person")

tbox = Dict(
    GoodStudent => [:or,  Smart, Studious],
    SmartPerson => Smart
)

premise = [:subsumes, [:exists, [:rule, attendedBy, [:and, Smart, Studious]]],
                      [:exists, [:rule, attendedBy, GoodStudent]]]


aboxes, answer = premise_subsumes(Set(), tbox, premise)
@test answer == true
```


#### Numbers Restrictions

```
joe = Object("joe")
ann = Object("ann")
eva = Object("eva")
mary = Object("mary")

# Assuming that the names are unique objects
abox = Set([
    [hasChild, mary, ann],
    [hasChild, mary, eva],
    [hasChild, mary, joe],
    [[:<=, 2, [:rule, hasChild, :T]], mary],
])

consistent, box = abox_consistent_with_obj_and_t(abox)
@test consistent == false


# Assuming that the names are not unique objects
abox = Set([
    [hasChild, mary, ann],
    [hasChild, mary, eva],
    [hasChild, mary, joe],
    [[:<=, 2, [:rule, hasChild, :T]], mary],
])

consistent, boxes = abox_consistent_with_t(abox)
```

The function `abox_consistent_with_t` adds allows the use of `:T` (my implementation adds the *T or not T* description to all objects). The function `abox_consistent_with_obj_and_t` additionally adds inequality to all objects to make them all unique.


#### More complicated example

```
r = Relation("r")
s = Relation("s")
A = Concept("A")
B = Concept("B")
C = Concept("C")

#∀r.∀s.A ⊓ ∃r.∀s.B ⊓ ∀r.∃s.C ⊑ ∃r.∃s.(A⊓B⊓C)

one = [:all, [:rule, r, [:all, [:rule, s, A]]]]
two = [:exists, [:rule, r, [:all, [:rule, s, B]]]]
three = [:all, [:rule, r, [:exists, [:rule, s, C]]]]
four = [:exists, [:rule, r, [:exists, [:rule, s,  [:and, A, [:and, B, C]]]]]]

premise = [:subsumes, [:and, [:and, one, two], three], four]

aboxes, answer = premise_subsumes(Set(), Dict(), premise)

@test answer == true


# ∀r.∀s.A ⊓ (∃r.∀s.¬A ⊔ ∀r.∃s.B) ⊑ ∀r.∃s.(A ⊓ B) ⊔ ∃r.∀s.¬B
one = [:all, [:rule, r, [:all, [:rule, s, A]]]]
two = [:exists, [:rule, r, [:all, [:rule, s, [:not, A]]]]]
three = [:all, [:rule, r, [:exists, [:rule, s, B]]]]
four = [:all, [:rule, r, [:exists, [:rule, s, [:and, A, B]]]]]
five = [:exists, [:rule, r, [:all, [:rule, s, [:not, B]]]]]

premise = [:subsumes, [:and, one, [:or, two, three]], [:or, four, five]]

aboxes, answer = premise_subsumes(Set(), Dict(), premise)

@test answer == true
```


## Tests
Run the following tests:

```
include("test_number.jl")
include("test_logic_conversion.jl")
```

## Assignment
The asignment questions are in `assignment_questions.jl`. They are framed as Julia tests, so you can run them with:

```
include("assignment_questions.jl"
```

All the tests will pass. You can see what the output of each problem in the file. For example:

```
abox = [
    [hasChild, mary, tom],
    [Woman, mary],
    [Person, tom],
    [Mother, mary]
]

result =  Set([
    [Concept("Female"), Object("mary")],
    [[:and, [:and, Concept("Person"), Concept("Female")], [:exists, [:rule, Relation("has child"), Concept("Person")]]], Object("mary")],
    [[:exists, [:rule, Relation("has child"), Concept("Person")]], Object("mary")],
    [Relation("has child"), Object("mary"), Object("tom")],
    [Concept("Person"), Object("tom")],
    [Concept("Person"), Object("mary")],
    [[:and, Concept("Person"), Concept("Female")], Object("mary")]
    ])

consistent, model = abox_consistent(abox, tbox)

@test consistent == true
@test model == result
```

The variable `result` contains the model given by the algorithm, and the variable `consistent` shows that the abox is consistent. Look through the rest of the test cases to see the rest of the answers.

## Julia Version & Installation 

#### Julia Version
`
Julia Version 1.3.1
Commit 2d5741174c (2019-12-30 21:36 UTC)
Platform Info:
  OS: Windows (x86_64-w64-mingw32)
  CPU: Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz
  WORD_SIZE: 64
  LIBM: libopenlibm
  LLVM: libLLVM-6.0.1 (ORCJIT, skylake)
Environment:
  JULIA_EDITOR = "C:\Users\James\AppData\Local\Programs\Microsoft VS Code\Code.exe"
  JULIA_NUM_THREADS = 1
`

#### Installation
1. Run `git clone https://github.com/jamesaud/description-logic-engine`

2. `cd logic_engine_2`

3. Run julia, and then you can run the `include` statements. 
