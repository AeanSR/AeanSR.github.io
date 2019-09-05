---
layout: post
title: Dynamic Type Switch in C++
---

I recently worked on a compiler in a research project.

Compilers usually get loads of types of structs to be programmed. Every syntax elements and IRs got their class.
A piece of syntax element definitions is shown below.

<div><button class="collapsible">Collapsed</button><div class="collapsible-content">
<pre><code>
// ...

struct unary_t : public ast_node_t {
  std::shared_ptr<unaryexpr_t> lhs;                                                                                    
  int type; 
  enum { INC, DEC, PROD, ALL, POS, NEG, NOT, REV, ANY, SIZEOF, TYPEOF };                                               

  virtual bool _parse() {
    return guard([&](){
      if (expect_multipuncs(type, INC, "++", "--", "*", "&", "+", "-", "!", "~", "|")) {                               
        return expect(lhs);
      } else if (expect_key("sizeof")) {                                                                               
        type = SIZEOF;
        return expect(lhs);
      } else if (expect_key("typeof")) {                                                                               
        type = TYPEOF;
        return expect(lhs);
      } else return false;                                                                                             
    }());                                                                                                              
  }
};

struct castexpr_t : public ast_node_t {
  std::shared_ptr<unaryexpr_t> unary;
  std::shared_ptr<cast_t> cast;

  virtual bool _parse() {
    return expect(cast) || expect(unary);
  }
};

struct cast_t : public ast_node_t {
  std::shared_ptr<castexpr_t> lhs;
  std::shared_ptr<type_t> type;

  virtual bool _parse() {
    return guard(expect_punc("(") && expect(type) && expect_punc(")") && expect(lhs));
  }
};

struct mulexpr_t : public ast_node_t {
  std::shared_ptr<castexpr_t> cast;
  std::shared_ptr<mul_t> mul;

  virtual bool _parse() {
    return expect(mul) || expect(cast);
  }
};

struct mul_t : public ast_node_t {
  std::shared_ptr<mulexpr_t> lhs;
  std::shared_ptr<castexpr_t> rhs;
  int type;
  enum { MUL, DIV, MOD };

  virtual bool _parse() {
    return left_aggregate(mul, cast, expect_multipuncs(type, MUL, "*", "/", "%"));
  }
};

struct addexpr_t : public ast_node_t {
  std::shared_ptr<mulexpr_t> mul;
  std::shared_ptr<add_t> add;

  virtual bool _parse() {
    return expect(add) || expect(mul);
  }
};

// ...
</code></pre>
</div></div>

Each derived class has its own `_parse` function, which is short and brief. In this example,
it fits well in the  _Polymorphism_ style.

But when it goes to the semantics, which is still working on AST nodes, each virtual function
has grown into tens, if not hundreds, lines of code. Logics of semantics are distributed into 
the class definition of AST nodes, mixed in the definition of syntaxes.

<div><button class="collapsible">Collapsed</button><div class="collapsible-content">
<pre><code>
struct unary_t : public ast_node_t {
  std::shared_ptr<unaryexpr_t> lhs;                                                                                    
  int type; 
  enum { INC, DEC, PROD, ALL, POS, NEG, NOT, REV, ANY, SIZEOF, TYPEOF };                                               

  virtual bool _parse() {
    return guard([&](){
      if (expect_multipuncs(type, INC, "++", "--", "*", "&", "+", "-", "!", "~", "|")) {                               
        return expect(lhs);
      } else if (expect_key("sizeof")) {                                                                               
        type = SIZEOF;
        return expect(lhs);
      } else if (expect_key("typeof")) {                                                                               
        type = TYPEOF;
        return expect(lhs);
      } else return false;                                                                                             
    }());                                                                                                              
  }
  
  virtual symbol_t::ptr _analysis() {                                                                   
    auto lhs = lhs->analysis()->to<symbol_val_t>();                                                                     
    std::vector<std::tuple<int, std::string, bool>> oplut {                                                            
      { symbol_unary_t::INC, "++"s, true },                                                                            
      { symbol_unary_t::DEC, "--"s, true },
      { symbol_unary_t::PROD, "*"s, true },                                                                            
      { symbol_unary_t::ALL, "&"s, true },                                                                             
      { symbol_unary_t::POS, "+"s, true },                                                                             
      { symbol_unary_t::NEG, "-"s, true },                                                                             
      { symbol_unary_t::NOT, "!"s, true },                                                                             
      { symbol_unary_t::REV, "~"s, true },                                                                             
      { symbol_unary_t::ANY, "|"s, true },                                                                             
      { symbol_unary_t::SIZEOF, "sizeof"s, true },                                                                     
      { symbol_unary_t::TYPEOF, "typeof"s, true },
    }; 
    int opcode = std::get<0>(oplut[type]);                                                                        
    if (!std::get<2>(oplut[type])) {                                                                              
      error() << "prefix unary operator " << std::get<1>(oplut[type]) << " is not implemented." << eol();      
    } else if (!symbol_unary_t::op_accept(lhs->type, opcode)) {                                                        
      ast->error() << "prefix unary operator " << std::get<1>(oplut[type]) << " does not accept operand of type " << lhs->type->name() << "." << eol();
      lhs->type->note() << "type defined from here:" << lhs->type->eol();                                              
    }  
    if (lhs->type->external && lhs->type is typeid(symbol_vec_type_t) && !(opcode in std::set<int>{symbol_unary_t::SIZEOF, symbol_unary_t::TYPEOF})) {
      error() << "extern vector must be explicitly moved to intern variables before participating operations." << eol();                                                                                                            
      lhs->type->note() << "type defined from here:" << lhs->type->eol();                                              
    }                                                                                                                  
    auto rvck = [&,this](symbol_val_t::ptr opr) {                                                                           
      if (opr->rvvalue()) {
        error() << "expression do not accept rv-value. convert to lvalue by assignment first." << eol();     
        opr->type->note() << "type defined from here:" << opr->type->eol();
      } return 0;
    };
    if (opcode in std::set<int>{symbol_unary_t::PROD, symbol_unary_t::ALL,
                                symbol_unary_t::POS, symbol_unary_t::ANY})
      rvck(lhs);
    return std::make_shared<symbol_unary_t>(ast, lhs, opcode);
  }
  
};

struct castexpr_t : public ast_node_t {
  std::shared_ptr<unaryexpr_t> unary;
  std::shared_ptr<cast_t> cast;

  virtual bool _parse() {
    return expect(cast) || expect(unary);
  }
  
  virtual symbol_t::ptr _analysis() {
    return cast ? cast->analysis() : unary->analysis();
  }
};

...
</code></pre>
</div></div>

Without a doubt, you may separate the member function definitions and the class declaration into sources and headers.
I was working in a single-source-file compiler, thus I am unwilling to write declarations separately.
I would like to have all my `_analysis` functions put together, to form a huge switch-case according to the runtime type
information of a given AST node pointer.

    int acc = 0;
    switch([The runtime type of *p]) {
      case [type foo_t]: [upgrade p to point to a derived class] {
        p->a++;            // access the derived class members of *p.
        acc += p->a + 0;   // return results out of the switch statement.
      } break;
      // ...
    }

How could I do that?

One of the naive approaches is to use cascading `if` statements, calling `dynamic_pointer_cast` to check if `*p`
is of a specific derived class. Then do `dynamic_pointer_cast` again to promote the pointer, to access its members.

    ;      if (std::dynamic_pointer_cast<foo_t>(p)) { 
      acc += std::dynamic_pointer_cast<foo_t>(p)->a++ + 0; 
    } else if (std::dynamic_pointer_cast<bar_t>(p)) { 
      acc += std::dynamic_pointer_cast<bar_t>(p)->b++ + 1; 
    } else if (std::dynamic_pointer_cast<buz_t>(p) { 
      acc += std::dynamic_pointer_cast<buz_t>(p)->c++ + 2; 
    } else if (std::dynamic_pointer_cast<qux_t>(p) { 
      acc += std::dynamic_pointer_cast<qux_t>(p)->d++ + 3; 
    } 

This approach is ugly, lengthy, but not so bad for efficiency considerations. It runs about 10x slower than a virtual function call.
That is already a comparable performance, isn't it!

Can we convert the cascading `if`s into a switch? Since C++ STL has provided [`std::type_index`](https://en.cppreference.com/w/cpp/types/type_index)
to get unique id from types, the code should be envisaged as:

    switch((int)std::type_index(typeid(*p))) {
      case std::type_index(typeid(foo_t)): {
        acc += std::dynamic_pointer_cast<foo_t>(p)->a++ + 0;
      } break;
      case std::type_index(typeid(bar_t)): {
        acc += std::dynamic_pointer_cast<bar_t>(p)->b++ + 1;
      } break;
      case std::type_index(typeid(buz_t)): {
        acc += std::dynamic_pointer_cast<buz_t>(p)->c++ + 2;
      } break;
      case std::type_index(typeid(qux_t)): {
        acc += std::dynamic_pointer_cast<qux_t>(p)->d++ + 3;
      } break;
    }

Unfortunately, `std::type_index` cannot be `constexpr`, which is required for case labels. The code above is not legal, although we have expressed all the information we need to compile it.
We could have some workarounds, e.g. to provide a lookup table to translate `std::type_index(typeid(*p))` into enumerations,
or use [ctti](https://github.com/Manu343726/ctti), which is the abbrev. of <u style="text-decoration: underline;">C</u>ompile-<u style="text-decoration: underline;">T</u>ime <u style="text-decoration: underline;">T</u>ype <u style="text-decoration: underline;">I</u>nfo.
But neither do I want to write LUTs each time when I write type-switches, nor do I want to include additional dependencies to the program.
I made my solution, which must requires no extra codes whenever I use it.

    #include <memory>
    #include <unordered_map>
    #include <functional>
    #include <typeindex>
    
    template<class Base, class Ret, class... Args>
    Ret type_switch(std::shared_ptr<Base> obj, Ret(*...args)(std::shared_ptr<Args>)) {
      std::unordered_map<std::type_index, std::function<Ret(std::shared_ptr<Base>)>> _case_map {
        {
          std::type_index(typeid(Args)),
          std::function<Ret(std::shared_ptr<Base>)>(
            [=](std::shared_ptr<Base> fwd) {
              return (*args)(std::dynamic_pointer_cast<Args>(fwd));
            }
          )
        }...
      };
      return _case_map[std::type_index(typeid(*obj))](obj);
    }

I will dive into it a little bit. Starting from the function-template signature.

    template<class Base, class Ret, class... Args>
    Ret type_switch(std::shared_ptr<Base> obj, Ret(*...args)(std::shared_ptr<Args>)) {

This is a templated function `type_switch` which takes 1+N parameters and 2+N template-parameters.

The first parameter `obj`, is of type `std::shared_ptr<Base>`, which represents "a pointer of the base class".
In this parameter type, `Base` itself is a template-parameter, which can be auto-deduced from the type of first func-call argument.

The second parameter `args`, is a [parameter pack](https://en.cppreference.com/w/cpp/language/parameter_pack).
It accepts any number of arguments of any type, including zero arguments.
Each argument in the parameter pack will deduce two template-parameter types, `Ret` and `Args`,
forming a function-pointer type `Ret(*)(std::shared_ptr<Args>)`.

Since `Ret` is a template-parameter but `Args` is a template-parameter-pack, each argument in the parameter pack `args` may have different type `Args`, but must have the same type `Ret`.
`args` are switched functions to be invoked for each derived class `Args`.

    std::unordered_map<std::type_index, std::function<Ret(std::shared_ptr<Base>)>> _case_map

In the function body, we will build a map from type indexes of the derived class to the functions to be invoked.
The key of `_case_map` will be of type `std::type_index`, which will be probed from `typeid(*obj)` to differentiate runtime derived classes under the given pointer of base `obj`.
The value should be of a function wrapper type, which has the signature of `Ret(std::shared_ptr<Base>)`.
It is basically the same with `args`, despite that `Args` are derived classes and can be different, but it must be the same for all values in the map, so we change `Args` to `Base`.

The map is initialized with a [braced-initializer-list](https://en.cppreference.com/w/cpp/language/list_initialization),
where kv-pairs are unpacked from the parameter-pack `args` (together with the template-parameter-pack `Args`) by that ellipsis (it is not written as comments!).
The ellipsis will find the expression just before it, which will be a braced-initializer-list:

        {
          std::type_index(typeid(Args)),
          std::function<Ret(std::shared_ptr<Base>)>(
            [=](std::shared_ptr<Base> fwd) {
              return (*args)(std::dynamic_pointer_cast<Args>(fwd));
            }
          )
        }

For each value in the pack `args` and `Args`, the expression will be expanded for once.
Thus if `args` contains `a`, `b`, `c` and `d`, and `Args` contains `foo_t`, `bar_t`, `buz_t` and `qux_t`,
the code will be expanded as:

<div><button class="collapsible">Collapsed</button><div class="collapsible-content">
<pre><code>
      std::unordered_map<std::type_index, std::function<Ret(std::shared_ptr<Base>)>> _case_map {
        {
          std::type_index(typeid(foo_t)),
          std::function<Ret(std::shared_ptr<Base>)>(
            [=](std::shared_ptr<Base> fwd) {
              return (*a)(std::dynamic_pointer_cast<foo_t>(fwd));
            }
          )
        },{
          std::type_index(typeid(bar_t)),
          std::function<Ret(std::shared_ptr<Base>)>(
            [=](std::shared_ptr<Base> fwd) {
              return (*b)(std::dynamic_pointer_cast<bar_t>(fwd));
            }
          )
        },{
          std::type_index(typeid(buz_t)),
          std::function<Ret(std::shared_ptr<Base>)>(
            [=](std::shared_ptr<Base> fwd) {
              return (*c)(std::dynamic_pointer_cast<buz_t>(fwd));
            }
          )
        },{
          std::type_index(typeid(qux_t)),
          std::function<Ret(std::shared_ptr<Base>)>(
            [=](std::shared_ptr<Base> fwd) {
              return (*d)(std::dynamic_pointer_cast<qux_t>(fwd));
            }
          )
        }
      };
</code></pre>
</div></div>

That expanded key is identical to what we want to use as case labels in the previous attempt.
The value is constructing a lambda function and wrapping it into a function wrapper (value type of the map).
Since each lambda object will have its unique type, it must be wrapped if we want to keep it in an STL container (which is expecting all its values are of the same type).

          std::function<Ret(std::shared_ptr<Base>)>(
          
            [=](std::shared_ptr<Base> fwd) {
              return (*args)(std::dynamic_pointer_cast<Args>(fwd));
            }
            
          )

The lambda is capturing `args` (after expansion), then dereferencing it from a function-pointer (`Ret(*args)(std::shared_ptr<Args>)`) to a function (`Ret args(std::shared_ptr<Args>)`),
then invoke it with an argument `std::dynamic_pointer_cast<Args>(fwd)` which will do the pointer promotion for us thus we don't bother writing the second `dynamic_pointer_cast` for each case label.

    return _case_map[std::type_index(typeid(*obj))](obj);
    
For each call, we first look up the `_case_map` we built to find out the corresponding lambda we should invoke. This is the _switching_ process.
Then we invoke the lambda, which will promote the base class pointer to a derived class pointer, then invoke the case routine we given in the argument packs.

It will be used like this:

    acc += type_switch(obj,

      +[](std::shared_ptr<foo_t> obj) {
        return obj->a++ + 0;
      },
      +[](std::shared_ptr<bar_t> obj) {
        return obj->b++ + 1;
      },
      +[](std::shared_ptr<buz_t> obj) {
        return obj->c++ + 2;
      },
      +[](std::shared_ptr<qux_t> obj) {
        return obj->d++ + 3;
      }
    );

You call `type_switch` with the base class pointer for the first argument and write a pure-lambda for each derived class.
You will get the promoted pointer as the lambda parameter.

The [unary `+` operator](https://stackoverflow.com/questions/18889028/a-positive-lambda-what-sorcery-is-this) will convert the pure-lambda to a function pointer, as expected in the type deduction of `Ret` and `Args` in `type_switch`.

This magic looks far briefer than cascading `if`s. However, everything has its price. It runs 10x slower than cascading `if`s. It is harder to implement. And most critically, [GCC before 8.1.0 doesn't like it](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=47226).

According to your demands, it can be expanded further to be more flexible, e.g. a `default` statement, or enabling captures instead of pure-lambdas for each case routine. I will leave it here just as it is.
