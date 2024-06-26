# sword
Sword is a **fine-grained, function-first, front-end** framework for the web that leverages the power of the *Onyx* programming language and runs on WebAssembly.

Sword is still in active and early development and as of current relies on the nightly version of Onyx.

```fsharp
greeting :: (name: str) =>
    div(class="greeting", id=name)(
        // Get excited!
        h1("Hello ", name) |> onclick(el => el |> extend("!"))
        style("text-decoration: underline")
    )

greeting("Bob") |> append(get_body())
greeting(name="Sarah") |> append(get_body())
```

# Philosophy
Sword seeks to stay away from a very macro-heavy way of web development which is very common. Macros can enable very
nice and exact syntax, but they can also introduce a lot of complexity, forcing the library developer and the programmer
to fight with a new syntax, in which everything is redefined in such a way that has to be effectively compatible with the
host language. Instead, Sword wants to enable markup inside of true Onyx syntax, and tap into the powers that the language
has available to it by default without reinventing four-hundred wheels. As a result there is not a distinct notion of a "component" or a "property" in Sword as is common in
most web frameworks - and this is because every "component" and "property" is, in actuality, just a procedure (or function)
with arguments. This greatly boosts flexibility and allows you to be much more programmatic with your markup manipulation
and introduces some very nice benefits like type-safety on properties and easy handling of state. 

Sword is philosophically comprised of three core tenents: ***simplicity, composability, and beauty.*** 
1. Sword is ***simple.*** If you have a good understanding of HTML and CSS, and a decent working knowledge
of core programming concepts, it should be easy to pick up Sword and begin creating UIs that are dynamic,
interactive, and performant. No more of a framework jumping out with lots of complex classes and APIs that cause you to feel like you're no longer working with the language that you are - Sword is designed to feel like you're using Onyx how it is. Instead of crowding the developer with a ridiculous amount of features, Sword wants to get out of your way.
2. Sword is ***composable.*** Every piece of a UI can quickly and easily be pulled into components, manipulated and extended upon
as a template for further components, and ultimately handled however the programmer may wish. There is no view macro, there is no alternative syntax, there is no special rule for defining components and properties, and there is nothing besides pure programming
power. Composability doesn't just apply to the macro level; it also applies to the micro. Every individual HTML element can be composed of reusable attributes via currying and piped into extension functions to truly leverage composability at every level of programming.
3. Sword is ***beautiful.*** If a tool is not fun to use (either to write or read) it will never truly sustain itself. Sword is
designed to be beautiful to look at, to write in, to work with - abstracting away what deserves to be abstracted without removing agency from the programmer and taking control out of their hands. Sword should be *fun* to work with everyday, so that your job won't suck. 

# Examples
Here are some examples of Sword in action:

## Reactivity
```fsharp
count := signal(0)
double := computed(([count]) => count->get() * 2)

div (
    "Count: ", count
    "Double: ", double
    div (
        style("color: blue; padding: 2em; border: 1px solid blue")
        "Increment"
    ) |> onclick(([count]) => count->set(count->get() + 1))
)
```

## DOM Querying
```fsharp
count := signal(0)

counter_h1 := get_dom()
|> query("h1#counter")
|> react_inner("Count: ", count)

new_button := get_dom()
|> create("button")
|> set_inner("Increment")
|> append(get_body())
|> onclick(([count]) => count->set(count->get() + 1))
```

## View Templating
```fsharp
reusable_attr_div := div(class="curried" id="composable" style="color: blue")

header_text := h1(style="color: green") (
    "Hello World!"
)

main_view :=
    div (
        id("non-curried")
        style("color: red; margin: 1em")

        header_text
        |> extend(
            "Extend onto existing elements."
            div (
                "A div in an h1"
            )
        )

        "Check it out!"

        reusable_attr_div (
            "Use the curried form to define attributes first and reuse them!"
        )
    )
    |> append(get_body())
```

## Batch Functions
```fsharp
div(
    div()
    div()
    div()
)
// Could batch right away, but want to preserve parent div for later piping
|> keep(el => el
    |> get_children
    |> batch(el, i => el |> set_text("I'm Div #", i))
)
|> onclick(el => el |> remove())
```

## Properties and Components
```fsharp
count := signal(0)

// "Components" and "props" are just functions and arguments
counter :: (input: Signal(i32), num: i32) =>
    div(class="main")(
        h1("Count #", num, ": ", input)
        button("Increment") |> onclick(([input]) => input->increment())
        button("Decrement") |> onclick(([input]) => input->decrement())
    )

counter(count, 1) |> append(get_body())

// Sync state across components with signal arguments
counter(count, 2) |> append(get_body())

// Pass anonymous signals as props to isolate state
counter(signal(0), 3) |> append(get_body())
```

## Customizing with Pipes or Insertions
```fsharp
greeting :: (name: str) =>
    // You can either use pipe form or inserted form to modify elements
    // Organize however is best for your project/situation
    div(class="main", id=name)(
        // Get excited!
        h1(
            "Hello ", name

            // inserted event function runs on click
            onclick(el => el |> extend("!"))

            // apply runs on creation to allow for extra logic 
            // on element from inside view without piping after
            apply(el => el |> extend("??"))

            // this id func applies to the h1 using pipe form
        ) |> id("piped-id")

        // this style func applies to the div using inserted form
        style("text-decoration: underline")

        // this onclick func applies to div using pipe form
    ) |> onclick(el => el |> extend(button("Wanna click?")))

greeting("Bob") |> append(get_body())
```

## Custom Events
```fsharp
div(
    div(
        "Hello..."
    ) |> listen("burger", el => el |> extend(" World!"))
    button(
        "burger?"
    ) |> onclick(() => trigger("burger"))
) |> append(get_body())
```

## Todo MVC
```fsharp
todo :: (item: str) => {
    completeness := signal(false)
    main_div := 
        div(
            h1(item)
            button(
                "Complete"
            ) |> onclick(([completeness]) => completeness->toggle())
            button(
                "Remove"
            ) |> onclick((el) => el |> get_parent() |> remove())
        )

    effect(([completeness, main_div]) => {
        if completeness->get() {
            main_div |> set_style("text-decoration", "line-through") |> set_style("font-style", "italic")
        } else {
            main_div |> set_style("text-decoration", "none") |> set_style("font-style", "normal")
        }
    })

    return main_div
}

todolist :: () =>
    msg := signal("")
    div(
        h1("Todo List")
        input(attr="placeholder=Todo...")() |> model(msg)
        button("Add") |> onclick(() => { 
            msg->get() |> todo() |> append(get_body())
            msg->set("")
        })
    )

main :: () {
     todolist() |> append(get_body())
} 
```
# Reactive Contracts and Applicators
```fs
// Applicators allow you to create a function that can
// be called inside an element to apply behavior/props
_even_shy :: (input) => applicator(([input], el) =>
    el 
    // Contracts use .[T, F] signature for attributes
    // apply from result of paired reactive function 
    |> style_contract(
        .{.["color: blue", "color: red"]
            ([input]) => input->get() % 2 == 0}
        .{.["text-decoration: underline", "font-style: italic"]
            ([input]) => input->get() % 2 == 0}
    )
    |> class_contract(
        .{.["even", "odd"], ([input]) => input->get() % 2 == 0})
    // Child contract can use 1-length slice for truth statements too
    |> child_contract(
        .{.[ h1("Hello"), h1("World") ], ([input]) => input->get() % 2 == 0})
    |> attr_contract(
        .{.["href=#target", ""]
            ([input]) => input->get() % 4 == 0}
        .{.["", "type=funny"]
            ([input]) => input->get() % 8 == 0})
)

count := signal(0)

div(
    h1(
        "Count: ", count
        // By convention, precede applicator names 
        // with underscore for readability
        _even_shy(count)
    ) 
    button("Increment") |> onclick(([count]) => count->increment())
) 
|> append(get_body())
```

# Reactive Form Input Modelling
```fs
msg := signal("")

div(
    textarea(attr="placeholder=Type...")() |> model(msg)
    br()
    span(msg) |> style_contract(
        .{.["color: blue", "color: red"]
            ([msg]) => msg->get().length % 2 == 0}
        .{.["text-decoration: underline", "font-style: italic"]
            ([msg]) => msg->get().length % 3 == 0}
    )
) 
|> append(get_body())
```

# Installation
```
onyx self-upgrade nightly
onyx add https://github.com/elias-michaias/sword
```
