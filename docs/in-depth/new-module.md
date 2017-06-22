# Adding a new module

You can easily extend MoMEMta by adding new modules to perform operations not supported by the base modules.

## Defining the module's interface

You declare a new module in MoMEMta by subclassing the [Module class](https://momemta.github.io/MoMEMta/dev/classModule.html), and then registering the new module into the MoMEMta system.

Let's suppose you want to declare a new module printing `Hello world!` at each step of the integration. You first need to include the right header:

```c++
#include <momemta/Module.h>
```

and then to subclass the `Module` class:

```c++
class HelloWorldModule: public Module {

};
```

If you want your new module to be useful, you'll want to override some specific functions, starting by the constructor and the [work](https://momemta.github.io/MoMEMta/dev/classModule.html#a43c0b7c70e9dd8b0cf6c78aa5052eeee) function:

```c++
class HelloWorldModule: public Module {
public:
  HelloWorldModule(PoolPtr pool, const ParameterSet& parameters): Module(pool, parameters.getModuleName()) { }

  virtual Status work() override {
    std::cout << "Hello world!" << std::endl;

    return Status::OK;
  }
};
```

The `work` function is called by MoMEMta each time the integrand needs to be evaluated (thousand times per integration). With this code, our module would only output the string `Hello world!` and not produce anything useful, but it's fine, that's what we wanted! The only remaining thing is to register the new module into the system.

## Registering the module

Before being available to MoMEMta, a module must be registered into the system. This is done using the `REGISTER_MODULE` macro. After the declaration of your module, simply add:

```c++
REGISTER_MODULE(HelloWorldModule)
    .Sticky();
```

Because your module does not produce any output, MoMEMta will automatically remove it from the computation graph, since it does not contribute in any way to the final integrand value. You can prevent this behaviour by registering your module as `sticky`: a `sticky` module is not removed from the computation graph if it's found to not contribute.

## Loading the module

So, you have successfully defined a new module and registered it with the system. In order to use it during the computation of the weight, you first have to _load_ the shared library where your module is.

Let's suppose your module is contained into `libhelloworldmodule.so`. In the lua configuration file, you need to first instruct MoMEMta to load this library to gain access to the new module:

```lua
load_modules('libhelloworldmodule.so')
```

You can now use your new module as you would do for a built-in module:

```lua
HelloWorldModule.hello = {}
```

## Using advanced features into your module

The module we built before is nice, but not very useful. Ideally, our module wants to gather some input data, transform them and output new data that in turn can be used by other modules. So we need two features: a way to get **inputs** and a way to produce **outputs**.

### Inputs and outputs

A module retrieves its inputs from the memory pool. The only moment the pool is accessible is during the construction of the module (see the first argument of the constructor). The [`Pool` class](https://momemta.github.io/MoMEMta/dev/classPool.html) has a `get` method you must use to obtain a pointer to the input. The `get` method accepts only a single argument, the `InputTag` of the input to grab.

!!! warning
    Inputs can only be accessed inside the `work` method. Trying to access to an input in another method will result in an undefined behaviour.

Let's edit our `Hello, world!` module to retrieve an input:

```C++ hl_lines="4 9 15" class HelloWorldModule: public Module { public: HelloWorldModule(PoolPtr pool, const ParameterSet& parameters): Module(pool, parameters.getModuleName()) { input = pool->get

<double>(InputTag("module", "output"));
  }</double>

virtual Status work() override { std::cout << "Hello world!" << std::endl; std::cout << "Our input is: " << *input << std::endl;

```
return Status::OK;
```

}

private: Value

<double> input;
};</double>

````

The first addition grab an input of type `double` from the memory pool, expected to be produced by a module named `module` and called `output` (remember that the _output_ of a module becomes the _input_ of another module). The second addition print the actual value of the input at each integration step. In order to access the value of the input, you must dereference the variable (`Value` actually acts as a pointer to the real value, so you can also use the arrow (`->`) operator).

Outputs (values produced by the module) are declared in a similar way. The `Module` class has a method called `produce` you can use to register an output. It accepts a single argument which is the name of the output. A module can register any numbers of outputs, but they all must have a unique name.

!!! tip
    Output values are not reset at the beginning of each integration step. It's good practice to start your `work` function by resetting the value of all your outputs.

 Let's again modify our module to add a new output:

```C++ hl_lines="6 13 21"
class HelloWorldModule: public Module {
public:
  HelloWorldModule(PoolPtr pool, const ParameterSet& parameters): Module(pool, parameters.getModuleName()) {
    input = pool->get<double>(InputTag("module", "output"));

    output = produce<double>("my_output");
  }

  virtual Status work() override {
    std::cout << "Hello world!" << std::endl;
    std::cout << "Our input is: " << *input << std::endl;

    *output = *input * 2;

    return Status::OK;
  }

private:
  Value<double> input;

  std::shared_ptr<double> output;
};
```

First addition register an output named `my_output` for this module, of type `double`. The second addition set the value of the output to be twice the input value.

### Parameters

In the above example, we hardcoded the `InputTag` used to retrieve the input. This is not a good practice, because you can't know when writing your module how the user will want to use your module and which output he wants to connect to your module. To overcome this, you can use the parameters (second argument of the constructor) to get data specified by the user inside the configuration file. In our example here, we can request the user to tell, inside the configuration, how he wants to connect our module by letting him specify the `InputTag` to use.

Parameters are stored into a [`ParameterSet`](https://momemta.github.io/MoMEMta/dev/classParameterSet.html), which is simply a dictionary of heterogeneous types. You can access values using the `get` function, specifying the name of the parameter you want.

!!! note
    Only a subset of types are supported by `ParameterSet`. You can only store and retrieve `int64_t` (integers), `double` (floating point numbers), `bool`, `std::string`, `InputTag`, `ParameterSet`, as well as `std::vector` of any of the previous types.

We will modify our module to remove the hardcoded `InputTag` and except the user to pass it as a parameter:

```C++ hl_lines="4 5"
class HelloWorldModule: public Module {
public:
  HelloWorldModule(PoolPtr pool, const ParameterSet& parameters): Module(pool, parameters.getModuleName()) {
    InputTag inputTag = parameters.get<InputTag>("which_input");
    input = pool->get<double>(inputTag);

    output = produce<double>("my_output");
  }

  virtual Status work() override {
    std::cout << "Hello world!" << std::endl;
    std::cout << "Our input is: " << *input << std::endl;

    *output = *input * 2;

    return Status::OK;
  }

private:
  Value<double> input;

  std::shared_ptr<double> output;
};
```

You can now see it's a two-step process to retrieve an input:

  1. We retrieve the parameter `which_input` of type `InputTag`, specified by the user inside the configuration file.
  2. We use this `InputTag` to retrieve our input.

On the configuration side, the user must now specify a value for the `which_input` parameter:

```lua
HelloWorldModule.hello = {
  which_input = "module::output"
}
```

!!! tip
    You can provide a default value when retrieving an argument. This value will be returned if the parameter is not found in the set.

!!! tip
    You can use the `exists` method of `ParameterSet` to check if a given parameter exists.

### Module registration