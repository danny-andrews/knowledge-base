Level 1 (Loops) - (`map`, `reduce`, `filter`, [more](http://ramdajs.com/)).

Level 2 (Callbacks/Async) - Use `Promise`s (or better yet, `Future`s).[*](https://github.com/fluture-js/Fluture).

Level 3 (Assignment) - [`pipe`](http://ramdajs.com/docs/#pipe) or [`compose`](http://ramdajs.com/docs/#compose) functions together to avoid unnecessary intermediate variable declarations.

*A wild monad appears!* [Monet Olé](https://monet.github.io/monet.js/)!

Level 4 (`null`/`undefined`) - Use `Maybe` monad.[*](https://monet.github.io/monet.js/#maybe).

Level 5 (`throw`) - Just `return` `Error`s instead. The `function` in this case will have a [union](https://guide.elm-lang.org/types/union_types.html)/[variant](https://realworldocaml.org/v1/en/html/variants.html) return type e.g. `Int | Error`. Alternatively, use the `Task` [monad](http://package.elm-lang.org/packages/elm-lang/core/4.0.0/Task).

Level 6 (Branches) - Use `Either` monad[*](https://monet.github.io/monet.js/#either)

*[Purity](https://www.youtube.com/watch?v=ETbGpGJNVLM&list=RDEMaCHvahMw3bRKtOfeqW0eDA&index=5) enters the stage*

Level 7 (Side-effects) - Use I/O Monad[*](https://monet.github.io/monet.js/#io)

Continue using other OO patterns!

🚨🚨🚨Use this!🚨🚨🚨: https://mat3e.github.io/brains/# and thank creator

Inspired by: https://www.youtube.com/watch?v=SfWR3dKnFIo
