name: title
class: title-slide-image
background-image: url(img/test.jpg)
<!-- Image source: -->

# Unordered containers and std::hash

## Internals, efficiency, examples.

August 25, 2016

???

This talk presents basic knowledge

---

# What is a pointer?

--

.info[
A **pointer** is a programming language data type whose value refers directly to (or "points to") another value stored elsewhere in the computer memory using its address.

.right[*-- Wikipedia*]
]

---

# What is a pointer?

--

.info[
**Pointers** due to their ambiguous nature are the source of most problems and bugs in C++ code.
Lack of understanding between code architect and the user results in crashes, asserts and memory leaks!!!

.right[*-- Mateusz Pusz &#9786;*] <!-- White Smiling Face -->
]

---

# Pointers misuse leads to
--

<p class="floating-label-1" style="top: 50px; left: 600px; transform: rotate(-5deg);">Null Pointer Dereference</p>

--

<p class="floating-label-1" style="top: 150px; left: 100px; transform: rotate(10deg);">Resource leaks</p>

--

<p class="floating-label-1" style="top: 400px; left: 300px; transform: rotate(5deg);">Buffer Overflows</p>

--

<p class="floating-label-2" style="top: 10px; left: 200px; transform: rotate(-7deg);">Hangs!</p>

--

<p class="floating-label-2" style="top: 350px; left: 900px; transform: rotate(-30deg);">Crashes!</p>

--

<p class="floating-label-3" style="top: 240px; left: 200px; transform: rotate(-10deg);">Stability!!!</p>

--

<p class="floating-label-3" style="top: 180px; left: 700px; transform: rotate(5deg);">Security!!!</p>

---

class: big-code

# Quiz - Guess what?

```cpp
B* func(A* arg);
```

---

class: big-code

# Quiz - Guess what?

```cpp
person* add(name* n);
```

--

.note[
Is it better now?  
Is it enough?  
Do you know how to *use* that function?  
Do you know how to *implement* that function?
]

---

# add() usage – Take \#1

```cpp
person* add(name* n);

void foo()
{
* person* p = add(new name{"Mateusz Pusz"});
  assert(p != nullptr);
  process(p->id(), p->name());
  delete p;
}
```

--

.note[
Is that a valid usage of `add()` interface?  
What are potential problems with above code?
]

---

# add() usage – Take \#2

```cpp
person* add(name* n);

void foo()
{
  name n{"Mateusz Pusz"};
* person* p = add(&n);
  if(p != nullptr)
    process(p->id(), p->name());
}
```

--

.note[
Is that a valid usage of `add()` interface?
]

---

# add() usage – Take \#3

```cpp
person* add(name* n);

void foo()
{
  name* n = new name{"Mateusz Pusz"};
* person* p = add(n);
  if(p != nullptr)
    process(p->id(), p->name());
  delete n;
}
```

--

.note[
Is that a valid usage of `add()` interface?  
What are potential problems with above code?
]

---

# add() usage – Take \#4

```cpp
person* add(name* n);

void foo()
{
  name names[] = {"Mateusz Pusz", "Jan Kowalski", ""};
* person* people = add(names);
  assert(people != nullptr);
  for(int i=0; i<sizeof(names)/sizeof(*names) - 1; ++i)
    process(people[i].id(), people[i].name());
  delete[] people;
}
```

--

.note[
Is that a valid usage of `add()` interface?  
What are potential problems with above code?
]

---

layout: true
class: small-code

.left-column[
### 1
```cpp
void foo()
{
  person* p = add(new name{"Mateusz Pusz"});
  assert(p != nullptr);
  process(p->id(), p->name());
  delete p;

}
```

### 2
```cpp
void foo()
{
  name n{"Mateusz Pusz"};
  person* p = add(&n);
  if(p != nullptr)
    process(p->id(), p->name());


}
```
]

.right-column[
### 3
```cpp
void foo()
{
  name* n = new name{"Mateusz Pusz"};
  person* p = add(n);
  if(p != nullptr)
    process(p->id(), p->name());
  delete n;
}
```

### 4
```cpp
void foo()
{
  name names[] = {"Mateusz Pusz", "Jan Kowalski", ""};
  person* people = add(names);
  assert(people != nullptr);
  for(int i=0; i<sizeof(names)/sizeof(*names) - 1; ++i)
    process(people[i].id(), people[i].name());
  delete[] people;
}
```
]

---

# Which one is correct?

---

# Quiz - Match Implementation

.slide-center[
```cpp
std::deque<person> people;

person* add(name* n)
{
  people.emplace_back((n != nullptr) ? *n : "anonymous");
  return &people.back();
}
```
]

---

layout: false
class: small-code

# Quiz - Match Implementation

.left-column[
### 1
```cpp
void foo()
{
  person* p = add(new name{"Mateusz Pusz"});
  assert(p != nullptr);
  process(p->id(), p->name());
  delete p;

}
```

.transparent[### 2
```cpp
void foo()
{
  name n{"Mateusz Pusz"};
  person* p = add(&n);
  if(p != nullptr)
    process(p->id(), p->name());


}
```
]
]

.right-column[
### 3
```cpp
void foo()
{
  name* n = new name{"Mateusz Pusz"};
  person* p = add(n);
  if(p != nullptr)
    process(p->id(), p->name());
  delete n;
}
```

### 4
```cpp
void foo()
{
  name names[] = {"Mateusz Pusz", "Jan Kowalski", ""};
  person* people = add(names);
  assert(people != nullptr);
  for(int i=0; i<sizeof(names)/sizeof(*names) - 1; ++i)
    process(people[i].id(), people[i].name());
  delete[] people;
}
```
]

.slide-center[
```cpp
person* add(name* n)
{
  assert(n);
  int num = 0;
  for(auto ptr = n; *ptr != ""; ++ptr)
    ++num;
  person* p = new person[num];
  for(auto i = 0; i<num; ++i)
    p[i].name(n[i]);
  return p;
}
```
]

---

class: small-code

# Quiz - Match Implementation

.left-column[
### 1
```cpp
void foo()
{
  person* p = add(new name{"Mateusz Pusz"});
  assert(p != nullptr);
  process(p->id(), p->name());
  delete p;

}
```

.transparent[### 2
```cpp
void foo()
{
  name n{"Mateusz Pusz"};
  person* p = add(&n);
  if(p != nullptr)
    process(p->id(), p->name());


}
```
]
]

.right-column[
### 3
```cpp
void foo()
{
  name* n = new name{"Mateusz Pusz"};
  person* p = add(n);
  if(p != nullptr)
    process(p->id(), p->name());
  delete n;
}
```

.transparent[### 4
```cpp
void foo()
{
  name names[] = {"Mateusz Pusz", "Jan Kowalski", ""};
  person* people = add(names);
  assert(people != nullptr);
  for(int i=0; i<sizeof(names)/sizeof(*names) - 1; ++i)
    process(people[i].id(), people[i].name());
  delete[] people;
}
```
]
]

.slide-center[
```cpp
person* add(name* n)
{
  assert(n != nullptr);
  return new person{n};
}
```
]

---

class: small-code

# Quiz - Match Implementation

.transparent[
.left-column[
### 1
```cpp
void foo()
{
  person* p = add(new name{"Mateusz Pusz"});
  assert(p != nullptr);
  process(p->id(), p->name());
  delete p;

}
```

### 2
```cpp
void foo()
{
  name n{"Mateusz Pusz"};
  person* p = add(&n);
  if(p != nullptr)
    process(p->id(), p->name());


}
```
]
]

.right-column[
### 3
```cpp
void foo()
{
  name* n = new name{"Mateusz Pusz"};
  person* p = add(n);
  if(p != nullptr)
    process(p->id(), p->name());
  delete n;
}
```

.transparent[### 4
```cpp
void foo()
{
  name names[] = {"Mateusz Pusz", "Jan Kowalski", ""};
  person* people = add(names);
  assert(people != nullptr);
  for(int i=0; i<sizeof(names)/sizeof(*names) - 1; ++i)
    process(people[i].id(), people[i].name());
  delete[] people;
}
```
]
]

.slide-center[
```cpp
std::deque<person> people;

person* add(name* n)
{
  if(n == nullptr)
    return nullptr;
  auto it = find_if(begin(people), end(people),
                    [&](person& p)
                    { return p.name() == *n; });
  if(it != end(people))
    return nullptr;
  people.emplace_back(*n);
  return &people.back();
}
```
]

---

class: small-code

# Quiz - Match Implementation

.transparent[
.left-column[
### 1
```cpp
void foo()
{
  person* p = add(new name{"Mateusz Pusz"});
  assert(p != nullptr);
  process(p->id(), p->name());
  delete p;

}
```

### 2
```cpp
void foo()
{
  name n{"Mateusz Pusz"};
  person* p = add(&n);
  if(p != nullptr)
    process(p->id(), p->name());


}
```
]

.right-column[
### 3
```cpp
void foo()
{
  name* n = new name{"Mateusz Pusz"};
  person* p = add(n);
  if(p != nullptr)
    process(p->id(), p->name());
  delete n;
}
```

### 4
```cpp
void foo()
{
  name names[] = {"Mateusz Pusz", "Jan Kowalski", ""};
  person* people = add(names);
  assert(people != nullptr);
  for(int i=0; i<sizeof(names)/sizeof(*names) - 1; ++i)
    process(people[i].id(), people[i].name());
  delete[] people;
}
```
]
]

---

layout: false
class: big-code

# Is there a better solution?

```cpp
person*
add(name* n);
```

--

```cpp
std::unique_ptr<person>
add(std::unique_ptr<name> n);
```

--

.note[
Do you know how to *use* that function?  
Do you know how to *implement* that function?
]

---

# Using C++ the right way – Case #1

.left-column[
```cpp
person*
add(name* n)
{
  assert(n != nullptr);
  return new person{n};
}
```

```cpp
void foo()
{
  person* p = add(
    new name{"Mateusz Pusz"});
  assert(p != nullptr);
  process(p->id(), p->name());
  delete p;
}
```
]

---

count: false

# Using C++ the right way – Case #1

.transparent[.left-column[
```cpp
person*
add(name* n)
{
  assert(n != nullptr);
  return new person{n};
}
```

```cpp
void foo()
{
  person* p = add(
    new name{"Mateusz Pusz"});
  assert(p != nullptr);
  process(p->id(), p->name());
  delete p;
}
```
]]

.right-column[
```cpp
std::unique_ptr<person>
add(std::unique_ptr<name> n)
{
  assert(n != nullptr);
  return std::make_unique<person>(std::move(n));
}
```

```cpp
void foo()
{
  auto p = add(
    std::make_unique<name>("Mateusz Pusz"));
  assert(p != nullptr);
  process(p->id(), p->name());

}
```
]

---

class: big-code

# Is there a better solution?

```cpp
person*
add(name* n);
```

--

```cpp
person&
add(std::optional<name> n);
```

--

.note[
Do you know how to *use* that function?  
Do you know how to *implement* that function?
]

---

# Using C++ the right way – Case #2

.left-column[
```cpp
std::deque<person> people;

person* add(name* n)
{
  people.emplace_back(
    (n != nullptr) ? *n : "anonymous");
  return &people.back();
}
```

```cpp
void foo()
{
  name n{"Mateusz Pusz"};
  person* p = add(&n);
  if(p != nullptr)
    process(p->id(), p->name());
}
```
]

---

count: false

# Using C++ the right way – Case #2

.transparent[.left-column[
```cpp
std::deque<person> people;

person* add(name* n)
{
  people.emplace_back(
    (n != nullptr) ? *n : "anonymous");
  return &people.back();
}
```

```cpp
void foo()
{
  name n{"Mateusz Pusz"};
  person* p = add(&n);
  if(p != nullptr)
    process(p->id(), p->name());
}
```
]]

.right-column[
```cpp
std::deque<person> people;

person& add(std::optional<name> n)
{
  people.emplace_back(
      n ? std::move(*n) : "anonymous");
  return people.back();
}
```

```cpp
void foo()
{
  person& p = add(name{"Mateusz Pusz"});
  process(p.id(), p.name());


}
```
]

---

class: big-code

# Is there a better solution?

```cpp
person*
add(name* n);
```

--

```cpp
std::tuple<person&, bool>
add(name n);
```

--

.note[
Do you know how to *use* that function?  
Do you know how to *implement* that function?
]

---

class: small-code

# Using C++ the right way – Case #3

.left-column[
```cpp
std::deque<person> people;

person* add(name* n)
{
  if(n == nullptr) return nullptr;
  auto it = find_if(
       begin(people), end(people),
       [&](person& p) { return p.name() == *n; });
  if(it != end(people))
    return nullptr;
  people.emplace_back(*n);
  return &people.back();
}
```

```cpp
void foo()
{
  name* n = new name{"Mateusz Pusz"};
  person* p = add(n);
  if(p != nullptr)
    process(p->id(), p->name());
  delete n;
}
```
]

---

class: small-code
count: false

# Using C++ the right way – Case #3

.transparent[
.left-column[
```cpp
std::deque<person> people;

person* add(name* n)
{
  if(n == nullptr) return nullptr;
  auto it = find_if(
       begin(people), end(people),
       [&](person& p) { return p.name() == *n; });
  if(it != end(people))
    return nullptr;
  people.emplace_back(*n);
  return &people.back();
}
```

```cpp
void foo()
{
  name* n = new name{"Mateusz Pusz"};
  person* p = add(n);
  if(p != nullptr)
    process(p->id(), p->name());
  delete n;
}
```
]
]

.right-column[
```cpp
std::deque<person> people;

std::tuple<person&, bool> add(name n)
{
  auto it = find_if(
       begin(people), end(people),
       [&](person& p) { return p.name() == n; });
  if(it != end(people))
    return { *it, false };
  people.emplace_back(std::move(n));
  return { people.back(), true };

}
```

```cpp
void foo()
{
  auto r = add(name{"Mateusz Pusz"});
  if(std::get<bool>(r))
    process(std::get<0>(r).id(), std::get<0>(r).name());


}
```
]

---

class: big-code

# Is there a better solution?

```cpp
person*
add(name* n);
```

--

```cpp
std::vector<person>
add(std::vector<name> names);
```

--

.note[
Do you know how to *use* that function?  
Do you know how to *implement* that function?
]

---


class: small-code

# Using C++ the right way – Case #4

.left-column[
```cpp
person* add(name* n)
{
  assert(n);
  int num = 0;
  for(auto ptr = n; *ptr != ""; ++ptr)
    ++num;
  person* p = new person[num];
  for(auto i = 0; i<num; ++i)
    p[i].name(n[i]);
  return p;
}
```

```cpp
void foo()
{
  name names[] = {"Mateusz Pusz", "Jan Kowalski", ""};
  person* people = add(names);
  assert(people != nullptr);
  for(int i=0; i<sizeof(names)/sizeof(*names) - 1; ++i)
    process(people[i].id(), people[i].name());
  delete[] people;
}
```
]

---

class: small-code
count: false

# Using C++ the right way – Case #4

.transparent[.left-column[
```cpp
person* add(name* n)
{
  assert(n);
  int num = 0;
  for(auto ptr = n; *ptr != ""; ++ptr)
    ++num;
  person* p = new person[num];
  for(auto i = 0; i<num; ++i)
    p[i].name(n[i]);
  return p;
}
```

```cpp
void foo()
{
  name names[] = {"Mateusz Pusz", "Jan Kowalski", ""};
  person* people = add(names);
  assert(people != nullptr);
  for(int i=0; i<sizeof(names)/sizeof(*names) - 1; ++i)
    process(people[i].id(), people[i].name());
  delete[] people;
}
```
]]

.right-column[
```cpp
std::vector<person> add(std::vector<name> names)
{
  std::vector<person> p;
  p.reserve(names.size());
  for(auto& n : names)
    p.emplace_back(std::move(n));
  return p;



}
```

```cpp
void foo()
{
  auto people = add({"Mateusz Pusz", "Jan Kowalski"});
  for(auto& p : people)
    process(p.id(), p.name());
  
  
  
}
```
]

---

class: section-title-slide
background-image: url(img/no-excuses.jpg)
<!-- Image source: https://pixabay.com/en/shield-traffic-sign-note-billboard-417827/ -->

---


# Pointers usage in ANSI C

| Argument type            | Pointer argument declaration |
|--------------------------|------------------------------|
| Mandatory big value      | `void foo(A* in);`           |
| Output function argument | `void foo(A* out);`          |
| Array                    | `void foo(A* array);`        |
| Optional value           | `void foo(A* opt);`          |
| Ownership passing        | `void foo(A* ptr);`          |

--

.note[
Pointer ambiguity makes it really hard to understand the intent of the interface author.
]

---

# Doing it C++ way

| Argument type            | Pointer argument declaration                                   |
|--------------------------|----------------------------------------------------------------|
| Mandatory big value      | `void foo(const A& in);`                                       |
| Output function argument | `A foo();` or `std::tuple<...> foo();` or `void foo(A& out);`  |
| Array                    | `void foo(const std::vector<A>& a);`                           |
| Optional value           | `void foo(std::optional<A> opt);` or `void foo(A* opt);`       |
| Ownership passing        | `void foo(std::unique_ptr<A> ptr);`                            |

--

.note[
Use above Modern C++ constructs to explicitly state your design intent.
]

---

class: big-code, center

# Quiz – Guess what?

```cpp
B foo(std::optional<A> arg);
```

```cpp
const A& foo(const std::array<A, 3>& arg);
```

```cpp
std::unique_ptr<B> foo(A arg);
```

```cpp
std::vector<B> foo(const A& arg);
```

---

class: section-title-slide
background-image: url(img/takeaway.png)
<!-- Image source: https://pixabay.com/en/brown-paperbags-lunch-bags-309963/ -->

# Takeaways

---

class: image-right-slide
background-image: url(img/swiss-army-knife.jpg)
<!-- Image source: https://pixabay.com/en/army-blade-compact-cut-equipment-2185 -->

# C++ is not ANSI C!!!

C++ is a powerful tool:
- strong type system
- better abstractions
- templates
- C++ STD library

---

class: section-title-slide
background-image: url(img/questions.jpg)
<!-- Image source: https://pixabay.com/en/idea-question-light-bulb-1296140/ -->

---

class: section-title-slide
background-image: url(img/warning.png)
<!-- https://openclipart.org/detail/18736/programming-addictive-sign -->
